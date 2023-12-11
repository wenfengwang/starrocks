---
displayed_sidebar: "Japanese"
---

# 非同期物理ビュー

本トピックでは、非同期物理ビューの理解、作成、使用、管理方法について説明します。非同期物理ビューは、StarRocks v2.4からサポートされています。

同期物理ビューと比較して、非同期物理ビューは複数のテーブルの結合とより多くの集計関数をサポートしています。非同期物理ビューの更新は、手動またはスケジュールされたタスクによってトリガーできます。また、全体の物理ビューではなく一部のパーティションのみを更新することも可能であり、更新のコストを大幅に削減できます。さらに、非同期物理ビューは自動的で透過的なクエリの高速化を可能にし、さまざまなクエリ書き換えシナリオをサポートしています。

同期物理ビュー（ロールアップ）のシナリオおよび使用方法については、[同期物理ビュー（ロールアップ）](../using_starrocks/Materialized_view-single_table.md)を参照してください。

## 概要

データベースのアプリケーションでは、大規模なテーブルでの複雑なクエリが頻繁に行われます。これらのクエリは、数十億行を含むテーブルの結合や集計を必要とし、計算には多くのシステムリソースと時間がかかります。

StarRocksの非同期物理ビューは、これらの課題に対処するために設計されています。非同期物理ビューは、1つ以上の基本テーブルから事前に計算されたクエリの結果を保持する特別な物理テーブルです。基本テーブル上で複雑なクエリを実行すると、StarRocksは関連する物理ビューから事前に計算された結果を返し、これにより繰り返し発生する複雑な計算を回避できます。クエリのパフォーマンスが向上し、頻繁に実行されるクエリや十分に複雑なクエリの場合、このパフォーマンスの違いは重要です。

さらに、非同期物理ビューはデータウェアハウスに数学モデルを構築するために特に有用です。これにより、上位層のアプリケーションに統一されたデータ仕様を提供し、基盤の実装を隠し、または基本テーブルの生データのセキュリティを保護できます。

### StarRocksの物理ビューの理解

StarRocks v2.3以前のバージョンでは、1つのテーブルにのみ構築できる同期物理ビュー（ロールアップ）が提供されていました。同期物理ビュー (v2.4からサポートされる非同期物理ビューと比較して) は、データの新鮮さが高く、更新コストが低い特徴を保持しています。ただし、非同期物理ビューでは、v2.4からサポートされる非同期物理ビューと比較して、多くの制限事項があります。同期物理ビューを構築してクエリを高速化または書き換えする際に、集計演算子の選択肢が限られています。

以下の表は、StarRocksにおける非同期物理ビュー (非同期 MV) と同期物理ビュー (同期 MV) の特徴を比較したものです:

|                       | **1つのテーブルによる集約** | **複数のテーブルの結合** | **クエリの書き換え** | **更新ストラテジー** | **基本テーブル** |
| --------------------- | ---------------------------- | -------------------- | ----------------- | -------------------- | -------------- |
| **非同期 MV** | はい | はい | はい | <ul><li>非同期更新</li><li>手動更新</li></ul> | デフォルトカタログの複数のテーブルおよび以下からのテーブル: <ul><li>外部カタログ (v2.5から)</li><li>既存の非同期物理ビュー (v2.5から)</li><li>既存のビュー (v3.1から)</li></ul> |
| **同期 MV (ロールアップ)**  | 集約関数の選択肢が限られている | いいえ | はい | データのロード中の同期更新 | デフォルトカタログの単一のテーブル |

### 基本概念

- **基本テーブル**

  基本テーブルは、物理ビューの基本的なテーブルです。

  StarRocksの非同期物理ビューでは、基本テーブルは、[デフォルトカタログ](../data_source/catalog/default_catalog.md)のStarRocksネイティブテーブル、外部カタログのテーブル (v2.5からサポート)、または既存の非同期物理ビュー (v2.5からサポート) およびビュー (v3.1からサポート) であることができます。StarRocksはすべての[StarRocksテーブルの種類](../table_design/table_types/table_types.md)で非同期物理ビューを作成することができます。

