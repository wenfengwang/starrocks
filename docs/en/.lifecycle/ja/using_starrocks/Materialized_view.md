---
displayed_sidebar: "Japanese"
---

# 非同期マテリアライズドビュー

このトピックでは、非同期マテリアライズドビューの理解、作成、使用、および管理方法について説明します。非同期マテリアライズドビューは、StarRocks v2.4以降でサポートされています。

同期マテリアライズドビューと比較して、非同期マテリアライズドビューは、複数のテーブルの結合とより多くの集計関数をサポートしています。非同期マテリアライズドビューのリフレッシュは、手動でトリガーすることも、スケジュールされたタスクによってトリガーすることもできます。マテリアライズドビュー全体ではなく、一部のパーティションのみをリフレッシュすることもでき、リフレッシュのコストを大幅に削減することができます。さらに、非同期マテリアライズドビューは、自動的で透過的なクエリの書き換えシナリオをサポートしており、クエリの高速化を実現します。

同期マテリアライズドビュー（ロールアップ）のシナリオと使用法については、[同期マテリアライズドビュー（ロールアップ）](../using_starrocks/Materialized_view-single_table.md)を参照してください。

## 概要

データベースのアプリケーションでは、大規模なテーブルに対して複雑なクエリを実行することがよくあります。このようなクエリは、数十億行を含むテーブルの結合と集計を伴います。これらのクエリの処理は、システムリソースの面や結果の計算にかかる時間の面でコストがかかることがあります。

StarRocksの非同期マテリアライズドビューは、これらの問題に対処するために設計されています。非同期マテリアライズドビューは、1つ以上のベーステーブルからの事前計算済みのクエリ結果を保持する特別な物理テーブルです。ベーステーブル上で複雑なクエリを実行すると、StarRocksは関連するマテリアライズドビューから事前計算済みの結果を返し、これらのクエリを処理します。これにより、繰り返し行われる複雑な計算を回避することで、クエリのパフォーマンスを向上させることができます。このパフォーマンスの違いは、クエリが頻繁に実行されるか、十分に複雑である場合には大きなものになる可能性があります。

さらに、非同期マテリアライズドビューは、データウェアハウス上に数学モデルを構築するのに特に有用です。これにより、上位レイヤーのアプリケーションに統一されたデータ仕様を提供し、基礎となる実装を隠蔽したり、ベーステーブルの生データのセキュリティを保護したりすることができます。

### StarRocksでのマテリアライズドビューの理解

StarRocks v2.3およびそれ以前のバージョンでは、単一のテーブルにのみ構築できる同期マテリアライズドビュー（ロールアップ）が提供されていました。同期マテリアライズドビューまたはロールアップは、より高いデータの新鮮さと低いリフレッシュコストを保持しています。ただし、v2.4以降でサポートされる非同期マテリアライズドビューと比較して、同期マテリアライズドビューには多くの制限があります。同期マテリアライズドビューを作成してクエリを高速化または書き換えする場合、集計演算子の選択肢が制限されます。

以下の表は、StarRocksにおける非同期マテリアライズドビュー（ASYNC MV）と同期マテリアライズドビュー（SYNC MV）の機能の観点からの比較です。

|                       | **単一テーブルの集計** | **複数テーブルの結合** | **クエリの書き換え** | **リフレッシュ戦略** | **ベーステーブル** |
| --------------------- | ---------------------------- | -------------------- | ----------------- | -------------------- | -------------- |
| **ASYNC MV** | はい | はい | はい | <ul><li>非同期リフレッシュ</li><li>手動リフレッシュ</li></ul> | デフォルトカタログからの複数のテーブル:<ul><li>デフォルトカタログ</li><li>外部カタログ（v2.5）</li><li>既存のマテリアライズドビュー（v2.5）</li><li>既存のビュー（v3.1）</li></ul> |
| **SYNC MV（ロールアップ）**  | 集計関数の選択肢が制限される | いいえ | はい | データのロード中に同期リフレッシュ | デフォルトカタログの単一テーブル |

