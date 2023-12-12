---
displayed_sidebar: "Japanese"
---

# 非同期マテリアライズドビュー

このトピックでは、非同期マテリアライズドビューの理解、作成、使用、管理方法について説明します。非同期マテリアライズドビューは、StarRocks v2.4以降でサポートされています。

同期マテリアライズドビューと比較して、非同期マテリアライズドビューは複数のテーブルの結合および集計など、さまざまな集計関数をサポートしています。非同期マテリアライズドビューの更新は、手動でトリガーすることも、スケジュールされたタスクによって自動的にトリガーすることもできます。マテリアライズドビュー全体ではなく、一部のパーティションのみを更新することも可能であり、更新のコストを大幅に削減することができます。さらに、非同期マテリアライズドビューは、自動的で透過的なクエリのリライトシナリオをサポートし、クエリの高速化を実現します。

同期マテリアライズドビュー（ロールアップ）のシナリオと使用方法については、[同期マテリアライズドビュー（ロールアップ）](../using_starrocks/Materialized_view-single_table.md)を参照してください。

## 概要

データベース上のアプリケーションでは、大きなテーブルに対して複雑なクエリが頻繁に実行されます。これらのクエリでは、数十億行を含むテーブルの複数のテーブルを結合したり集計したりする必要があります。これらのクエリの処理は、システムリソースの面でも、結果を計算するのにかかる時間の面でも非常に費用がかかります。

StarRocksの非同期マテリアライズドビューは、このような問題に対処するために設計されています。非同期マテリアライズドビューは、1つまたは複数のベーステーブルから事前計算されたクエリの結果を保持する特別な物理テーブルです。ベーステーブルで複雑なクエリを実行すると、StarRocksは関連するマテリアライズドビューから事前計算された結果を返し、これらのクエリを処理します。これにより、繰り返し行われる複雑な計算を回避することで、クエリのパフォーマンスを改善することができます。クエリが頻繁に実行される場合や十分に複雑な場合、このパフォーマンスの差は大きくなる可能性があります。

さらに、非同期マテリアライズドビューは、データウェアハウスに数学モデルを構築する際に特に有用です。これにより、上位レイヤのアプリケーションに統一されたデータ仕様を提供し、基礎となる実装を隠蔽したり、基底テーブルの生データのセキュリティを保護したりすることができます。

### StarRocksのマテリアライズドビューの理解

StarRocks v2.3以前のバージョンでは、単一のテーブルにのみ構築できる同期マテリアライズドビュー（ロールアップ）が提供されていました。同期マテリアライズドビューは、より高いデータの新鮮さと低い更新コストを保持しています。ただし、v2.4以降でサポートされる非同期マテリアライズドビューと比較すると、同期マテリアライズドビューには多くの制限があります。同期マテリアライズドビューを作成してクエリの高速化やリライトを実現する場合、選択できる集計演算子の選択肢は限られます。

次の表は、StarRocksにおける非同期マテリアライズドビュー（ASYNC MV）と同期マテリアライズドビュー（SYNC MV）の機能を比較したものです。

|                       | **単一テーブルの集計** | **複数テーブルの結合** | **クエリリライト** | **更新戦略** | **ベーステーブル** |
| --------------------- | ---------------------------- | -------------------- | ----------------- | -------------------- | -------------- |
| **ASYNC MV** | はい | はい | はい | <ul><li>非同期リフレッシュ</li><li>手動リフレッシュ</li></ul> | <ul><li>デフォルトカタログ</li><li>外部カタログ（v2.5）</li><li>既存のマテリアライズドビュー（v2.5）</li><li>既存のビュー（v3.1）</li></ul> |
| **SYNC MV (ロールアップ)**  | 集計関数の選択肢が限られる | いいえ | はい | データのロード時に同期的にリフレッシュ | デフォルトカタログの単一テーブル |

### 基本的な概念

- **ベーステーブル**

  ベーステーブルは、マテリアライズドビューの基になるテーブルです。

  StarRocksの非同期マテリアライズドビューでは、基となるテーブルは、デフォルトカタログのStarRocksネイティブテーブル、外部カタログ（v2.5以降でサポート）、既存の非同期マテリアライズドビュー（v2.5以降でサポート）、ビュー（v3.1以降でサポート）のいずれかにすることができます。StarRocksは、[すべてのStarRocksテーブルのタイプ](../table_design/table_types/table_types.md)で非同期マテリアライズドビューを作成することができます。

