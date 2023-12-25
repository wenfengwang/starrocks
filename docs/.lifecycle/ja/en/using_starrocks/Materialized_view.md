---
displayed_sidebar: English
---

# 非同期マテリアライズドビュー

このトピックでは、非同期マテリアライズドビューの理解、作成、使用、および管理方法について説明します。非同期マテリアライズドビューは、StarRocks v2.4以降でサポートされています。

同期マテリアライズドビューと比較して、非同期マテリアライズドビューは複数テーブル結合とより多くの集約関数をサポートします。非同期マテリアライズドビューのリフレッシュは、手動またはスケジュールされたタスクによってトリガーされます。また、マテリアライズドビュー全体ではなく、一部のパーティションのみをリフレッシュすることで、リフレッシュコストを大幅に削減することができます。さらに、非同期マテリアライズドビューは、さまざまなクエリ書き換えシナリオをサポートし、自動的かつ透過的なクエリ加速を実現します。

同期マテリアライズドビュー（Rollup）のシナリオと使用方法については、[同期マテリアライズドビュー（Rollup）](../using_starrocks/Materialized_view-single_table.md)を参照してください。

## 概要

データベースアプリケーションはしばしば、大規模なテーブルに対して複雑なクエリを実行します。これらのクエリには、数十億行を含むテーブルに対する複数テーブル結合と集約が含まれます。これらのクエリを処理することは、システムリソースと結果を計算するのにかかる時間の点でコストがかかる可能性があります。

StarRocksの非同期マテリアライズドビューは、これらの問題に対処するように設計されています。非同期マテリアライズドビューは、1つまたは複数のベーステーブルから事前に計算されたクエリ結果を保持する特別な物理テーブルです。ベーステーブルで複雑なクエリを実行すると、StarRocksは関連するマテリアライズドビューから事前に計算された結果を返し、これらのクエリを処理します。これにより、繰り返し行われる複雑な計算を避けることができ、クエリのパフォーマンスが向上します。このパフォーマンスの差は、クエリが頻繁に実行されるか、十分に複雑な場合に顕著になります。

さらに、非同期マテリアライズドビューは、データウェアハウス上で数学的モデルを構築する際に特に有用です。これにより、上層アプリケーションに統一されたデータ仕様を提供し、基盤となる実装を隠蔽し、ベーステーブルの生データセキュリティを保護することができます。

### StarRocksのマテリアライズドビューを理解する

StarRocks v2.3以前のバージョンでは、単一テーブル上でのみ構築可能な同期マテリアライズドビューが提供されていました。同期マテリアライズドビュー、またはRollupは、データの新鮮さを高く保ち、リフレッシュコストを低く抑えます。しかし、v2.4以降でサポートされる非同期マテリアライズドビューと比較すると、同期マテリアライズドビューには多くの制約があります。クエリを加速または書き換えるために同期マテリアライズドビューを構築したい場合、集約演算子の選択肢が限られています。

以下の表は、StarRocksの非同期マテリアライズドビュー（ASYNC MV）と同期マテリアライズドビュー（SYNC MV）を、それぞれがサポートする機能の観点から比較したものです。

|                       | **単一テーブル集約** | **複数テーブル結合** | **クエリ書き換え** | **リフレッシュ戦略** | **ベーステーブル** |
| --------------------- | ---------------------------- | -------------------- | ----------------- | -------------------- | -------------- |
| **ASYNC MV** | はい | はい | はい | <ul><li>非同期リフレッシュ</li><li>手動リフレッシュ</li></ul> | 複数テーブルから:<ul><li>デフォルトカタログ</li><li>外部カタログ (v2.5)</li><li>既存のマテリアライズドビュー (v2.5)</li><li>既存のビュー (v3.1)</li></ul> |
| **SYNC MV (Rollup)**  | 集約関数の選択肢が限られる | いいえ | はい | データロード中の同期リフレッシュ | デフォルトカタログ内の単一テーブル |

### 基本概念

- **ベーステーブル**

  ベーステーブルは、マテリアライズドビューの駆動テーブルです。

  StarRocksの非同期マテリアライズドビューにおいて、ベーステーブルは[デフォルトカタログ](../data_source/catalog/default_catalog.md)内のStarRocksネイティブテーブル、v2.5からサポートされる外部カタログのテーブル、またはv2.5からサポートされる既存の非同期マテリアライズドビューやv3.1からサポートされる既存のビューにすることができます。StarRocksは、[StarRocksテーブルの全タイプ](../table_design/table_types/table_types.md)に対して非同期マテリアライズドビューの作成をサポートしています。