### 基本的な概念

- **ベーステーブル**

  ベーステーブルは、マテリアライズドビューの駆動テーブルです。

  StarRocksの非同期マテリアライズドビューでは、ベーステーブルはStarRocksのネイティブテーブルである場合があります。[デフォルトカタログ](../data_source/catalog/default_catalog.md)のテーブル、外部カタログ（v2.5以降でサポート）、または既存の非同期マテリアライズドビュー（v2.5以降でサポート）やビュー（v3.1以降でサポート）でもベーステーブルとして使用できます。StarRocksは、すべての[StarRocksテーブルのタイプ](../table_design/table_types/table_types.md)で非同期マテリアライズドビューを作成することをサポートしています。

- **リフレッシュ**

  非同期マテリアライズドビューを作成すると、そのデータは作成時点でのベーステーブルの状態のみを反映します。ベーステーブルのデータが変更されると、変更を同期させるためにマテリアライズドビューをリフレッシュする必要があります。

  現在、StarRocksは2つの一般的なリフレッシュ戦略をサポートしています。

  - ASYNC: 非同期リフレッシュモード。ベーステーブルのデータが変更されるたびに、マテリアライズドビューは事前定義されたリフレッシュ間隔に従って自動的にリフレッシュされます。
  - MANUAL: 手動リフレッシュモード。マテリアライズドビューは自動的にリフレッシュされません。リフレッシュタスクはユーザーによって手動でトリガーすることしかできません。

- **クエリの書き換え**

  クエリの書き換えとは、マテリアライズドビューが構築されたベーステーブル上でクエリを実行する際に、システムが自動的にマテリアライズドビューの事前計算済みの結果をクエリに再利用できるかどうかを判断することを意味します。再利用できる場合、システムは関連するマテリアライズドビューからデータを直接ロードして、時間とリソースを消費する計算や結合を回避します。

  StarRocks v2.5以降では、SPJGタイプの非同期マテリアライズドビューに基づいた自動的で透過的なクエリの書き換えをサポートしています。SPJGタイプのマテリアライズドビューは、スキャン、フィルター、プロジェクト、および集計の演算子のみを含む計画を持つマテリアライズドビューを指します。

  > **注意**
  >
  > JDBCカタログのベーステーブルに作成された非同期マテリアライズドビューは、クエリの書き換えをサポートしていません。

## マテリアライズドビューを作成するタイミングを決定する

データウェアハウス環境で次の要件がある場合、非同期マテリアライズドビューを作成することができます。

- **繰り返し集計関数を使用してクエリを高速化する**

  データウェアハウスのほとんどのクエリが、同じ集計関数を含むサブクエリを含んでおり、これらのクエリが計算リソースの大部分を消費している場合を想定してみてください。このサブクエリに基づいて、サブクエリのすべての結果を計算して保存する非同期マテリアライズドビューを作成することができます。マテリアライズドビューが作成された後、サブクエリを含むすべてのクエリが書き換えられ、マテリアライズドビューに格納されている中間結果がロードされ、これらのクエリが高速化されます。

- **複数のテーブルの定期的な結合**

  データウェアハウスで複数のテーブルを定期的に結合して新しいワイドテーブルを作成する必要があるとします。これらのテーブルに対して非同期マテリアライズドビューを作成し、リフレッシュタスクを固定の時間間隔でトリガーするASYNCリフレッシュ戦略を設定することができます。マテリアライズドビューが作成された後、クエリ結果はマテリアライズドビューから直接返されるため、結合操作によるレイテンシが回避されます。

- **データウェアハウスのレイヤリング**

  データウェアハウスには大量の生データが含まれており、それに対するクエリには複雑なETL操作のセットが必要です。データウェアハウスのデータを層別にするために、複数のレイヤーの非同期マテリアライズドビューを作成し、クエリを一連の単純なサブクエリに分解することができます。これにより、繰り返し計算を大幅に削減することができ、さらに重要なことに、DBAが問題を簡単かつ効率的に特定するのに役立ちます。さらに、データウェアハウスのレイヤリングは、生データと統計データを分離することで、機密性の高い生データのセキュリティを保護します。