- **リフレッシュ**

  非同期マテリアライズドビューを作成すると、そのデータは作成時点でのベーステーブルの状態のみを反映しています。ベーステーブルのデータが変更された場合、変更を同期させるためにマテリアライズドビューをリフレッシュする必要があります。

  現在、StarRocksは次の2つの一般的なリフレッシュ戦略をサポートしています。

  - ASYNC: 非同期リフレッシュモード。ベーステーブルのデータが変更されるたびに、事前に定義されたリフレッシュ間隔に従ってマテリアライズドビューが自動的にリフレッシュされます。
  - MANUAL: 手動リフレッシュモード。マテリアライズドビューは自動的にリフレッシュされません。リフレッシュタスクを手動でトリガーすることしかできません。

- **クエリリライト**

  クエリリライトとは、マテリアライズドビューが構築されているベーステーブル上でクエリを実行する際に、システムが自動的にマテリアライズドビューの事前計算結果をクエリのために再利用できるかどうかを判断し、再利用できる場合は計算や結合に時間とリソースを消費せずに関連するマテリアライズドビューからデータを直接ロードします。

  StarRocksは、v2.5以降、SPJGタイプの非同期マテリアライズドビューに基づいて自動的で透過的なクエリリライトをサポートしています。SPJGタイプのマテリアライズドビューとは、プランにスキャン、フィルタ、プロジェクト、および集計のタイプの演算子のみを含むマテリアライズドビューを指します。

  > **注意**
  >
  > JDBCカタログ上のベーステーブルに作成された非同期マテリアライズドビューは、クエリのリライトをサポートしていません。

## マテリアライズドビューを作成するタイミングを決定する

データウェアハウス環境で以下の要求を持つ場合、非同期マテリアライズドビューを作成することができます。

- **繰り返しのある集計関数によるクエリの高速化**

  データウェアハウスのほとんどのクエリに、集計関数を含む同じサブクエリが含まれており、これらのクエリが計算リソースの大部分を消費している場合を想定してください。このサブクエリに基づいて、サブクエリのすべての結果を計算して保存する非同期マテリアライズドビューを作成することができます。マテリアライズドビューが構築されると、サブクエリを含むすべてのクエリをリライトし、マテリアライズドビューに格納されている中間結果をロードしてクエリを高速化します。

- **複数テーブルの定期的な結合**

  データウェアハウスで複数のテーブルを定期的に結合して新しいワイドテーブルを作成する必要がある場合を想定してください。これらのテーブルのために非同期マテリアライズドビューを構築し、固定時間間隔でリフレッシュタスクをトリガーするASYNCのリフレッシュ戦略を設定することができます。マテリアライズドビューが構築された後、クエリの結果はマテリアライズドビューから直接返されるため、JOIN操作によるレイテンシが回避されます。

- **データウェアハウスのレイヤリング**

  データウェアハウスには大量の生データが含まれており、それに対するクエリには複雑なETL操作が必要な場合を想定してください。データウェアハウスのデータを階層化するために、複数の層からなる非同期マテリアライズドビューを構築することができます。これにより、クエリを複数の単純なサブクエリに分解することができ、繰り返し計算を大幅に削減することができます。さらに重要なことは、データウェアハウスのレイヤリングが、生データと統計データを切り離し、機密性の高い生データを保護するのに役立つことです。

- **データレイクのクエリ高速化**

  データレイクへのクエリは、ネットワークの遅延やオブジェクトストレージのスループットのために遅くなる場合があります。データレイクの上に非同期マテリアライズドビューを構築することで、クエリのパフォーマンスを向上させることができます。さらに、StarRocksはクエリを知的にリライトし、既存のマテリアライズドビューを使用するようにしますので、クエリを手動で変更する手間を省くことができます。

非同期マテリアライズドビューの具体的な使用例については、次のコンテンツを参照してください：

- [データモデリング](./data_modeling_with_materialized_views.md)
- [クエリリライト](./query_rewrite_with_materialized_views.md)
- [データレイクのクエリ高速化](./data_lake_query_acceleration_with_materialized_views.md)

## 非同期マテリアライズドビューを作成する