- **更新**

  非同期物理ビューを作成すると、そのデータは当時の基本テーブルの状態のみを反映します。基本テーブルのデータが変更されると、変更を同期させるために非同期物理ビューを更新する必要があります。

  現在、StarRocksは2つの一般的な更新戦略をサポートしています:

  - ASYNC: 非同期更新モード。基本テーブルのデータが変更されるたびに、事前定義された更新間隔に従って非同期物理ビューが自動的に更新されます。
  - MANUAL: 手動更新モード。非同期物理ビューは自動的に更新されません。更新タスクはユーザーが手動でトリガーすることができます。

- **クエリの書き換え**

  クエリの書き換えとは、物理ビューに蓄積したクエリの事前計算結果をクエリの実行時に再利用できるかどうかをシステムが自動的に判断することを意味します。再利用できる場合、システムは計算や結合に時間とリソースを消費するのを避けるため、関連する物理ビューからデータを直接読み込みます。

  v2.5から、StarRocksは、SPJGタイプの非同期物理ビューに基づく自動的で透過的なクエリの書き換えをサポートしています。SPJGタイプの物理ビューは、スキャン、フィルター、プロジェクト、および集計の演算子のみが含まれる計画を指します。

  > **注記**
  >
  > [JDBCカタログ](../data_source/catalog/jdbc_catalog.md)の基本テーブルに作成された非同期物理ビューは、クエリの書き換えをサポートしていません。

## 物理ビューの作成時期の決定

データウェアハウス環境において、以下の要求がある場合は、非同期物理ビューを作成することができます:

- **繰り返し発生する集約関数を使用したクエリの高速化**

  データウェアハウスのほとんどのクエリに同じ集約関数を含むサブクエリが含まれ、これらのクエリが多くの計算リソースを消費している場合を想定してください。このサブクエリに基づいて、非同期物理ビューを作成し、サブクエリのすべての結果を計算して保存することができます。物理ビューが構築された後、非同期物理ビューで格納された中間結果を読み込み、これによりサブクエリを含むすべてのクエリが書き換えられ、クエリが高速化されます。

- **複数のテーブルの定期的な結合**

  データウェアハウスで定期的に複数のテーブルを結合して新しいワイドテーブルを作成する必要があるとします。これらのテーブルに対して非同期物理ビューを構築し、固定された時間間隔で更新タスクをトリガーするASYNC更新戦略を設定することができます。物理ビューが構築された後、クエリの結果が物理ビューから直接返され、結合操作による遅延が回避されます。

- **データウェアハウスの階層化**

  データウェアハウスには多くの生データが含まれ、それをクエリするには複雑な一連のETL操作が必要です。データウェアハウス内で複数の階層の非同期物理ビューを構築し、データウェアハウスのデータを分層にすることができます。クエリを一連の単純なサブクエリに分解することができます。これにより、繰り返し計算が大幅に削減され、さらに重要なことに、DBAが問題を簡単で効率的に特定できます。さらに、データウェアハウスの階層化は生データと統計データを切り離し、機密性の高い生データを保護するのに役立ちます。

- **データレイクのクエリの高速化**

  データレイクのクエリは、ネットワークの遅延やオブジェクトストレージのスループットによって遅くなる場合があります。データレイクの上に非同期物理ビューを構築することで、クエリのパフォーマンスを向上させることができます。さらに、StarRocksはクエリを適切な物理ビューを使用するように自動的に書き換えるため、手動でクエリを変更する手間を省くことができます。

非同期物理ビューの具体的な使用例については、次のコンテンツを参照してください:

- [データモデリング](./data_modeling_with_materialized_views.md)
- [クエリの書き換え](./query_rewrite_with_materialized_views.md)
- [データレイククエリの高速化](./data_lake_query_acceleration_with_materialized_views.md)

## 非同期物理ビューの作成

StarRocksの非同期物理ビューは、以下の基本テーブル上に作成することができます:

- StarRocksのネイティブテーブル（すべてのStarRocksテーブルタイプをサポート）

- 外部カタログのテーブル (v2.5以降サポート) には次のものが含まれます:

  - Hiveカタログ (v2.5以降)
  - Hudiカタログ (v2.5以降)
  - Icebergカタログ (v2.5以降)
  - JDBCカタログ (v3.0以降)

