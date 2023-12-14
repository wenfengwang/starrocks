---
displayed_sidebar: "Chinese"
---

# 异步物化视图

本主题描述了如何了解、创建、使用和管理异步物化视图。StarRocks v2.4及更高版本支持异步物化视图。

与同步物化视图相比，异步物化视图支持多表连接和更多的聚合函数。异步物化视图的刷新可以通过手动触发或定期任务触发。您还可以仅刷新部分分区而不是整个物化视图，从而大大降低刷新成本。此外，异步物化视图支持各种查询重写场景，允许自动透明地加速查询。

有关同步物化视图（Rollup）的场景和用法，请参见[同步物化视图（Rollup）](../using_starrocks/Materialized_view-single_table.md)。

## 概述

数据库中的应用程序经常在大型表上执行复杂查询。这些查询涉及对包含数十亿行的表进行多表连接和聚合。处理这些查询可能会消耗大量的系统资源和计算结果所需的时间。

StarRocks中的异步物化视图旨在解决这些问题。异步物化视图是一个特殊的物理表，它保存了来自一个或多个基本表的预先计算的查询结果。当您对基本表执行复杂查询时，StarRocks返回相关物化视图中的预先计算结果来处理这些查询。这样，查询性能可以得到改善，因为避免了重复的复杂计算。当查询频繁运行或足够复杂时，这种性能差异可能是显著的。

此外，异步物化视图对于在数据仓库上构建数学模型特别有用。通过这样做，您可以为上层应用程序提供统一的数据规范，屏蔽底层实现，或保护基本表的原始数据安全。

### 了解StarRocks中的物化视图

StarRocks v2.3及更早版本提供了仅能在单个表上构建的同步物化视图。同步物化视图（或Rollup）保留了更高的数据新鲜度和更低的刷新成本。但是，与从v2.4开始支持的异步物化视图相比，同步物化视图有许多限制。当您想要构建同步物化视图以加速或重写查询时，聚合函数的选择受到限制。

下表对比了StarRocks中异步物化视图（ASYNC MV）和同步物化视图（SYNC MV）在它们支持的特性的角度：

|                       | **单表聚合** | **多表连接** | **查询重写** | **刷新策略** | **基本表** |
| --------------------- | ------------ | ------------ | ------------ | ------------ | ---------- |
| **ASYNC MV** | 是 | 是 | 是 | <ul><li>异步刷新</li><li>手动刷新</li></ul> | 来自以下多个表：<ul><li>默认目录中的多个表</li><li>外部目录中的表（v2.5）</li><li>现有物化视图（v2.5）</li><li>现有视图（v3.1）</li></ul> |
| **SYNC MV (Rollup)**  | 聚合函数选择受限 | 否 | 是 | 在数据加载期间同步刷新 | 默认目录中的单个表 |

### 基本概念

- **基本表**

  基本表是物化视图的驱动表。

  对于StarRocks的异步物化视图，基本表可以是[默认目录](../data_source/catalog/default_catalog.md)中的StarRocks原生表、外部目录中的表（从v2.5开始支持），甚至是已存在的异步物化视图（从v2.5开始支持）和视图（从v3.1开始支持）。StarRocks支持在所有[类型的StarRocks表](../table_design/table_types/table_types.md)上创建异步物化视图。

- **刷新**

  当您创建一个异步物化视图时，其数据仅反映了基本表在那个时间点的状态。当基本表的数据发生变化时，您需要刷新物化视图以保持变化的同步。

  目前，StarRocks支持两种通用的刷新策略：

  - ASYNC：异步刷新模式。每次基本表数据发生变化时，根据预定义的刷新间隔，自动刷新物化视图。
  - MANUAL：手动刷新模式。物化视图不会自动刷新。刷新任务只能由用户手动触发。