StarRocksの非同期マテリアライズドビューは、次のベーステーブル上に作成することができます：

- StarRocksのネイティブテーブル（すべてのStarRocksテーブルタイプがサポートされています）

- 外部カタログのテーブル（次のカタログがサポートされています）

  - Hiveカタログ（v2.5以降）
  - Hudiカタログ（v2.5以降）
  - Icebergカタログ（v2.5以降）
  - JDBCカタログ（v3.0以降）

- 既存の非同期マテリアライズドビュー（v2.5以降）
- 既存のビュー（v3.1以降）

### 開始する前に

以下の例では、デフォルトカタログの2つのベーステーブルが関係しています：

- テーブル`goods`は、商品ID `item_id1`、商品名 `item_name`、商品価格 `price`を記録しています。
- テーブル`order_list`は、注文ID `order_id`、クライアントID `client_id`、商品ID `item_id2`、注文日 `order_date`を記録しています。

カラム`goods.item_id1`は、カラム`order_list.item_id2`と同等です。

以下のステートメントを実行して、テーブルを作成し、データを挿入します：

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
（10002,103,1003,"2022-03-14"),
    （10003,102,1003,"2022-03-14"),
    (10003,102,1001,"2022-03-14");
```

下面的示例中的场景要求对每个订单的总金额进行频繁的计算。它需要频繁地连接两个基本表并强调使用聚合函数 `sum()`。此外，业务场景要求每天刷新一次数据。

查询语句如下所示：

```SQL
SELECT
    order_id,
    sum(goods.price) as total
FROM order_list INNER JOIN goods ON goods.item_id1 = order_list.item_id2
GROUP BY order_id;
```

### 创建物化视图

可以使用 [CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md) 根据特定的查询语句创建物化视图。

基于表 `goods`、`order_list` 和上述的查询语句，以下示例创建物化视图 `order_mv` 来分析每个订单的总金额。该物化视图设置为每天刷新一次。

```SQL
CREATE MATERIALIZED VIEW order_mv
DISTRIBUTED BY HASH（`order_id`）
REFRESH ASYNC START（'2022-09-01 10:00:00'） EVERY （interval 1 day）
AS SELECT
    order_list.order_id,
    sum(goods.price) as total