- 既存の非同期物理ビュー (v2.5以降)
- 既存のビュー (v3.1以降)

### 作成の前に

次の例は、デフォルトカタログの2つの基本テーブルを対象としています:

- テーブル`goods`には、アイテムID `item_id1`、アイテム名 `item_name`、およびアイテム価格 `price`が記録されています。
- テーブル`order_list`には、注文ID `order_id`、クライアントID `client_id`、アイテムID `item_id2`、および注文日 `order_date`が記録されています。

カラム`goods.item_id1`は、カラム`order_list.item_id2`に相当します。

以下のステートメントを実行して、テーブルを作成し、データを挿入してください:

```SQL
CREATE TABLE goods(
    item_id1          INT,
    item_name         STRING,
    price             FLOAT
) DISTRIBUTED BY HASH(item_id1);

INSERT INTO goods
VALUES
    (1001,"apple",6.5),
    (1002,"pear",8.0),
    (1003,"potato",2.2);

CREATE TABLE order_list(
    order_id          INT,
    client_id         INT,
    item_id2          INT,
    order_date        DATE
) DISTRIBUTED BY HASH(order_id);

INSERT INTO order_list
VALUES
    (10001,101,1001,"2022-03-13"),
    (10001,101,1002,"2022-03-13"),
    (10002,103,1002,"2022-03-13"),
```sql
(10002,103,1003,"2022-03-14"),
    (10003,102,1003,"2022-03-14"),
    (10003,102,1001,"2022-03-14");
```

以下の例では、次のシナリオにおいて各注文の合計を頻繁に計算する必要があります。また、2つの基本テーブルを頻繁に結合し、`sum()` 集約関数を頻繁に使用する必要があります。さらに、ビジネスシナリオでは1日ごとにデータをリフレッシュする必要があります。

クエリ文は以下の通りです:

```SQL
SELECT
    order_id,
    sum(goods.price) as total
FROM order_list INNER JOIN goods ON goods.item_id1 = order_list.item_id2
GROUP BY order_id;
```

### マテリアライズド・ビューの作成

[CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md) を使用して特定のクエリ文に基づいてマテリアライズド・ビューを作成できます。

テーブル `goods`、`order_list` および上記のクエリ文に基づき、次の例では、各注文の合計を分析するためのマテリアライズド・ビュー `order_mv` を作成しています。マテリアライズド・ビューは1日ごとに自動的にリフレッシュされるように設定されています。

```SQL
CREATE MATERIALIZED VIEW order_mv
DISTRIBUTED BY HASH(`order_id`)
REFRESH ASYNC START('2022-09-01 10:00:00') EVERY (interval 1 day)
AS SELECT
    order_list.order_id,
    sum(goods.price) as total