- **データレイクのクエリの高速化**

  データレイクのクエリは、ネットワークの遅延やオブジェクトストレージのスループットのために遅くなる場合があります。データレイクの上に非同期マテリアライズドビューを作成することで、クエリのパフォーマンスを向上させることができます。さらに、StarRocksはクエリをインテリジェントに書き換えて既存のマテリアライズドビューを使用することができるため、クエリを手動で変更する手間を省くことができます。

非同期マテリアライズドビューの具体的な使用例については、次のコンテンツを参照してください。

- [データモデリング](./data_modeling_with_materialized_views.md)
- [クエリの書き換え](./query_rewrite_with_materialized_views.md)
- [データレイクのクエリの高速化](./data_lake_query_acceleration_with_materialized_views.md)

## 非同期マテリアライズドビューを作成する

StarRocksの非同期マテリアライズドビューは、次のベーステーブルに作成することができます。

- StarRocksのネイティブテーブル（すべてのStarRocksテーブルタイプがサポートされています）

- 外部カタログのテーブル、次のような

  - Hiveカタログ（v2.5以降）
  - Hudiカタログ（v2.5以降）
  - Icebergカタログ（v2.5以降）
  - JDBCカタログ（v3.0以降）

- 既存の非同期マテリアライズドビュー（v2.5以降）
- 既存のビュー（v3.1以降）

### 開始する前に

次の例では、デフォルトカタログの2つのベーステーブルを使用します。

- テーブル`goods`には、アイテムID`item_id1`、アイテム名`item_name`、アイテム価格`price`が記録されています。
- テーブル`order_list`には、注文ID`order_id`、クライアントID`client_id`、アイテムID`item_id2`、注文日`order_date`が記録されています。

カラム`goods.item_id1`は、カラム`order_list.item_id2`と等価です。

次のステートメントを実行して、テーブルを作成し、データを挿入します。

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
    (10002,103,1003,"2022-03-14"),
    (10003,102,1003,"2022-03-14"),
    (10003,102,1001,"2022-03-14");
```

以下の例では、各注文の合計を頻繁に計算するシナリオが求められます。このシナリオでは、2つのベーステーブルの頻繁な結合と`sum()`集計関数の集計が必要です。さらに、ビジネスシナリオでは、データのリフレッシュを1日ごとに行う必要があります。

クエリステートメントは次のようになります。

```SQL
SELECT
    order_id,
    sum(goods.price) as total