FROM order_list INNER JOIN goods ON goods.item_id1 = order_list.item_id2
GROUP BY order_id;
```

> **注意**
>
> - 在创建异步物化视图时，必须指定物化视图的数据分布策略、刷新策略或两者都指定。
> - 您可以为异步物化视图设置与其基本表不同的分区和哈希策略，但是在用于创建物化视图的查询语句中需要包括物化视图的分区键和哈希键。
> - 异步物化视图支持更长时间跨度的动态分区策略。例如，如果基本表按一天的间隔进行分区，则可以将物化视图分区设置为一个月的间隔。
> - 目前，StarRocks 不支持使用列表分区策略创建异步物化视图，也不支持基于使用列表分区策略创建的表创建异步物化视图。
> - 用于创建物化视图的查询语句不支持随机函数，包括 rand()、random()、uuid() 和 sleep()。
> - 异步物化视图支持多种数据类型。有关更多信息，请参见 [CREATE MATERIALIZED VIEW - 支持的数据类型](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md#supported-data-types)。
> - 默认情况下，执行 CREATE MATERIALIZED VIEW 语句会立即触发刷新任务，这可能会消耗一定比例的系统资源。如果要推迟刷新任务，可以在 CREATE MATERIALIZED VIEW 语句中添加 REFRESH DEFERRED 参数。

- **关于异步物化视图的刷新机制**

  目前，StarRocks 支持两种按需刷新策略：手动刷新和异步刷新。

  在 StarRocks v2.5 中，异步物化视图进一步支持多种异步刷新机制，用于控制刷新成本并提高成功率：

  - 如果一个物化视图有很多大分区，每次刷新可能会消耗大量的资源。在 v2.5 中，StarRocks 支持拆分刷新任务。您可以指定要刷新的分区的最大数量，StarRocks 会分批进行刷新，每个批次的大小小于或等于指定的最大分区数量。这个特性保证了大型异步物化视图的稳定刷新，增强了数据建模的稳定性和鲁棒性。
  - 您可以为异步物化视图的分区指定存活时间（TTL），减小物化视图占用的存储空间。
  - 您可以指定刷新范围，仅刷新最新的几个分区，减小刷新开销。
  - 您可以指定当数据发生更改时不自动触发相应物化视图刷新的基础表。
  - 您可以为刷新任务分配资源组。

  有关更多信息，请参见 [CREATE MATERIALIZED VIEW - 参数](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md#parameters) 中的 **PROPERTIES** 部分。您还可以使用 [ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md) 修改现有异步物化视图的机制。

  > **注意**
  >
  > 为了防止全刷新操作耗尽系统资源并导致任务失败，建议基于具有分区的基本表创建分区的物化视图。这样，当基本表分区内发生数据更新时，只会刷新相应分区的物化视图，而不是刷新整个物化视图。有关更多信息，请参阅[使用物化视图进行数据建模 - 分区建模](./data_modeling_with_materialized_views.md#partitioned-modeling)。

- **关于嵌套物化视图**

  StarRocks v2.5 支持创建嵌套的异步物化视图。您可以基于现有的异步物化视图构建异步物化视图。每个物化视图的刷新策略不影响上层或下层物化视图。目前，StarRocks 对嵌套层次的数量没有限制。在生产环境中，建议嵌套层数不要超过三层。

- **关于外部目录物化视图**

  StarRocks 支持基于 Hive 目录（自 v2.5）、Hudi 目录（自 v2.5）、Iceberg 目录（自 v2.5）和 JDBC 目录（自 v3.0）创建异步物化视图。在外部目录上创建物化视图与在默认目录上创建异步物化视图类似，但有一些使用限制。有关更多信息，请参阅 [在数据湖上加速查询](./data_lake_query_acceleration_with_materialized_views.md)。

## 手动刷新异步物化视图

可以通过 [REFRESH MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/REFRESH_MATERIALIZED_VIEW.md) 来刷新异步物化视图，无论其刷新策略如何。StarRocks v2.5 支持通过指定分区名称来刷新异步物化视图的特定分区。StarRocks v3.1 支持同步调用刷新任务，只有当任务成功或失败时才返回 SQL 语句。

```SQL
-- 通过异步调用刷新物化视图（默认）。
REFRESH MATERIALIZED VIEW order_mv;
-- 通过同步调用刷新物化视图。
REFRESH MATERIALIZED VIEW order_mv WITH SYNC MODE;
```

可以使用 [CANCEL REFRESH MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/CANCEL_REFRESH_MATERIALIZED_VIEW.md) 取消通过异步调用提交的刷新任务。

## 直接查询异步物化视图

您创建的异步物化视图本质上是一个包含根据查询语句预先计算结果的完整物理表。因此，在物化视图第一次刷新之后，可以直接查询物化视图。

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
> 您可以直接查询异步物化视图，但是结果可能与对其基本表进行查询时获得的结果不一致。

## 使用异步物化视图重写和加速查询

StarRocks v2.5 支持基于 SPJG 类型的异步物化视图进行自动和透明的查询重写。SPJG 类型物化视图的查询重写包括单表查询重写、联接查询重写、聚合查询重写、Union 查询重写以及基于嵌套物化视图的查询重写。有关更多信息，请参阅[使用物化视图进行查询重写](./query_rewrite_with_materialized_views.md)。

目前，StarRocks 支持重写对默认目录或外部目录（如 Hive 目录、Hudi 目录或 Iceberg 目录）上的异步物化视图查询的查询。在查询默认目录中的数据时，StarRocks 通过排除其数据与基本表数据不一致的物化视图来确保重写的查询与原始查询的结果具有强一致性。物化视图的数据过期后，不会将物化视图用作候选物化视图。在查询外部目录中的数据时，StarRocks 不能保证结果的强一致性，因为 StarRocks 无法感知外部目录中的数据更改。有关基于外部目录创建的异步物化视图的更多信息，请参阅[在数据湖上加速查询](./data_lake_query_acceleration_with_materialized_views.md)。

> **注意**
>
> 在 JDBC 目录的基本表上创建的异步物化视图不支持查询重写。

## 管理异步物化视图

### 修改异步物化视图
非同期マテリアライズド ビューのプロパティを変更する場合は、[ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md)を使用します。

- 無効な非同期マテリアライズド ビューを有効にする。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv ACTIVE;
  ```