FROM order_list INNER JOIN goods ON goods.item_id1 = order_list.item_id2
GROUP BY order_id;
```

> **注記**
>
> - 非同期マテリアライズド・ビューを作成する場合、マテリアライズド・ビューのデータ分散戦略またはリフレッシュ戦略、またはその両方を指定する必要があります。
> - 非同期マテリアライズド・ビューのパーティショニングおよびバケット分割戦略を基本テーブルと異なるものに設定することができますが、マテリアライズド・ビューのパーティションキーとバケットキーをクエリ文に含める必要があります。
> - 非同期マテリアライズド・ビューでは、より長期的な間隔での動的なパーティショニング戦略をサポートします。たとえば、基本テーブルが1日ごとにパーティション分割されている場合、マテリアライズド・ビューを1か月ごとにパーティション分割するように設定できます。
> - 現在、StarRocksではリスト分割戦略に基づく非同期マテリアライズド・ビューの作成やそのようなテーブルに基づくビューはサポートされていません。
> - マテリアライズド・ビューの作成に使用されるクエリ文では、rand()、random()、uuid()、sleep() を含むランダム関数をサポートしていません。
> - 非同期マテリアライズド・ビューはさまざまなデータ型をサポートします。詳細については、[CREATE MATERIALIZED VIEW - サポートされるデータ型](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md#supported-data-types) を参照してください。
> - デフォルトでは、CREATE MATERIALIZED VIEW ステートメントの実行時にリフレッシュタスクが直ちにトリガーされ、システムリソースの一定比率を消費することがあります。リフレッシュタスクを遅延させたい場合は、CREATE MATERIALIZED VIEW ステートメントに REFRESH DEFERRED パラメータを追加することができます。

- **非同期マテリアライズド・ビューのリフレッシュメカニズムについて**

  現在、StarRocksでは MANUAL refresh および ASYNC refresh の2つの ON DEMAND リフレッシュ戦略をサポートしています。

  StarRocks v2.5では、非同期マテリアライズド・ビューは多くの大きなパーティションを持つ場合、各リフレッシュが大量のリソースを消費する場合があります。v2.5では、StarRocksはリフレッシュタスクを分割することをサポートしています。リフレッシュタスクをバッチで実行し、バッチサイズを指定した最大パーティション数以下に設定することができます。この機能により、大きな非同期マテリアライズド・ビューが安定してリフレッシュされ、データモデリングの安定性と堅牢性が強化されます。
  - 非同期マテリアライズド・ビューのパーティションに対するタイム・トゥ・リブ （TTL）の指定が可能であり、マテリアライズド・ビューのストレージサイズを減らすことができます。
  - 最新の数個のパーティションのみをリフレッシュ範囲として指定することができ、リフレッシュのオーバーヘッドを削減できます。
  - データの変更により、対応する非同期マテリアライズド・ビューが自動的にリフレッシュされないようにすることができます。
  - リフレッシュタスクにリソースグループを割り当てることができます。

  詳細については、[CREATE MATERIALIZED VIEW - パラメータ](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md#parameters) の **PROPERTIES** セクションをご参照ください。また、[ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md) を使用して既存の非同期マテリアライズド・ビューのメカニズムを変更することもできます。

  > **注意**
  >
  > システムリソースを消費し、タスクの失敗を引き起こすリフレッシュ操作を防ぐため、推奨されるのは、パーティションされた基本テーブルに基づいてパーティション分割されたマテリアライズド・ビューを作成することです。これにより、基本テーブルのパーティション内でデータが更新された場合、マテリアライズド・ビューの対応するパーティションのみがリフレッシュされ、マテリアライズド・ビュー全体がリフレッシュされることはありません。詳細については、[マテリアライズド・ビューを使用したデータモデリング - パーティショニングモデリング](./data_modeling_with_materialized_views.md#partitioned-modeling) を参照してください。

- **ネストされたマテリアライズド・ビューについて**

  StarRocks v2.5は、ネストされた非同期マテリアライズド・ビューを作成することをサポートしています。既存の非同期マテリアライズド・ビューに基づく非同期マテリアライズド・ビューを作成できます。各マテリアライズド・ビューのリフレッシュ戦略は、上位層または下位層のマテリアライズド・ビューに影響を与えません。現在、StarRocksではネストレベルの数に制限はありませんが、本番環境ではネストレベルの数が3を超えないようにすることをお勧めします。

- **外部カタログに基づくマテリアライズド・ビューについて**

  StarRocksは、Hive カタログ（v2.5 以降）、Hudi カタログ（v2.5 以降）、Iceberg カタログ（v2.5 以降）、および JDBC カタログ（v3.0 以降）に基づいた非同期マテリアライズド・ビューの構築をサポートしています。外部カタログ上にマテリアライズド・ビューを作成する方法は、デフォルトのカタログ上の非同期マテリアライズド・ビューを作成する方法と類似していますが、いくつかの使用上の制約があります。詳細については、[データレイククエリの高速化 - マテリアライズド・ビュー](./data_lake_query_acceleration_with_materialized_views.md) を参照してください。

## 非同期マテリアライズド・ビューの手動リフレッシュ

リフレッシュ戦略に関係なく、[REFRESH MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/REFRESH_MATERIALIZED_VIEW.md) を使用して非同期マテリアライズド・ビューをリフレッシュできます。StarRocks v2.5では、非同期マテリアライズド・ビューの特定のパーティションを指定することでリフレッシュできます。さらに、v3.1では同期的にリフレッシュタスクを行い、タスクが成功または失敗した場合にのみSQL文が返されます。

```SQL
-- 非同期呼び出し（デフォルト）によるマテリアライズド・ビューのリフレッシュ
REFRESH MATERIALIZED VIEW order_mv;
-- 同期呼び出しによるマテリアライズド・ビューのリフレッシュ
REFRESH MATERIALIZED VIEW order_mv WITH SYNC MODE;
```

非同期呼び出しによって送信されたリフレッシュタスクは、[CANCEL REFRESH MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/CANCEL_REFRESH_MATERIALIZED_VIEW.md) を使用してキャンセルすることができます。

## 非同期マテリアライズド・ビューへの直接クエリ

作成した非同期マテリアライズド・ビューは、クエリ文に従って事前に計算された結果セットを含む物理的なテーブルとして基本テーブルのデータがリフレッシュされた後で直接クエリすることができます。

```Plain
MySQL > SELECT * FROM order_mv;
+----------+--------------------+
| order_id | total              |
+----------+--------------------+
|    10001 |               14.5 |
|    10002 | 10.200000047683716 |
|    10003 |  8.700000047683716 |
+----------+--------------------+
3 rows in set (0.01 sec)
```

> **注記**
>
> 非同期マテリアライズド・ビューを直接クエリできますが、結果は基本テーブルのクエリと一貫していない場合があります。

## 非同期マテリアライズド・ビューを使用したクエリのリライトと高速化

StarRocks v2.5では、SPJGタイプの非同期マテリアライズド・ビューに基づくクエリの自動的で透過的なリライトをサポートしています。SPJGタイプのマテリアライズド・ビューのクエリリライトには、単一テーブルクエリのリライト、結合クエリのリライト、集計クエリのリライト、Union クエリのリライト、およびネストされたマテリアライズド・ビューに基づくクエリリライトが含まれます。詳細については、[マテリアライズド・ビューによるクエリのリライト](./query_rewrite_with_materialized_views.md) を参照してください。

現在、StarRocksでは、デフォルトカタログまたはHiveカタログ、Hudiカタログ、Icebergカタログなどの外部カタログ上に作成された非同期マテリアライズド・ビューに基づくクエリのリライトがサポートされています。デフォルトカタログのデータをクエリする場合、StarRocksはデータが基本テーブルと一貫しない非同期マテリアライズド・ビューを除外することで、リライトされたクエリと元のクエリの結果の強い一貫性を確保します。非同期マテリアライズド・ビューのデータが期限切れになると、そのマテリアライズド・ビューは候補として使用されません。外部カタログのデータをクエリする場合、StarRocksは外部カタログのデータの変更を認識できないため、結果の強い一貫性を保証しません。外部カタログに基づく非同期マテリアライズド・ビューについては、[データレイククエリの高速化 - マテリアライズド・ビュー](./data_lake_query_acceleration_with_materialized_views.md) を参照してください。

> **注記**
>
> JDBCカタログのベーステーブルに基づいて作成された非同期マテリアライズド・ビューはクエリリライトをサポートしていません。

## 非同期マテリアライズド・ビューの管理

### 非同期マテリアライズド・ビューの変更
非同期マテリアライズド・ビューのプロパティを変更するには、[ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md) を使用できます。

- 非アクティブなマテリアライズド・ビューを有効化する。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv ACTIVE;
  ```