- **查询重写**

  查询重写意味着当在构建了物化视图的基本表上执行查询时，系统会自动判断预先计算结果在物化视图中是否可被重用。如果可以重用，系统将直接从相关物化视图加载数据，避免了计算或连接所需的时间和资源。

  从v2.5开始，StarRocks支持基于SPJG类型的异步物化视图的自动透明查询重写。SPJG类型的物化视图指的是其计划仅包括扫描、过滤、投影和聚合类型的运算符的物化视图。

  > **注意**
  >
  > 在[JDBC目录](../data_source/catalog/jdbc_catalog.md)中创建的异步物化视图不支持查询重写。

## 决定何时创建物化视图

如果在数据仓库环境中有以下需求，您可以创建一个异步物化视图：

- **加速重复聚合函数的查询**

  假设数据仓库中大多数查询包含相同的具有聚合函数的子查询，并且这些查询消耗了大量计算资源。基于这个子查询，您可以创建一个异步物化视图，该视图将计算和存储子查询的所有结果。建立物化视图后，StarRocks会重写包含子查询的所有查询，加载物化视图中存储的中间结果，从而加速这些查询。

- **定期连接多个表**

  假设您需要定期连接数据仓库中的多个表，以创建一个新的宽表。您可以为这些表构建一个异步物化视图，并设置触发刷新任务的ASYNC刷新策略。物化视图建立后，查询结果将直接从物化视图返回，从而避免了连接操作带来的延迟。

- **数据仓库分层**

  假设您的数据仓库包含大量原始数据，并且其中的查询需要复杂的ETL操作。您可以构建多层异步物化视图来将数据分层，从而将查询分解为一系列简单的子查询。这可以显著减少重复计算，并且更重要的是，有助于您的DBA轻松高效地识别问题。此外，数据仓库分层有助于解耦原始数据和统计数据，保护敏感原始数据的安全性。

- **加速数据湖中的查询**

 由于网络延迟和对象存储吞吐量，查询数据湖可能会很慢。您可以通过在数据湖上构建异步物化视图来提高查询性能。此外，StarRocks还可以智能地重写查询以使用已有的物化视图，省去手动修改查询的麻烦。

有关异步物化视图的具体用例，请参考以下内容：

- [数据建模](./data_modeling_with_materialized_views.md)
- [查询重写](./query_rewrite_with_materialized_views.md)
- [数据湖查询加速](./data_lake_query_acceleration_with_materialized_views.md)

## 创建异步物化视图

在StarRocks上可以在以下基本表上创建异步物化视图：

- StarRocks原生表（支持所有StarRocks表类型）

- 外部目录中的表，包括

  - Hive目录（自v2.5起）
  - Hudi目录（自v2.5起）
  - Iceberg目录（自v2.5起）
  - JDBC目录（自v3.0起）

- 已存在的异步物化视图（自v2.5起）
- 已存在的视图（自v3.1起）

### 开始之前

以下示例涉及默认目录中的两个基本表：

- 表`goods`记录了商品ID`item_id1`、商品名称`item_name`和商品价格`price`。
- 表`order_list`记录了订单ID`order_id`、客户ID`client_id`、商品ID`item_id2`和订单日期`order_date`。

列`goods.item_id1`相当于列`order_list.item_id2`。

执行以下语句来创建这些表并向其插入数据：

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

以下示例情境需要对每个订单的总额进行频繁计算。它需要频繁地将两个基本表连接起来，以及频繁使用聚合函数`sum()`。此外，业务情境要求数据需要每隔一天刷新一次。

查询语句如下：

```SQL
SELECT
    order_id,
    sum(goods.price) as total
FROM order_list INNER JOIN goods ON goods.item_id1 = order_list.item_id2
GROUP BY order_id;
```

### 创建物化视图

您可以使用[CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md)基于特定查询语句创建物化视图。

