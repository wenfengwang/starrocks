---
displayed_sidebar: English
---

# 异步物化视图

本主题描述了如何理解、创建、使用和管理异步物化视图。StarRocks v2.4 之后支持异步物化视图。

与同步物化视图相比，异步物化视图支持多表连接和更多的聚合函数。异步物化视图的刷新可以手动触发，也可以通过计划任务触发。您还可以选择刷新部分分区，而不是整个物化视图，大大降低刷新成本。此外，异步物化视图支持各种查询重写场景，实现自动、透明的查询加速。

有关同步物化视图（Rollup）的场景和用法，请参阅[同步物化视图（Rollup）](../using_starrocks/Materialized_view-single_table.md)。

## 概述

数据库中的应用程序经常对大型表执行复杂查询。这些查询涉及多表连接和对包含数十亿行的表进行聚合。处理这些查询可能会消耗大量系统资源，并且计算结果所需的时间也很长。

StarRocks 中的异步物化视图旨在解决这些问题。异步物化视图是一个特殊的物理表，用于保存来自一个或多个基表的预先计算的查询结果。当您对基表执行复杂查询时，StarRocks 会返回相关物化视图中的预先计算结果，以处理这些查询。这样一来，由于避免了重复的复杂计算，查询性能得以提高。当查询频繁运行或足够复杂时，这种性能差异可能会很大。

此外，异步物化视图特别适用于在数据仓库上构建数学模型。通过这种方式，您可以为上层应用提供统一的数据规范，屏蔽底层实现，或者保护基表的原始数据安全。

### 了解 StarRocks 中的物化视图

StarRocks v2.3 及更早版本提供了同步物化视图，只能构建在单个表上。同步物化视图（即Rollup）保留了更高的数据新鲜度和更低的刷新成本。但是，与 v2.4 之后支持的异步物化视图相比，同步物化视图有许多限制。当您想要构建同步物化视图以加速或重写查询时，聚合函数的选择受到限制。

下表从功能支持的角度对比了StarRocks中的异步物化视图（ASYNC MV）和同步物化视图（SYNC MV）：

|                       | **单表聚合** | **多表连接** | **查询重写** | **刷新策略** | **基表** |
| --------------------- | ---------------------------- | -------------------- | ----------------- | -------------------- | -------------- |
| **ASYNC MV** | 是 | 是 | 是 | <ul><li>异步刷新</li><li>手动刷新</li></ul> | 来自以下多个表：<ul><li>默认目录</li><li>外部目录（v2.5）</li><li>现有物化视图（v2.5）</li><li>现有视图（v3.1）</li></ul> |
| **SYNC MV（Rollup）**  | 聚合函数选择有限 | 否 | 是 | 数据加载期间的同步刷新 | 默认目录中的单个表 |

### 基本概念

- **基表**

  基表是物化视图的驱动表。

  对于StarRocks的异步物化视图，基表可以是[默认目录](../data_source/catalog/default_catalog.md)中的StarRocks原生表，也可以是外部目录中的表（从v2.5开始支持），甚至可以是已有的异步物化视图（从v2.5开始支持）和视图（从v3.1开始支持）。StarRocks支持在所有[类型的StarRocks表](../table_design/table_types/table_types.md)上创建异步物化视图。

- **刷新**

  创建异步物化视图时，其数据仅反映基表当时的状态。当基表中的数据发生更改时，需要刷新物化视图以保持更改同步。

  目前，StarRocks支持两种通用的刷新策略：

  - ASYNC：异步刷新模式。每次基表数据发生变化时，物化视图都会根据预定义的刷新间隔自动刷新。
  - MANUAL：手动刷新模式。物化视图不会自动刷新，刷新任务只能由用户手动触发。

- **查询重写**

  查询重写是指在对基表进行查询时，系统会自动判断物化视图中预先计算的结果是否可以重用于查询。如果可以重用，系统将直接从相关的物化视图加载数据，避免耗时和资源的计算或连接。

  从v2.5开始，StarRocks支持基于SPJG类型的异步物化视图进行自动、透明的查询重写。SPJG类型的物化视图是指其计划仅包含Scan、Filter、Project和Aggregate类型的算子的物化视图。

  > **注意**
  >
  > 在[JDBC目录](../data_source/catalog/jdbc_catalog.md)中创建的异步物化视图不支持查询重写。