- 非同期マテリアライズド ビューの名前を変更する。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv RENAME order_total;
  ```

- 非同期マテリアライズド ビューのリフレッシュ間隔を2日に変更する。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv REFRESH ASYNC EVERY(INTERVAL 2 DAY);
  ```

### 非同期マテリアライズド ビューの表示

[SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)を使用するか、Information Schema のシステムメタデータビューをクエリすることで、データベース内の非同期マテリアライズド ビューを表示できます。

- データベース内のすべての非同期マテリアライズド ビューを確認する。

  ```SQL
  SHOW MATERIALIZED VIEWS;
  ```

- 特定の非同期マテリアライズド ビューを確認する。

  ```SQL
  SHOW MATERIALIZED VIEWS WHERE NAME = "order_mv";
  ```

- 名前を一致させて特定の非同期マテリアライズド ビューを確認する。

  ```SQL
  SHOW MATERIALIZED VIEWS WHERE NAME LIKE "order%";
  ```

- Information Schema の `materialized_views` メタデータビューをクエリすることで、データベース内のすべての非同期マテリアライズド ビューを確認できます。詳細については [information_schema.materialized_views](../reference/information_schema/materialized_views.md) を参照してください。

  ```SQL
  SELECT * FROM information_schema.materialized_views;
  ```

### 非同期マテリアライズド ビューの定義を確認する

非同期マテリアライズド ビューを作成する際に使用されたクエリは、[SHOW CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_MATERIALIZED_VIEW.md)を使用して確認できます。

```SQL
SHOW CREATE MATERIALIZED VIEW order_mv;
```

### 非同期マテリアライズド ビューの実行状況を確認する

非同期マテリアライズド ビューの実行（構築またはリフレッシュ）状況は、[Information Schema](../reference/overview-pages/information_schema.md) の `tasks` および `task_runs` をクエリすることで確認できます。

以下の例は、最も最近作成されたマテリアライズド ビューの実行状況を確認するものです:

1. `tasks` テーブルで最新のタスクの `TASK_NAME` を確認します。

    ```Plain
    mysql> select * from information_schema.tasks  order by CREATE_TIME desc limit 1\G;
    *************************** 1. row ***************************
      TASK_NAME: mv-59299
    CREATE_TIME: 2022-12-12 17:33:51
      SCHEDULE: MANUAL
      DATABASE: ssb_1
    DEFINITION: ･･･
    FROM `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`
    WHERE `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderdate` = '1997-01-01'
    EXPIRE_TIME: NULL
    1 row in set (0.02 sec)
    ```

2. 見つけた `TASK_NAME` を使用して、 `task_runs` テーブルで実行状況を確認します。

    ```Plain
    mysql> select * from information_schema.task_runs where task_name='mv-59299' order by CREATE_TIME \G;
    *************************** 1. row ***************************
        QUERY_ID: ･･･
        TASK_NAME: mv-59299
      CREATE_TIME: 2022-12-12 17:39:19
      FINISH_TIME: 2022-12-12 17:39:22
            STATE: SUCCESS
        DATABASE: ssb_1
      DEFINITION: ･･･
    FROM `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`
    WHERE `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderdate` = '1997-01-01'
      EXPIRE_TIME: 2022-12-15 17:39:19
      ERROR_CODE: 0
    ERROR_MESSAGE: NULL
        PROGRESS: 100%
    2 rows in set (0.02 sec)
    ```

### 非同期マテリアライズド ビューを削除する

非同期マテリアライズド ビューは、[DROP MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/DROP_MATERIALIZED_VIEW.md)を使用して削除できます。

```Plain
DROP MATERIALIZED VIEW order_mv;
```

### 関連するセッション変数

以下の変数が非同期マテリアライズド ビューの動作を制御します:

- `analyze_mv`: リフレッシュ後にマテリアライズド ビューを分析するかどうか。有効な値は空文字列（分析を行わない）、`sample`（サンプリングされた統計情報の収集）、`full`（完全な統計情報の収集）です。デフォルトは `sample` です。
- `enable_materialized_view_rewrite`: マテリアライズド ビューの自動リライトを有効にするかどうか。有効な値は `true`（v2.5 以降のデフォルト）および `false` です。