- **リフレッシュ**

  非同期マテリアライズドビューを作成すると、そのデータは作成時点でのベーステーブルの状態を反映します。ベーステーブルのデータが変更された場合、マテリアライズドビューをリフレッシュして変更を同期させる必要があります。

  現在、StarRocksは2つの一般的なリフレッシュ戦略をサポートしています：

  - ASYNC: 非同期リフレッシュモード。ベーステーブルのデータが変更されるたびに、マテリアライズドビューは事前定義されたリフレッシュ間隔に従って自動的にリフレッシュされます。
  - MANUAL: 手動リフレッシュモード。マテリアライズドビューは自動的にリフレッシュされません。リフレッシュタスクはユーザーによって手動でのみトリガーされます。

- **クエリ書き換え**

  クエリ書き換えとは、マテリアライズドビューが構築されたベーステーブル上でのクエリ実行時に、システムが自動的に判断し、マテリアライズドビュー内の事前計算結果をクエリに再利用できるかどうかを判断することを意味します。再利用可能な場合、システムは関連するマテリアライズドビューから直接データをロードし、時間とリソースを消費する計算や結合を避けます。

  v2.5以降、StarRocksはSPJG型の非同期マテリアライズドビューに基づく自動的かつ透過的なクエリ書き換えをサポートしています。SPJG型マテリアライズドビューとは、プランにスキャン、フィルタ、プロジェクト、集約のタイプのオペレーターのみが含まれるマテリアライズドビューを指します。

  > **注記**
  >
  > [JDBCカタログ](../data_source/catalog/jdbc_catalog.md)内のベーステーブル上で作成された非同期マテリアライズドビューは、クエリ書き換えをサポートしていません。

## マテリアライズドビューを作成するタイミングを決定する

データウェアハウス環境において、以下のような要求がある場合に非同期マテリアライズドビューを作成できます：

- **繰り返し行われる集約関数を使用したクエリの高速化**


  データウェアハウス内のほとんどのクエリが集約関数を含む同じサブクエリを使用しており、これらのクエリがコンピューティングリソースの大部分を消費しているとします。このサブクエリに基づいて、サブクエリのすべての結果を計算して保存する非同期マテリアライズドビューを作成できます。マテリアライズドビューが構築された後、StarRocksはサブクエリを含むすべてのクエリを書き換え、マテリアライズドビューに格納された中間結果をロードし、これらのクエリを加速します。

- **複数テーブルの通常のJOIN**

  データウェアハウスで複数のテーブルを定期的に結合して新しいワイドテーブルを作成する必要があるとします。これらのテーブルに対して非同期マテリアライズドビューを構築し、一定の時間間隔で更新タスクをトリガーするASYNC更新戦略を設定できます。マテリアライズドビューが構築されると、クエリ結果はマテリアライズドビューから直接返されるため、JOIN操作によるレイテンシが回避されます。

- **データウェアハウスの階層化**

  データウェアハウスに大量の生データが含まれており、それに対するクエリには複雑なETL操作のセットが必要だとします。非同期マテリアライズドビューの複数のレイヤーを構築してデータウェアハウス内のデータを階層化し、クエリを一連の単純なサブクエリに分解できます。これにより、繰り返しの計算が大幅に削減され、さらに重要なことに、DBAが問題を容易かつ効率的に特定できるようになります。また、データウェアハウスの階層化は、生データと統計データを分離し、機密性の高い生データのセキュリティを保護するのに役立ちます。
  
- **データレイクでのクエリ加速**

  データレイクでのクエリは、ネットワークレイテンシやオブジェクトストレージのスループットにより遅くなることがあります。データレイク上に非同期マテリアライズドビューを構築することでクエリパフォーマンスを向上させることができます。さらに、StarRocksは既存のマテリアライズドビューを使用するようにクエリをインテリジェントに書き換えるため、クエリを手動で変更する手間を省くことができます。

非同期マテリアライズドビューの具体的なユースケースについては、以下のコンテンツを参照してください：

- [データモデリング](./data_modeling_with_materialized_views.md)
- [クエリ書き換え](./query_rewrite_with_materialized_views.md)
- [Data Lakeクエリ加速](./data_lake_query_acceleration_with_materialized_views.md)

## 非同期マテリアライズドビューの作成

StarRocksの非同期マテリアライズドビューは、以下のベーステーブル上で作成できます：