## 确定何时创建物化视图

如果您的数据仓库环境中存在以下需求，则可以创建异步物化视图：

- **加速具有重复聚合函数的查询**

  假设数据仓库中的大多数查询都包含相同的子查询和聚合函数，并且这些查询消耗了大量计算资源。基于这个子查询，您可以创建一个异步物化视图，该视图将计算和存储子查询的所有结果。物化视图构建完成后，StarRocks会重写所有包含子查询的查询，并加载存储在物化视图中的中间结果，从而加速这些查询。

- **定期联接多个表**

  假设您需要定期联接数据仓库中的多个表，以创建新的宽表。您可以为这些表构建异步物化视图，并设置以固定时间间隔触发刷新任务的ASYNC刷新策略。物化视图构建完成后，查询结果直接从物化视图返回，避免了JOIN操作带来的延迟。

- **数据仓库分层**

  假设您的数据仓库包含大量原始数据，并且其中的查询需要一系列复杂的ETL操作。您可以构建多层异步物化视图，对数据仓库中的数据进行分层，从而将查询分解为一系列简单的子查询。这可以显著减少重复计算，更重要的是，它可以帮助您的DBA轻松高效地识别问题。此外，数据仓库分层有助于解耦原始数据和统计数据，保护敏感原始数据的安全。

- **加速数据湖中的查询**

  由于网络延迟和对象存储吞吐量，查询数据湖的速度可能会很慢。您可以通过在数据湖上构建异步物化视图来增强查询性能。此外，StarRocks还可以智能地重写查询，使用已有的物化视图，省去您手动修改查询的麻烦。

有关异步物化视图的具体用例，请参阅以下内容：

- [数据建模](./data_modeling_with_materialized_views.md)
- [查询重写](./query_rewrite_with_materialized_views.md)
- [数据湖查询加速](./data_lake_query_acceleration_with_materialized_views.md)

## 创建异步物化视图

StarRocks的异步物化视图可以在以下基表上创建：

- StarRocks原生表（支持所有StarRocks表类型）

- 外部目录中的表，包括

  - Hive目录（自v2.5起）
  - Hudi目录（自v2.5起）
  - Iceberg目录（自v2.5起）
  - JDBC目录（自v3.0起）

- 现有的异步物化视图（自 v2.5 起）
- 现有视图（自 v3.1 起）

### 开始之前

以下示例涉及默认目录中的两个基本表：

- 表 `goods` 记录了项目 ID `item_id1`、项目名称 `item_name` 和项目价格 `price`。
- 表 `order_list` 记录了订单 ID `order_id`、客户端 ID `client_id`、项目 ID `item_id2` 和订单日期 `order_date`。

`goods.item_id1` 列等同于 `order_list.item_id2` 列。

执行以下语句以创建这些表并向其中插入数据：

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

以下示例中的场景需要频繁计算每个订单的总额。它要求频繁连接这两个基本表，并大量使用聚合函数 `sum()`。此外，业务场景要求每隔一天刷新一次数据。

查询语句如下：

```SQL
SELECT
    order_id,
    sum(goods.price) as total
FROM order_list INNER JOIN goods ON goods.item_id1 = order_list.item_id2
GROUP BY order_id;
```

### 创建物化视图

您可以使用 [CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md) 基于特定查询语句创建物化视图。