- 非同期マテリアライズド・ビューの名前を変更する。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv RENAME order_total;
  ```

- 非同期マテリアライズド・ビューの更新間隔を2日に変更する。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv REFRESH ASYNC EVERY(INTERVAL 2 DAY);
  ```

### 非同期マテリアライズド・ビューの表示

データベース内の非同期マテリアライズド・ビューは、[SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md) を使用するか、Information Schema のシステムメタデータビューをクエリすることで表示できます。

- データベース内のすべての非同期マテリアライズド・ビューを確認する。

  ```SQL
  SHOW MATERIALIZED VIEWS;
  ```

- 特定の非同期マテリアライズド・ビューを確認する。

  ```SQL
  SHOW MATERIALIZED VIEWS WHERE NAME = "order_mv";
  ```

- 名前に一致する特定の非同期マテリアライズド・ビューを確認する。

  ```SQL
  SHOW MATERIALIZED VIEWS WHERE NAME LIKE "order%";
  ```

- Information Schema の `materialized_views` メタデータビューをクエリして、すべての非同期マテリアライズド・ビューを確認します。詳細については、 [information_schema.materialized_views](../reference/information_schema/materialized_views.md) を参照してください。

  ```SQL
  SELECT * FROM information_schema.materialized_views;
  ```

### 非同期マテリアライズド・ビューの定義を確認