FROM order_list INNER JOIN goods ON goods.item_id1 = order_list.item_id2
GROUP BY order_id;
```

### マテリアライズドビューを作成する

特定のクエリステートメントに基づいてマテリアライズドビューを作成するには、[CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md)を使用します。

テーブル`goods`、`order_list`、および上記で説明したクエリステートメントに基づいて、次の例では、マテリアライズドビュー`order_mv`を作成して各注文の合計を分析します。マテリアライズドビューは、1日ごとの間隔で自動的にリフレッシュされるように設定されています。

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

> **注意**
>
> - 非同期マテリアライズドビューを作成する際は、マテリアライズドビューのデータ分散戦略またはリフレッシュ戦略、またはその両方を指定する必要があります。
> - 非同期マテリアライズドビューには、ベーステーブルとは異なるパーティショニングおよびバケット化戦略を設定することができますが、マテリアライズドビューのパーティションキーとバケットキーを作成するために使用されるクエリステートメントに含める必要があります。
> - 非同期マテリアライズドビューは、より長いスパンで動的なパーティショニング戦略をサポートしています。たとえば、ベーステーブルが1日ごとにパーティション化されている場合、マテリアライズドビューを1か月ごとにパーティション化することができます。
> - 現在、StarRocksはリストパーティショニング戦略で非同期マテリアライズドビューを作成することや、リストパーティショニング戦略で作成されたテーブルに基づいて非同期マテリアライズドビューを作成することはサポートしていません。
> - マテリアライズドビューを作成するために使用されるクエリステートメントは、rand()、random()、uuid()、sleep()を含むランダム関数をサポートしていません。
> - 非同期マテリアライズドビューはさまざまなデータ型をサポートしています。詳細については、[CREATE MATERIALIZED VIEW - サポートされているデータ型](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md#supported-data-types)を参照してください。
> - デフォルトでは、CREATE MATERIALIZED VIEWステートメントを実行すると、リフレッシュタスクがすぐにトリガーされ、システムリソースの一定の割合を消費する場合があります。リフレッシュタスクを遅延させる場合は、CREATE MATERIALIZED VIEWステートメントにREFRESH DEFERREDパラメータを追加することができます。

- **非同期マテリアライズドビューのリフレッシュメカニズムについて**

  現在、StarRocksは2つのON DEMANDリフレッシュ戦略をサポートしています：MANUALリフレッシュとASYNCリフレッシュ。

  StarRocks v2.5では、非同期マテリアライズドビューはリフレッシュのコストを制御し、成功率を向上させるためのさまざまな非同期リフレッシュメカニズムをさらにサポートしています。

  - MVに多くの大きなパーティションがある場合、各リフレッシュは大量のリソースを消費する場合があります。v2.5では、StarRocksはリフレッシュタスクを分割することをサポートしています。リフレッシュするパーティションの最大数を指定し、StarRocksは指定された最大数以下のバッチサイズでリフレッシュをバッチ処理します。この機能により、大規模な非同期マテリアライズドビューの安定したリフレッシュが確保され、データモデリングの安定性と堅牢性が向上します。
  - 非同期マテリアライズドビューのパーティションごとにタイムトゥリーブ（TTL）を指定することで、マテリアライズドビューが占有するストレージサイズを削減することができます。
  - リフレッシュオーバーヘッドを削減するために、最新の数個のパーティションのみをリフレッシュするためのリフレッシュ範囲を指定することができます。
  - データの変更が自動的に対応するマテリアライズドビューのリフレッシュをトリガーしないようにするために、データの変更が自動的にトリガーされないベーステーブルを指定することができます。
  - リフレッシュタスクにリソースグループを割り当てることができます。

  詳細については、[CREATE MATERIALIZED VIEW - パラメータ](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md#parameters)の**PROPERTIES**セクションを参照してください。また、[ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md)を使用して既存の非同期マテリアライズドビューのメカニズムを変更することもできます。

  > **注意**
  >
  > システムリソースを使い果たし、タスクの失敗を引き起こすことを防ぐために、システムリソースを使い果たすフルリフレッシュ操作を防ぐために、パーティション化されたベーステーブルに基づいてパーティション化されたマテリアライズドビューを作成することをお勧めします。これにより、ベーステーブルのパーティション内でデータの更新が発生した場合、マテリアライズドビューの対応するパーティションのみがリフレッシュされ、マテリアライズドビュー全体がリフレッシュされることはありません。詳細については、[マテリアライズドビューを使用したデータモデリング - パーティション化モデリング](./data_modeling_with_materialized_views.md#partitioned-modeling)を参照してください。

- **ネストされたマテリアライズドビューについて**

  StarRocks v2.5では、ネストされた非同期マテリアライズドビューを作成することができます。既存の非同期マテリアライズドビューに基づいて非同期マテリアライズドビューを作成することができます。各マテリアライズドビューのリフレッシュ戦略は、上位または下位のマテリアライズドビューに影響を与えません。現在、StarRocksはネストレベルの数を制限していません。本番環境では、ネストレベルの数が3を超えないようにすることをお勧めします。

- **外部カタログマテリアライズドビューについて**

  StarRocksは、Hiveカタログ（v2.5以降）、Hudiカタログ（v2.5以降）、Icebergカタログ（v2.5以降）、およびJDBCカタログ（v3.0以降）に基づいて非同期マテリアライズドビューを作成することをサポートしています。外部カタログ上のマテリアライズドビューを作成する方法は、デフォルトカタログ上の非同期マテリアライズドビューを作成する方法と似ていますが、一部の使用制限があります。詳細については、[マテリアライズドビューを使用したデータレイクのクエリの高速化](./data_lake_query_acceleration_with_materialized_views.md)を参照してください。

## 非同期マテリアライズドビューを手動でリフレッシュする

[REFRESH MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/REFRESH_MATERIALIZED_VIEW.md)を使用して、非同期マテリアライズドビューをリフレッシュすることができます。StarRocks v2.5では、パーティション名を指定することで、非同期マテリアライズドビューの特定のパーティションをリフレッシュすることができます。StarRocks v3.1では、リフレッシュタスクを同期的に呼び出すことができ、タスクが成功または失敗した場合にのみSQLステートメントが返されます。

```SQL
-- 非同期呼び出し（デフォルト）でマテリアライズドビューをリフレッシュします。
REFRESH MATERIALIZED VIEW order_mv;
-- 同期呼び出しでマテリアライズドビューをリフレッシュします。
REFRESH MATERIALIZED VIEW order_mv WITH SYNC MODE;
```

非同期呼び出しで送信されたリフレッシュタスクをキャンセルするには、[CANCEL REFRESH MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/CANCEL_REFRESH_MATERIALIZED_VIEW.md)を使用します。

## 非同期マテリアライズドビューを直接クエリする

作成した非同期マテリアライズドビューは、クエリステートメントに従って事前計算済みの結果を含む物理テーブルです。したがって、マテリアライズドビューが最初にリフレッシュされた後、マテリアライズドビューを直接クエリすることができます。

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

> **注意**
>
> 非同期マテリアライズドビューを直接クエリすることができますが、ベーステーブルでのクエリとの結果が一致しない場合があります。

## 非同期マテリアライズドビューを使用してクエリを書き換えて高速化する

StarRocks v2.5では、SPJGタイプの非同期マテリアライズドビューに基づいた自動的で透過的なクエリの書き換えをサポートしています。SPJGタイプのマテリアライズドビューのクエリの書き換えには、単一テーブルのクエリの書き換え、結合クエリの書き換え、集計クエリの書き換え、UNIONクエリの書き換え、およびネストされたマテリアライズドビューに基づくクエリの書き換えが含まれます。詳細については、[マテリアライズドビューを使用したクエリの書き換え](./query_rewrite_with_materialized_views.md)を参照してください。

現在、StarRocksは、デフォルトカタログまたはHiveカタログ、Hudiカタログ、Icebergカタログなどの外部カタログに基づいて作成された非同期マテリアライズドビュー上のクエリを書き換えることができます。デフォルトカタログのデータをクエリする場合、StarRocksはベーステーブルとデータが一貫していないマテリアライズドビューを除外することで、書き換えられたクエリと元のクエリの結果の強い一貫性を確保します。マテリアライズドビューのデータが期限切れになると、マテリアライズドビューは候補マテリアライズドビューとして使用されません。外部カタログのデータをクエリする場合、StarRocksは外部カタログのデータの変更を感知できないため、結果の強い一貫性を保証しません。外部カタログに基づいて作成された非同期マテリアライズドビューについては、[マテリアライズドビューを使用したデータレイクのクエリの高速化](./data_lake_query_acceleration_with_materialized_views.md)を参照してください。

> **注意**
>
> JDBCカタログのベーステーブルに作成された非同期マテリアライズドビューは、クエリの書き換えをサポートしていません。

## 非同期マテリアライズドビューの管理

### 非同期マテリアライズドビューを変更する

[ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md)を使用して、非同期マテリアライズドビューのプロパティを変更することができます。

- 非アクティブなマテリアライズドビューを有効にする。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv ACTIVE;
  ```