- StarRocksのネイティブテーブル（すべてのStarRocksテーブルタイプがサポートされています）

- 外部カタログのテーブル、以下を含む：

  - Hiveカタログ（v2.5以降）
  - Hudiカタログ（v2.5以降）
  - Icebergカタログ（v2.5以降）
  - JDBCカタログ（v3.0以降）

- 既存の非同期マテリアライズドビュー（v2.5以降）
- 既存のビュー（v3.1以降）

### 始める前に

以下の例では、デフォルトカタログ内の2つのベーステーブルが関係しています：

- テーブル`goods`は、アイテムID `item_id1`、アイテム名 `item_name`、アイテム価格 `price`を記録しています。
- テーブル`order_list`は、注文ID `order_id`、クライアントID `client_id`、アイテムID `item_id2`、注文日 `order_date`を記録しています。

カラム`goods.item_id1`は、カラム`order_list.item_id2`に相当します。

次のステートメントを実行してテーブルを作成し、データを挿入します：

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

次の例のシナリオでは、各注文の合計を頻繁に計算する必要があります。これには、2つのベーステーブルの頻繁な結合と集約関数`sum()`の集中的な使用が必要です。また、ビジネスシナリオでは、1日ごとの間隔でデータを更新することが求められます。

クエリステートメントは以下の通りです：

```SQL
SELECT
    order_id,
    sum(goods.price) as total
FROM order_list INNER JOIN goods ON goods.item_id1 = order_list.item_id2
GROUP BY order_id;
```

### マテリアライズドビューの作成

[CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md)を使用して、特定のクエリステートメントに基づいてマテリアライズドビューを作成できます。