非同期マテリアライズド・ビューの作成に使用されたクエリは、[SHOW CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_MATERIALIZED_VIEW.md) を使用して確認できます。

```SQL
SHOW CREATE MATERIALIZED VIEW order_mv;
```

### 非同期マテリアライズド・ビューの実行ステータスを確認

非同期マテリアライズド・ビューの実行（構築または更新）ステータスは、[Information Schema](../reference/overview-pages/information_schema.md) の `tasks` と `task_runs` をクエリすることで確認できます。

以下の例では、最も最近作成されたマテリアライズド・ビューの実行ステータスを確認します。

1. `tasks` テーブル内の最新のタスクの `TASK_NAME` を確認します。

    ```Plain
    mysql> select * from information_schema.tasks  order by CREATE_TIME desc limit 1\G;
    *************************** 1. row ***************************
      TASK_NAME: mv-59299
    CREATE_TIME: 2022-12-12 17:33:51
      SCHEDULE: MANUAL
      DATABASE: ssb_1
    DEFINITION: insert overwrite hive_mv_lineorder_flat_1 SELECT `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderkey`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_linenumber`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_custkey`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_partkey`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderpriority`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_ordtotalprice`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_revenue`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`p_mfgr`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`s_nation`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`c_city`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`c_nation`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderdate`
    FROM `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`
    WHERE `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderdate` = '1997-01-01'
    EXPIRE_TIME: NULL
    1 row in set (0.02 sec)
    ```

2. これで、見つけた `TASK_NAME` を使用して `task_runs` テーブル内の実行ステータスを確認します。

    ```Plain
    mysql> select * from information_schema.task_runs where task_name='mv-59299' order by CREATE_TIME \G;
    *************************** 1. row ***************************
        QUERY_ID: d9cef11f-7a00-11ed-bd90-00163e14767f
        TASK_NAME: mv-59299
      CREATE_TIME: 2022-12-12 17:39:19
      FINISH_TIME: 2022-12-12 17:39:22
            STATE: SUCCESS
        DATABASE: ssb_1
      DEFINITION: insert overwrite hive_mv_lineorder_flat_1 SELECT `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderkey`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_linenumber`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_custkey`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_partkey`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderpriority`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_ordtotalprice`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_revenue`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`p_mfgr`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`s_nation`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`c_city`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`c_nation`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderdate`
    FROM `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`
    WHERE `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderdate` = '1997-01-01'
      EXPIRE_TIME: 2022-12-15 17:39:19
      ERROR_CODE: 0
    ERROR_MESSAGE: NULL
        PROGRESS: 100%
    2 rows in set (0.02 sec)
    ```

### 非同期マテリアライズド・ビューの削除

非同期マテリアライズド・ビューは、[DROP MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/DROP_MATERIALIZED_VIEW.md) を使用して削除できます。

```Plain
DROP MATERIALIZED VIEW order_mv;
```

### 関連するセッション変数

次の変数が非同期マテリアライズド・ビューの動作を制御します:

- `analyze_mv`: マテリアライズド・ビューの更新後に分析を実行するかどうか。有効な値は空の文字列（分析なし）、`sample`（サンプリングされた統計情報の収集）、`full`（完全な統計情報の収集）です。デフォルトは `sample` です。
- `enable_materialized_view_rewrite`: マテリアライズド・ビューの自動リライトを有効にするかどうか。有効な値は `true`（v2.5 以降のデフォルト）と `false` です。