- 非同期マテリアライズドビューの名前を変更する。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv RENAME order_total;
  ```

- 非同期マテリアライズドビューのリフレッシュ間隔を2日に変更する。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv REFRESH ASYNC EVERY(INTERVAL 2 DAY);
  ```

### 非同期マテリアライズドビューを表示する

[SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)を使用するか、Information Schemaのシステムメタデータビューをクエリすることで、データベース内の非同期マテリアライズドビューを表示することができます。

- データベース内のすべての非同期マテリアライズドビューを確認する。

  ```SQL
  SHOW MATERIALIZED VIEWS;
  ```

- 特定の非同期マテリアライズドビューを確認する。

  ```SQL
  SHOW MATERIALIZED VIEWS WHERE NAME = "order_mv";
  ```

- 名前に一致する非同期マテリアライズドビューを確認する。

  ```SQL
  SHOW MATERIALIZED VIEWS WHERE NAME LIKE "order%";
  ```

- Information Schemaのメタデータビュー`materialized_views`をクエリすることで、すべての非同期マテリアライズドビューを確認することもできます。詳細については、[information_schema.materialized_views](../reference/information_schema/materialized_views.md)を参照してください。

  ```SQL
  SELECT * FROM information_schema.materialized_views;
  ```

### 非同期マテリアライズドビューの定義を確認する