基于表`goods`、`order_list`和上述查询语句，以下示例创建名为`order_mv`的物化视图，以分析每个订单的总额。物化视图设置为每隔一天自动刷新。

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
> - 在创建异步物化视图时，必须指定物化视图的数据分布策略或刷新策略，或两者兼具。
> - 您可以为异步物化视图设置不同的分区和分桶策略，但是在用于创建物化视图的查询语句中，必须包含物化视图的分区键和分桶键。
> - 异步物化视图支持较长时间跨度的动态分区策略。例如，如果基表以一天为间隔进行分区，您可以将物化视图设置为一个月一次的分区。
> - 目前，StarRocks不支持在基于列表分区策略或者基于使用列表分区策略创建的表的情况下创建异步物化视图。
> - 用于创建物化视图的查询语句不支持随机函数，包括rand()、random()、uuid()和sleep()。
> - 异步物化视图支持多种数据类型。了解更多信息，请参阅[CREATE MATERIALIZED VIEW - 支持的数据类型](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md#supported-data-types)。
> - 默认执行CREATE MATERIALIZED VIEW语句会立即触发刷新任务，这可能会占用系统资源的一定比例。如果要推迟刷新任务，可以向CREATE MATERIALIZED VIEW语句添加REFRESH DEFERRED参数。

- **关于异步物化视图的刷新机制**

  目前，StarRocks支持两种ON DEMAND刷新策略：手动刷新和异步刷新。

  在StarRocks v2.5中，异步物化视图进一步支持多种异步刷新机制，以控制刷新成本并提高成功率：

  - 如果一个物化视图有许多大的分区，每个刷新可能消耗大量资源。在v2.5中，StarRocks支持拆分刷新任务。您可以指定要刷新的分区的最大数量，StarRocks会分批进行刷新，每个批次的大小小于或等于指定的最大分区数。此功能确保大型异步物化视图能够稳定刷新，增强了数据建模的稳定性和健壮性。
  - 您可以为异步物化视图的分区指定生存时间（TTL），以减少物化视图占用的存储空间。
  - 您可以指定刷新范围，只刷新最新的几个分区，减少刷新开销。
  - 您可以指定当数据变化时不会自动触发相应物化视图刷新的基表。
  - 您可以将资源组分配给刷新任务。

  了解更多信息，请参阅[CREATE MATERIALIZED VIEW - 参数](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md#parameters)中的**PROPERTIES**章节。您也可以使用[ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md)修改现有异步物化视图的机制。

  > **注意**
  >
  > 为了防止完全刷新操作耗尽系统资源并导致任务失败，建议基于分区基表创建分区物化视图。这样当基表分区内发生数据更新时，只会刷新相应分区的物化视图，而不是刷新整个物化视图。了解更多信息，请参阅[使用物化视图进行数据建模 - 分区建模](./data_modeling_with_materialized_views.md#partitioned-modeling)。

- **关于嵌套的物化视图**

  StarRocks v2.5支持创建嵌套的异步物化视图。您可以基于现有的异步物化视图构建新的异步物化视图。每个物化视图的刷新策略不会影响其上层或下层的物化视图。目前，StarRocks不限制嵌套层级的数量。在生产环境中，建议嵌套层级的数量不要超过三层。

- **关于外部目录物化视图**

  StarRocks支持基于Hive目录（自v2.5起）、Hudi目录（自v2.5起）、Iceberg目录（自v2.5起）和JDBC目录（自v3.0起）构建异步物化视图。创建外部目录物化视图与在默认目录上创建异步物化视图类似，但存在一些使用限制。了解更多信息，请参阅[使用物化视图加速数据湖查询](./data_lake_query_acceleration_with_materialized_views.md)。

## 手动刷新异步物化视图

您可以使用[REFRESH MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/REFRESH_MATERIALIZED_VIEW.md)刷新异步物化视图，无论其刷新策略如何。StarRocks v2.5支持通过指定分区名称刷新异步物化视图的特定分区。StarRocks v3.1支持同步调用刷新任务，只有当任务成功或失败时才返回SQL语句。

```SQL
-- 通过异步调用（默认方式）刷新物化视图。
REFRESH MATERIALIZED VIEW order_mv;
-- 通过同步调用刷新物化视图。
REFRESH MATERIALIZED VIEW order_mv WITH SYNC MODE;
```

您可以使用[CANCEL REFRESH MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/CANCEL_REFRESH_MATERIALIZED_VIEW.md)取消通过异步调用提交的刷新任务。

## 直接查询异步物化视图

您创建的异步物化视图实质上是一个物理表，其中包含根据查询语句完整预先计算结果的数据集。因此，在物化视图首次刷新后，您可以直接查询物化视图。

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
> 您可以直接查询异步物化视图，但结果可能与对其基本表进行查询的结果存在不一致。

## 使用异步物化视图重写和加速查询

StarRocks v2.5支持基于SPJG类型的异步物化视图进行自动透明的查询重写。SPJG类型物化视图的查询重写包括单表查询重写、连接查询重写、聚合查询重写、Union查询重写，以及基于嵌套物化视图的查询重写。了解更多信息，请参阅[使用物化视图进行查询重写](./query_rewrite_with_materialized_views.md)。

目前，StarRocks支持对在默认目录或外部目录（如Hive目录、Hudi目录或Iceberg目录）上创建的异步物化视图进行查询重写。在查询默认目录的数据时，StarRocks通过排除数据与基表不一致的物化视图来确保查询重写的结果具有强一致性。当物化视图的数据过期时，将不会使用该物化视图作为候选物化视图。在查询外部目录的数据时，由于StarRocks无法感知外部目录中的数据变化，所以不保证结果的强一致性。了解更多关于基于外部目录的异步物化视图的信息，请参阅[使用物化视图加速数据湖查询](./data_lake_query_acceleration_with_materialized_views.md)。

> **注意**
>
> 在JDBC目录中基表上创建的异步物化视图不支持查询重写。

## 管理异步物化视图

### 修改异步物化视图

你可以使用[ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md)来更改异步物化视图的属性。

- 启用一个非活动的物化视图。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv ACTIVE;
  ```

- 重命名一个异步物化视图。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv RENAME order_total;
  ```

- 将异步物化视图的刷新间隔更改为2天。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv REFRESH ASYNC EVERY(INTERVAL 2 DAY);
  ```

### 显示异步物化视图

您可以通过使用[SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)或查询信息模式中的系统元数据视图来查看数据库中的异步物化视图。

- 检查数据库中的所有异步物化视图。

  ```SQL
  SHOW MATERIALIZED VIEWS;
  ```

- 检查特定的异步物化视图。

  ```SQL
  SHOW MATERIALIZED VIEWS WHERE NAME = "order_mv";
  ```

- 通过匹配名称来检查特定的异步物化视图。

  ```SQL
  SHOW MATERIALIZED VIEWS WHERE NAME LIKE "order%";
  ```

- 通过在信息模式中查询`materialized_views`元数据视图来检查所有异步物化视图。有关更多信息，请参阅[information_schema.materialized_views](../reference/information_schema/materialized_views.md)。

  ```SQL
  SELECT * FROM information_schema.materialized_views;
  ```

### 检查异步物化视图的定义

您可以通过[SHOW CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_MATERIALIZED_VIEW.md)来检查用于创建异步物化视图的查询。

```SQL
SHOW CREATE MATERIALIZED VIEW order_mv;
```

### 检查异步物化视图的执行状态

您可以通过在[Information Schema](../reference/overview-pages/information_schema.md)中查询`tasks`和`task_runs`来检查异步物化视图的执行（构建或刷新）状态。

下面的示例检查了最近创建的物化视图的执行状态：

1. 在`tasks`表中检查最近任务的`TASK_NAME`。

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

2. 使用您找到的`TASK_NAME`在`task_runs`表中检查执行状态。

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

### 删除一个异步物化视图

您可以通过[DROP MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/DROP_MATERIALIZED_VIEW.md)来删除一个异步物化视图。

```Plain
DROP MATERIALIZED VIEW order_mv;
```

### 相关会话变量

以下变量控制异步物化视图的行为：

- `analyze_mv`：在刷新后是否以及如何分析物化视图。有效值为一个空字符串（不分析）、`sample`（采样统计信息收集）和`full`（完整统计信息收集）。默认为`sample`。
- `enable_materialized_view_rewrite`：是否启用物化视图的自动重写。有效值为`true`（v2.5版以后的默认值）和`false`。