基于表 `goods`、`order_list` 和上述查询语句，以下示例创建物化视图 `order_mv` 以分析每个订单的总额。物化视图设置为每隔一天刷新一次。

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
> - 创建异步物化视图时，必须指定物化视图的数据分布策略、刷新策略，或两者都要指定。
> - 您可以为异步物化视图设置不同的分区和存储桶策略，但必须在用于创建物化视图的查询语句中包含物化视图的分区键和存储桶键。
> - 异步物化视图支持更长时间跨度内的动态分区策略。例如，如果基表按天分区，您可以将物化视图设置为按月分区。
> - 目前，StarRocks 不支持使用列表分区策略创建异步物化视图，也不支持基于列表分区策略创建的表创建异步物化视图。
> - 用于创建物化视图的查询语句不支持随机函数，包括 rand()、random()、uuid() 和 sleep()。
> - 异步物化视图支持多种数据类型。有关详细信息，请参阅 [CREATE MATERIALIZED VIEW - 支持的数据类型](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md#supported-data-types)。
> - 默认情况下，执行 CREATE MATERIALIZED VIEW 语句会立即触发刷新任务，这可能会消耗一定比例的系统资源。如果要推迟刷新任务，可以在 CREATE MATERIALIZED VIEW 语句中添加 REFRESH DEFERRED 参数。

- **关于异步物化视图的刷新机制**

  目前，StarRocks 支持两种 ON DEMAND 刷新策略：MANUAL 刷新和 ASYNC 刷新。

  在 StarRocks v2.5 中，异步物化视图进一步支持多种异步刷新机制，以控制刷新成本，提高刷新成功率：

  - 如果物化视图有许多大型分区，每次刷新都会消耗大量资源。在 v2.5 版本中，StarRocks 支持拆分刷新任务。您可以指定要刷新的最大分区数，StarRocks 会以批处理方式进行刷新，批处理大小小于或等于指定的最大分区数。该功能确保大型异步物化视图能够稳定刷新，增强数据建模的稳定性和鲁棒性。
  - 您可以指定异步物化视图分区的生存时间（TTL），从而减小物化视图占用的存储空间。
  - 您可以指定刷新范围，仅刷新最近的几个分区，从而减少刷新开销。
  - 您可以指定数据更改不会自动触发相应物化视图的刷新。
  - 您可以为刷新任务分配资源组。

  有关详细信息，请参阅 [CREATE MATERIALIZED VIEW - Parameters](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md#parameters) 中的 **PROPERTIES** 部分。您还可以使用 [ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md) 修改现有异步物化视图的机制。

  > **注意**
  >
  > 为避免全量刷新操作耗尽系统资源，导致任务失败，建议基于分区基表创建分区物化视图。这可确保在基表分区内发生数据更新时，仅刷新物化视图的相应分区，而不是刷新整个物化视图。有关详细信息，请参阅 [使用物化视图进行数据建模 - 分区建模](./data_modeling_with_materialized_views.md#partitioned-modeling)。

- **关于嵌套异步物化视图**

  StarRocks v2.5 支持创建嵌套的异步物化视图。您可以基于现有的异步物化视图构建新的异步物化视图。每个物化视图的刷新策略不会影响上层或下层的物化视图。目前，StarRocks 不限制嵌套层数。在生产环境中，建议嵌套层数不要超过三层。

- **关于外部目录物化视图**

  StarRocks 支持基于 Hive Catalog（自 v2.5 及以上）、Hudi Catalog（自 v2.5 及以上）、Iceberg Catalog（自 v2.5 及以上）和 JDBC Catalog（自 v3.0 起）构建异步物化视图。在外部目录上创建物化视图类似于在默认目录上创建异步物化视图，但有一些使用限制。更多信息，请参考 [使用物化视图进行数据湖查询加速](./data_lake_query_acceleration_with_materialized_views.md)。

## 手动刷新异步物化视图

您可以通过 [REFRESH MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/REFRESH_MATERIALIZED_VIEW.md) 刷新异步物化视图，而不考虑其刷新策略。StarRocks v2.5 支持通过指定分区名称来刷新异步物化视图的特定分区。StarRocks v3.1 支持以同步方式调用刷新任务，只有在任务成功或失败时才会返回 SQL 语句。

```SQL
-- 通过异步调用刷新物化视图（默认方式）。
REFRESH MATERIALIZED VIEW order_mv;
-- 通过同步调用刷新物化视图。
REFRESH MATERIALIZED VIEW order_mv WITH SYNC MODE;
```

您可以使用 [CANCEL REFRESH MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/CANCEL_REFRESH_MATERIALIZED_VIEW.md) 取消通过异步调用提交的刷新任务。

## 直接查询异步物化视图


您创建的异步具体化视图实质上是一个物理表，其中包含根据查询语句进行预计算结果的完整集。因此，在第一次刷新后，您可以直接查询具体化视图。

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
> 您可以直接查询异步具体化视图，但结果可能与从其基表上的查询中获得的结果不一致。

## 使用异步具体化视图重写和加速查询

StarRocks v2.5 支持基于 SPJG 类型的异步物化视图进行自动透明的查询改写。SPJG类型的物化视图查询重写包括单表查询重写、Join查询重写、聚合查询重写、联合查询重写和基于嵌套物化视图的查询重写。有关详细信息，请参阅 [使用物化视图进行查询重写](./query_rewrite_with_materialized_views.md)。

目前，StarRocks支持对默认目录或外部目录（如Hive目录、Hudi目录、Iceberg目录）创建的异步物化视图进行查询重写。在查询默认目录下的数据时，StarRocks会剔除数据与基表不一致的物化视图，从而保证重写查询与原查询结果的强一致性。当物化视图中的数据过期时，该物化视图将不再用作候选物化视图。查询外部目录下的数据时，由于StarRocks无法感知到外部目录的数据变化，因此无法保证结果的强一致性。关于基于外部目录创建的异步物化视图，请参见[使用物化视图加速数据湖查询。](./data_lake_query_acceleration_with_materialized_views.md)

> **注意**
>
> 在JDBC目录中的基表上创建的异步物化视图不支持查询重写。

## 管理异步具体化视图

### 更改异步具体化视图

您可以使用[ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md)更改异步具体化视图的属性。

- 启用非活动具体化视图。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv ACTIVE;
  ```

- 重命名异步具体化视图。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv RENAME order_total;
  ```

- 将异步具体化视图的刷新间隔更改为2天。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv REFRESH ASYNC EVERY(INTERVAL 2 DAY);
  ```

### 显示异步具体化视图

您可以通过[SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)或在Information Schema中查询系统元数据视图来查看数据库中的异步具体化视图。

- 检查数据库中的所有异步具体化视图。

  ```SQL
  SHOW MATERIALIZED VIEWS;
  ```

- 检查特定的异步具体化视图。

  ```SQL
  SHOW MATERIALIZED VIEWS WHERE NAME = "order_mv";
  ```

- 通过匹配名称来检查特定的异步具体化视图。

  ```SQL
  SHOW MATERIALIZED VIEWS WHERE NAME LIKE "order%";
  ```

- 通过查询Information Schema中的元数据视图来检查所有异步物化视图`materialized_views`。更多信息请参考[information_schema.materialized_views](../reference/information_schema/materialized_views.md)。

  ```SQL
  SELECT * FROM information_schema.materialized_views;
  ```

### 检查异步物化视图的定义

您可以通过[SHOW CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_MATERIALIZED_VIEW.md)检查用于创建异步物化视图的查询。

```SQL
SHOW CREATE MATERIALIZED VIEW order_mv;
```

### 检查异步物化视图的执行状态

您可以通过查询Information Schema中的[`tasks`](../reference/information_schema/tasks.md)和[`task_runs`](../reference/information_schema/task_runs.md)来检查异步物化视图的执行（构建或刷新）状态。

以下示例检查最近创建的物化视图的执行状态：

1. 检查`tasks`表中最新任务的`TASK_NAME`。

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

2. 使用您找到的`task_runs`检查`TASK_NAME`表中的执行状态。

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

### 删除异步具体化视图

您可以通过[DROP MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/DROP_MATERIALIZED_VIEW.md)删除异步具体化视图。

```Plain

DROP MATERIALIZED VIEW order_mv;
```

### 相关会话变量

以下变量控制异步物化视图的行为：

- `analyze_mv`：在刷新后是否以及如何分析物化视图。有效值为空字符串（不进行分析）、`sample`（采样统计收集）和 `full`（完整统计收集）。默认为 `sample`。
- `enable_materialized_view_rewrite`：是否启用物化视图的自动重写。有效值为 `true`（自 v2.5 起为默认值）和 `false`。