[SHOW CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_MATERIALIZED_VIEW.md)を使用して、非同期マテリアライズドビューを作成するために使用されたクエリを確認することができます。

```SQL
SHOW CREATE MATERIALIZED VIEW order_mv;
```

### 非同期マテリアライズドビューの実行ステータスを確認する

[Information Schema](../reference/information_schema/information_schema.md)の[`tasks`](../reference/information_schema/tasks.md)および[`task_runs`](../reference/information_schema/task_runs.md)をクエリすることで、非同期マテリアライズドビューの実行（ビルドまたはリフレッシュ）ステータスを確認することができます。

次の例では、最も最近作成されたマテリアライズドビューの実行ステータスを確認します。

1. `tasks`テーブルで最新のタスクの`TASK_NAME`を確認します。

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

2. 見つけた`TASK_NAME`を使用して、`task_runs`テーブルで実行ステータスを確認します。

    ```Plain
    mysql> select * from information_schema.task_runs where task_name='mv-59299' order by CREATE_TIME \G;
    *************************** 1. row ***************************
        QUERY_ID: d9cef11f-7a00-11ed-bd90-00163e14767f
        TASK_NAME: mv-59299
      CREATE_TIME: 2022-12-12 17:39:19
      FINISH_TIME: 2022-12-12 17:39:22
            STATE: SUCCESS
        DATABASE: ssb_1
    1 row in set (0.02 sec)
    ```


      DEFINITION: hive_mv_lineorder_flat_1に上書き挿入を行います。`hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderkey`、`hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_linenumber`、`hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_custkey`、`hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_partkey`、`hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderpriority`、`hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_ordtotalprice`、`hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_revenue`、`hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`p_mfgr`、`hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`s_nation`、`hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`c_city`、`hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`c_nation`、`hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderdate`を選択します。
    `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`から取得します。
    `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderdate`が'1997-01-01'の場合に対象とします。
      EXPIRE_TIME: 2022-12-15 17:39:19
      ERROR_CODE: 0
    ERROR_MESSAGE: NULL
        PROGRESS: 100%
    2 rows in set (0.02 sec)
    ```

### 非同期マテリアライズドビューの削除

[DROP MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/DROP_MATERIALIZED_VIEW.md)を使用して、非同期マテリアライズドビューを削除できます。

```Plain
DROP MATERIALIZED VIEW order_mv;
```

### 関連するセッション変数

以下の変数は非同期マテリアライズドビューの動作を制御します：

- `analyze_mv`：リフレッシュ後にマテリアライズドビューを分析するかどうかを制御します。有効な値は空の文字列（分析しない）、`sample`（サンプル統計の収集）、`full`（完全な統計の収集）です。デフォルトは`sample`です。
- `enable_materialized_view_rewrite`：マテリアライズドビューの自動リライトを有効にするかどうかを制御します。有効な値は`true`（v2.5以降のデフォルト）と`false`です。