テーブル`goods`、`order_list`、および上記のクエリステートメントに基づいて、次の例では各注文の合計を分析するマテリアライズドビュー`order_mv`を作成します。マテリアライズドビューは、1日ごとの間隔で自動的に更新されるように設定されています。

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
> - 非同期マテリアライズドビューを作成する際には、データ分散戦略またはマテリアライズドビューのリフレッシュ戦略を指定する必要があります。または、その両方を指定することもできます。
> - 非同期マテリアライズドビューには、ベーステーブルとは異なるパーティショニング戦略とバケッティング戦略を設定できますが、マテリアライズドビューを作成するために使用するクエリステートメントには、マテリアライズドビューのパーティションキーとバケットキーを含める必要があります。
> - 非同期マテリアライズドビューは、より長い期間での動的パーティショニング戦略をサポートします。例えば、ベーステーブルが1日ごとにパーティションされている場合、マテリアライズドビューを1ヶ月ごとにパーティションするように設定できます。
> - 現在、StarRocks はリストパーティショニング戦略を使用した非同期マテリアライズドビューの作成、またはリストパーティショニング戦略を使用して作成されたテーブルに基づく非同期マテリアライズドビューの作成をサポートしていません。
> - マテリアライズドビューの作成に使用されるクエリステートメントは、rand()、random()、uuid()、sleep() などのランダム関数をサポートしていません。
> - 非同期マテリアライズドビューは、さまざまなデータ型をサポートしています。詳細については、[CREATE MATERIALIZED VIEW - サポートされるデータ型](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md#supported-data-types)を参照してください。
> - デフォルトでは、CREATE MATERIALIZED VIEW ステートメントを実行すると、リフレッシュタスクが直ちにトリガーされ、システムリソースの一定割合を消費します。リフレッシュタスクを遅延させたい場合は、CREATE MATERIALIZED VIEW ステートメントに REFRESH DEFERRED パラメータを追加できます。

- **非同期マテリアライズドビューのリフレッシュメカニズムについて**

  現在、StarRocks は MANUAL リフレッシュと ASYNC リフレッシュの2つの ON DEMAND リフレッシュ戦略をサポートしています。

  StarRocks v2.5 では、非同期マテリアライズドビューは、リフレッシュのコストを制御し、成功率を高めるために、さまざまな非同期リフレッシュメカニズムをさらにサポートしています：

  - MV に大きなパーティションが多数ある場合、各リフレッシュは大量のリソースを消費する可能性があります。v2.5 では、StarRocks はリフレッシュタスクの分割をサポートしています。リフレッシュするパーティションの最大数を指定でき、StarRocks は指定された最大パーティション数以下のバッチサイズでバッチリフレッシュを実行します。この機能により、大規模な非同期マテリアライズドビューが安定してリフレッシュされ、データモデリングの安定性と堅牢性が向上します。
  - 非同期マテリアライズドビューのパーティションの存続時間 (TTL) を指定し、マテリアライズドビューが占めるストレージサイズを削減できます。
  - リフレッシュ範囲を指定して、最新のいくつかのパーティションのみをリフレッシュし、リフレッシュのオーバーヘッドを削減できます。
  - データ変更が対応するマテリアライズドビューの自動リフレッシュをトリガーしないベーステーブルを指定できます。
  - リフレッシュタスクにリソースグループを割り当てることができます。

  詳細については、[CREATE MATERIALIZED VIEW - パラメータ](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md#parameters)の **PROPERTIES** セクションを参照してください。また、[ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md) を使用して、既存の非同期マテリアライズドビューのメカニズムを変更することもできます。

  > **注意**
  >
  > システムリソースを枯渇させ、タスクの失敗を引き起こす可能性のある全リフレッシュ操作を防ぐために、パーティション化されたベーステーブルに基づいたパーティション化されたマテリアライズドビューを作成することを推奨します。これにより、ベーステーブルのパーティション内でデータ更新が発生した場合、マテリアライズドビュー全体ではなく、対応するマテリアライズドビューのパーティションのみがリフレッシュされます。詳細については、[マテリアライズドビューによるデータモデリング - パーティションモデリング](./data_modeling_with_materialized_views.md#partitioned-modeling)を参照してください。

- **ネストされたマテリアライズドビューについて**

  StarRocks v2.5 は、ネストされた非同期マテリアライズドビューの作成をサポートしています。既存の非同期マテリアライズドビューに基づいて非同期マテリアライズドビューを構築できます。各マテリアライズドビューのリフレッシュ戦略は、上位または下位のマテリアライズドビューに影響を与えません。現在、StarRocks はネストレベルの数に制限を設けていませんが、運用環境ではネストレイヤーの数が3を超えないことを推奨します。

- **外部カタログマテリアライズドビューについて**

  StarRocks は、Hive Catalog（v2.5以降）、Hudi Catalog（v2.5以降）、Iceberg Catalog（v2.5以降）、および JDBC Catalog（v3.0以降）に基づく非同期マテリアライズドビューの構築をサポートしています。外部カタログ上でのマテリアライズドビューの作成は、デフォルトカタログ上での非同期マテリアライズドビューの作成と似ていますが、いくつかの使用制限があります。詳細については、[データレイククエリの高速化にマテリアライズドビューを使用する](./data_lake_query_acceleration_with_materialized_views.md)を参照してください。

## 非同期マテリアライズドビューを手動でリフレッシュする

非同期マテリアライズドビューは、[REFRESH MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/REFRESH_MATERIALIZED_VIEW.md) を使用して、そのリフレッシュ戦略に関わらずリフレッシュできます。StarRocks v2.5 は、パーティション名を指定して非同期マテリアライズドビューの特定のパーティションをリフレッシュすることをサポートしています。StarRocks v3.1 は、リフレッシュタスクの同期呼び出しをサポートし、SQLステートメントはタスクが成功または失敗するまで返されません。

```SQL
-- 非同期呼び出しでマテリアライズドビューをリフレッシュします（デフォルト）。
REFRESH MATERIALIZED VIEW order_mv;
-- 同期呼び出しでマテリアライズドビューをリフレッシュします。
REFRESH MATERIALIZED VIEW order_mv WITH SYNC MODE;
```

[CANCEL REFRESH MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/CANCEL_REFRESH_MATERIALIZED_VIEW.md) を使用して、非同期呼び出しで送信されたリフレッシュタスクをキャンセルできます。

## 非同期マテリアライズドビューを直接クエリする

作成した非同期マテリアライズドビューは、クエリステートメントに従って事前に計算された結果の完全なセットを含む物理テーブルです。したがって、マテリアライズドビューが初めてリフレッシュされた後、直接クエリすることができます。

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
> 非同期マテリアライズドビューを直接クエリすることができますが、その結果はベーステーブルのクエリ結果と矛盾する可能性があります。

## 非同期マテリアライズドビューを使用したクエリの書き換えと加速

StarRocks v2.5は、SPJGタイプの非同期マテリアライズドビューに基づく自動的かつ透過的なクエリリライトをサポートしています。SPJGタイプのマテリアライズドビューによるクエリリライトには、単一テーブルクエリのリライト、Joinクエリのリライト、集約クエリのリライト、Unionクエリのリライト、およびネストされたマテリアライズドビューに基づくクエリのリライトが含まれます。詳細については、[マテリアライズドビューを使用したクエリリライト](./query_rewrite_with_materialized_views.md)を参照してください。

現在、StarRocksは、デフォルトカタログやHiveカタログ、Hudiカタログ、Icebergカタログなどの外部カタログで作成された非同期マテリアライズドビューに対するクエリのリライトをサポートしています。デフォルトカタログでデータをクエリする場合、StarRocksはベーステーブルとデータが一致しないマテリアライズドビューを除外することで、リライトされたクエリとオリジナルのクエリの間で結果の強い一貫性を保証します。マテリアライズドビューのデータが期限切れになると、そのマテリアライズドビューは候補として使用されません。外部カタログでデータをクエリする場合、StarRocksは外部カタログのデータ変更を検知できないため、結果の強い一貫性を保証しません。外部カタログに基づいて作成された非同期マテリアライズドビューの詳細については、[マテリアライズドビューを使用したデータレイククエリの高速化](./data_lake_query_acceleration_with_materialized_views.md)を参照してください。

> **注記**
>
> JDBCカタログのベーステーブルに作成された非同期マテリアライズドビューは、クエリリライトをサポートしていません。

## 非同期マテリアライズドビューの管理

### 非同期マテリアライズドビューの変更

非同期マテリアライズドビューのプロパティは、[ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md)を使用して変更できます。

- 非アクティブなマテリアライズドビューを有効にする。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv ACTIVE;
  ```

- 非同期マテリアライズドビューの名前を変更する。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv RENAME TO order_total;
  ```

- 非同期マテリアライズドビューのリフレッシュ間隔を2日に変更する。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv REFRESH ASYNC EVERY(INTERVAL 2 DAY);
  ```

### 非同期マテリアライズドビューの表示

データベース内の非同期マテリアライズドビューを[SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)を使用するか、Information Schemaのシステムメタデータビューをクエリすることで確認できます。

- データベース内のすべての非同期マテリアライズドビューを確認する。

  ```SQL
  SHOW MATERIALIZED VIEWS;
  ```

- 特定の非同期マテリアライズドビューを確認する。

  ```SQL
  SHOW MATERIALIZED VIEWS WHERE NAME = 'order_mv';
  ```

- 名前にマッチする特定の非同期マテリアライズドビューを確認する。

  ```SQL
  SHOW MATERIALIZED VIEWS WHERE NAME LIKE 'order%';
  ```

- Information Schemaの`materialized_views`メタデータビューをクエリして、すべての非同期マテリアライズドビューを確認する。詳細については、[information_schema.materialized_views](../reference/information_schema/materialized_views.md)を参照してください。

  ```SQL
  SELECT * FROM information_schema.materialized_views;
  ```

### 非同期マテリアライズドビューの定義を確認

非同期マテリアライズドビューを作成するために使用されたクエリは、[SHOW CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_MATERIALIZED_VIEW.md)で確認できます。

```SQL
SHOW CREATE MATERIALIZED VIEW order_mv;
```

### 非同期マテリアライズドビューの実行ステータスを確認

非同期マテリアライズドビューの実行（ビルドまたはリフレッシュ）ステータスは、Information Schemaの[`tasks`](../reference/information_schema/tasks.md)および[`task_runs`](../reference/information_schema/task_runs.md)をクエリすることで確認できます。

以下の例では、最も最近に作成されたマテリアライズドビューの実行ステータスを確認します：

1. `tasks`テーブルで最新のタスクの`TASK_NAME`を確認します。

    ```Plain
    mysql> select * from information_schema.tasks order by CREATE_TIME desc limit 1\G;
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

2. 見つけた`TASK_NAME`を使用して、`task_runs`テーブルの実行状態を確認します。

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

### 非同期マテリアライズドビューの削除

非同期マテリアライズドビューは、[DROP MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/DROP_MATERIALIZED_VIEW.md)を使用して削除できます。

```Plain
DROP MATERIALIZED VIEW order_mv;
```

### 関連するセッション変数

非同期マテリアライズドビューの動作を制御する以下の変数について説明します：

- `analyze_mv`: マテリアライズドビューを更新後に分析するかどうか、及びどのように分析するか。有効な値は、空文字列（分析しない）、`sample`（サンプルによる統計収集）、`full`（完全な統計収集）です。デフォルトは`sample`です。
- `enable_materialized_view_rewrite`: マテリアライズドビューの自動書き換えを有効にするかどうか。有効な値は`true`（v2.5以降のデフォルト）と`false`です。
