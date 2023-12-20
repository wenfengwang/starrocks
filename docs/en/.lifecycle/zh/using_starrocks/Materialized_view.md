---
displayed_sidebar: English
---

# 异步物化视图

本主题介绍如何理解、创建、使用和管理异步物化视图。StarRocks 从 v2.4 版本开始支持异步物化视图。

与同步物化视图相比，异步物化视图支持多表连接和更多聚合函数。异步物化视图的刷新可以手动触发，也可以通过计划任务触发。您还可以选择刷新部分分区而非整个物化视图，这大大降低了刷新成本。此外，异步物化视图支持多种查询重写场景，允许自动、透明地加速查询。

有关同步物化视图（Rollup）的场景和用法，请参见[Synchronous materialized view (Rollup)](../using_starrocks/Materialized_view-single_table.md)。

## 概述

数据库应用程序经常需要对大型表执行复杂查询。这些查询涉及对包含数十亿行的表进行多表连接和聚合。处理这些查询可能会消耗大量系统资源，并且计算结果所需的时间也可能很长。

StarRocks 中的异步物化视图旨在解决这些问题。异步物化视图是一种特殊的物理表，它保存了来自一个或多个基表的预计算查询结果。当您对基表执行复杂查询时，StarRocks 会从相关物化视图中返回预计算的结果来处理这些查询。这样，查询性能得以提升，因为避免了重复的复杂计算。当查询频繁运行或足够复杂时，性能提升可能非常显著。

此外，异步物化视图在构建数据仓库上的数学模型时特别有用。通过这种方式，您可以为上层应用提供统一的数据规范，屏蔽底层实现，或保护基表的原始数据安全。

### 了解 StarRocks 中的物化视图

StarRocks v2.3 及更早版本提供了只能在单个表上构建的同步物化视图。同步物化视图，也就是 Rollup，保持了更高的数据新鲜度和较低的刷新成本。然而，与 v2.4 及之后版本支持的异步物化视图相比，同步物化视图有许多限制。当您想要构建同步物化视图来加速或重写查询时，可选择的聚合运算符有限。

下表从支持的功能角度比较了 StarRocks 中的异步物化视图（ASYNC MV）和同步物化视图（SYNC MV）：

|**单表聚合**|**多表连接**|**查询重写**|**刷新策略**|**基表**|
|---|---|---|---|---|
|**ASYNC MV**|是|是|是|<ul><li>异步刷新</li><li>手动刷新</li></ul>|多个表，来自：<ul><li>默认目录</li><li>外部目录（v2.5）</li><li>现有物化视图（v2.5）</li><li>现有视图（v3.1）</li></ul>|
|**SYNC MV (Rollup)**|有限的聚合函数选择|否|是|数据加载时同步刷新|默认目录中的单个表|

### 基本概念

- **基表**

  基表是物化视图的驱动表。

  对于 StarRocks 的异步物化视图，基表可以是[默认目录](../data_source/catalog/default_catalog.md)中的 StarRocks 本机表、外部目录中的表（从 v2.5 版本开始支持），或者是现有的异步物化视图（从 v2.5 版本开始支持）和视图（从 v3.1 版本开始支持）。StarRocks 支持在所有[StarRocks 表的类型](../table_design/table_types/table_types.md)上创建异步物化视图。

- **刷新**

  当您创建异步物化视图时，其数据仅反映了基表当时的状态。当基表中的数据发生变化时，您需要刷新物化视图以保持数据同步。

  目前，StarRocks 支持两种通用刷新策略：

  - ASYNC：异步刷新模式。每次基表数据发生变化时，物化视图会根据预定义的刷新间隔自动刷新。
  - MANUAL：手动刷新模式。物化视图不会自动刷新。刷新任务只能由用户手动触发。

- **查询重写**

  查询重写指的是在对建立了物化视图的基表执行查询时，系统自动判断物化视图中的预计算结果是否可以用于该查询。如果可以重用，系统将直接从相关物化视图加载数据，避免耗时和资源消耗的计算或连接。

  从 v2.5 版本开始，StarRocks 支持基于 SPJG 类型异步物化视图的自动、透明查询重写。SPJG 类型物化视图指的是计划中只包含扫描（Scan）、过滤（Filter）、投影（Project）和聚合（Aggregate）类型算子的物化视图。

    > **注意**
    > 在 [JDBC 目录](../data_source/catalog/jdbc_catalog.md) 中创建的异步物化视图不支持查询重写。

## 决定何时创建物化视图

如果您的数据仓库环境有以下需求，您可以创建异步物化视图：

- **使用重复聚合函数加速查询**

  假设您的数据仓库中的大多数查询都包含具有聚合函数的相同子查询，并且这些查询消耗了您大量的计算资源。基于此子查询，您可以创建一个异步物化视图，它将计算并存储子查询的所有结果。物化视图构建完成后，StarRocks 会重写所有包含子查询的查询，加载物化视图中存储的中间结果，从而加速这些查询。

- **多表的常规 JOIN**

  假设您需要定期在数据仓库中连接多个表以生成一个新的宽表。您可以为这些表构建异步物化视图，并设置 ASYNC 刷新策略，以固定时间间隔触发刷新任务。物化视图构建完成后，查询结果将直接从物化视图返回，从而避免了 JOIN 操作带来的延迟。

- **数据仓库分层**

  假设您的数据仓库包含大量原始数据，并且其中的查询需要一组复杂的 ETL 操作。您可以构建多层异步物化视图来对数据仓库中的数据进行分层，从而将查询分解为一系列简单的子查询。这可以显著减少重复计算，并且更重要的是，帮助您的 DBA 轻松高效地识别问题。此外，数据仓库分层有助于解耦原始数据和统计数据，保护敏感原始数据的安全。

- **加速数据湖中的查询**

  由于网络延迟和对象存储吞吐量，查询数据湖可能会很慢。您可以通过在数据湖之上构建异步物化视图来提升查询性能。此外，StarRocks 可以智能地重写查询以使用现有的物化视图，省去了您手动修改查询的麻烦。

有关异步物化视图的具体使用案例，请参考以下内容：

- [数据建模](./data_modeling_with_materialized_views.md)
- [查询重写](./query_rewrite_with_materialized_views.md)
- [数据湖查询加速](./data_lake_query_acceleration_with_materialized_views.md)

## 创建异步物化视图

StarRocks 的异步物化视图可以在以下基表上创建：

- StarRocks 的原生表（支持所有 StarRocks 表类型）

- 外部目录中的表，包括：

  - Hive 目录（自 v2.5 起）
  - Hudi 目录（自 v2.5 起）
  - Iceberg 目录（自 v2.5 起）
  - JDBC 目录（自 v3.0 起）

- 现有的异步物化视图（自 v2.5 起）
```
- 现有视图（自 v3.1 起）

### 在您开始之前

以下示例涉及默认目录中的两个基表：

- 表 `goods` 记录了商品 ID `item_id1`、商品名称 `item_name` 和商品价格 `price`。
- 表 `order_list` 记录了订单 ID `order_id`、客户 ID `client_id`、商品 ID `item_id2` 和订单日期 `order_date`。

`goods.item_id1` 列与 `order_list.item_id2` 列等效。

执行以下语句以创建表并插入数据：

```SQL
CREATE TABLE goods(
    item_id1          INT,
    item_name         STRING,
    price             FLOAT
) DISTRIBUTED BY HASH(item_id1);

INSERT INTO goods
VALUES
    (1001, "apple", 6.5),
    (1002, "pear", 8.0),
    (1003, "potato", 2.2);

CREATE TABLE order_list(
    order_id          INT,
    client_id         INT,
    item_id2          INT,
    order_date        DATE
) DISTRIBUTED BY HASH(order_id);

INSERT INTO order_list
VALUES
    (10001, 101, 1001, "2022-03-13"),
    (10001, 101, 1002, "2022-03-13"),
    (10002, 103, 1002, "2022-03-13"),
    (10002, 103, 1003, "2022-03-14"),
    (10003, 102, 1003, "2022-03-14"),
    (10003, 102, 1001, "2022-03-14");
```

以下示例中的场景需要频繁计算每个订单的总金额。它需要频繁连接两个基表并大量使用聚合函数 `sum()`。此外，业务场景要求数据每天刷新一次。

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

以下示例基于表 `goods`、`order_list` 和上述查询语句，创建了物化视图 `order_mv` 来分析每个订单的总金额。物化视图设置为每隔一天自动刷新一次。

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
- 创建异步物化视图时，必须指定物化视图的数据分布策略或刷新策略，或两者都指定。
- 您可以为异步物化视图设置与其基表不同的分区和分桶策略，但在创建物化视图的查询语句中必须包含物化视图的分区键和分桶键。
- 异步物化视图支持更长时间跨度的动态分区策略。例如，如果基表的分区间隔为一天，您可以设置物化视图的分区间隔为一个月。
- 目前，StarRocks 不支持使用列表分区策略或基于使用列表分区策略创建的表来创建异步物化视图。
- 用于创建物化视图的查询语句不支持随机函数，包括 `rand()`、`random()`、`uuid()` 和 `sleep()`。
- 异步物化视图支持多种数据类型。有关详细信息，请参阅 [CREATE MATERIALIZED VIEW - 支持的数据类型](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md#supported-data-types)。
- 默认情况下，执行 CREATE MATERIALIZED VIEW 语句会立即触发刷新任务，这可能会消耗一定比例的系统资源。如果您希望推迟刷新任务，可以在 CREATE MATERIALIZED VIEW 语句中添加 REFRESH DEFERRED 参数。

- **关于异步物化视图的刷新机制**

  目前，StarRocks 支持两种按需刷新策略：手动刷新和异步刷新。

  在 StarRocks v2.5 中，异步物化视图进一步支持多种异步刷新机制，以控制刷新成本并提高成功率：

  - 如果 MV 有许多大分区，每次刷新都可能消耗大量资源。在 v2.5 中，StarRocks 支持拆分刷新任务。您可以指定最大刷新分区数，StarRocks 将批量进行刷新，批次大小小于或等于指定的最大分区数。这一特性确保了大型异步物化视图的稳定刷新，增强了数据建模的稳定性和鲁棒性。
  - 您可以为异步物化视图的分区指定生存时间（TTL），以减少物化视图占用的存储空间。
  - 您可以指定刷新范围，仅刷新最新的几个分区，以减少刷新开销。
  - 您可以指定基表，其中数据变更不会自动触发相应物化视图的刷新。
  - 您可以为刷新任务分配资源组。

  有关详细信息，请参阅 [CREATE MATERIALIZED VIEW - 参数](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md#parameters) 中的 **PROPERTIES** 部分。您还可以使用 [ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md) 修改现有异步物化视图的机制。

    > **警告**
    > 为防止全量刷新操作耗尽系统资源并导致任务失败，建议基于分区基表创建分区物化视图。这样，当基表分区内发生数据更新时，只需刷新物化视图的相应分区，而不是整个物化视图。有关详细信息，请参考 [使用物化视图进行数据建模 - 分区建模](./data_modeling_with_materialized_views.md#partitioned-modeling)。

- **关于嵌套物化视图**

  StarRocks v2.5 支持创建嵌套异步物化视图。您可以基于现有的异步物化视图构建新的异步物化视图。每个物化视图的刷新策略不会影响上层或下层物化视图。目前，StarRocks 不限制嵌套层数。在生产环境中，我们建议嵌套层数不超过三层。

- **关于外部目录物化视图**

  StarRocks 支持基于 Hive Catalog（自 v2.5 起）、Hudi Catalog（自 v2.5 起）、Iceberg Catalog（自 v2.5 起）和 JDBC Catalog（自 v3.0 起）构建异步物化视图。在外部目录上创建物化视图与在默认目录上创建异步物化视图类似，但存在一些使用限制。有关更多信息，请参阅 [使用物化视图加速数据湖查询](./data_lake_query_acceleration_with_materialized_views.md)。

## 手动刷新异步物化视图

您可以通过 [REFRESH MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/REFRESH_MATERIALIZED_VIEW.md) 刷新异步物化视图，无论其刷新策略如何。StarRocks v2.5 支持通过指定分区名称刷新异步物化视图的特定分区。StarRocks v3.1 支持同步调用刷新任务，SQL 语句只在任务成功或失败后返回。

```SQL
-- 通过异步调用刷新物化视图（默认方式）。
REFRESH MATERIALIZED VIEW order_mv;
-- 通过同步调用刷新物化视图。
REFRESH MATERIALIZED VIEW order_mv WITH SYNC MODE;
```

您可以使用 [CANCEL REFRESH MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/CANCEL_REFRESH_MATERIALIZED_VIEW.md) 取消通过异步调用提交的刷新任务。

## 直接查询异步物化视图

您创建的异步物化视图本质上是一个物理表，包含了根据查询语句预先计算的完整结果集。因此，在物化视图首次刷新后，您就可以直接对其进行查询。

```Plain
MySQL > SELECT * FROM order_mv;
```
```Plain
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
> 您可以直接查询异步物化视图，但结果可能与基于其基表查询得到的结果不一致。

## 使用异步物化视图重写和加速查询

StarRocks v2.5 支持基于 SPJG 类型异步物化视图的自动和透明查询重写。SPJG 类型物化视图的查询重写包括单表查询重写、联接查询重写、聚合查询重写、联合查询重写以及基于嵌套物化视图的查询重写。更多信息，请参阅[使用物化视图重写查询](./query_rewrite_with_materialized_views.md)。

目前，StarRocks 支持重写在默认目录或外部目录（如 Hive 目录、Hudi 目录或 Iceberg 目录）上创建的异步物化视图的查询。在查询默认目录中的数据时，StarRocks 通过排除与基表数据不一致的物化视图来确保重写查询与原始查询结果的强一致性。当物化视图中的数据过期时，该物化视图将不会被用作候选物化视图。在查询外部目录中的数据时，StarRocks 不保证结果的强一致性，因为 StarRocks 无法感知外部目录中的数据变化。有关基于外部目录创建的异步物化视图的更多信息，请参阅[使用物化视图加速数据湖查询](./data_lake_query_acceleration_with_materialized_views.md)。

> **注意**
> 在 JDBC 目录中创建的基表上的异步物化视图不支持查询重写。

## 管理异步物化视图

### 修改异步物化视图

您可以使用 [ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md) 修改异步物化视图的属性。

- 启用非活跃物化视图。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv ACTIVE;
  ```

- 重命名异步物化视图。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv RENAME TO order_total;
  ```

- 将异步物化视图的刷新间隔修改为 2 天。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv REFRESH ASYNC EVERY(INTERVAL 2 DAY);
  ```

### 显示异步物化视图

您可以使用 [SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md) 或查询信息模式中的系统元数据视图来查看数据库中的异步物化视图。

- 查看数据库中的所有异步物化视图。

  ```SQL
  SHOW MATERIALIZED VIEWS;
  ```

- 查看特定的异步物化视图。

  ```SQL
  SHOW MATERIALIZED VIEWS WHERE NAME = 'order_mv';
  ```

- 通过名称匹配来查看特定的异步物化视图。

  ```SQL
  SHOW MATERIALIZED VIEWS WHERE NAME LIKE 'order%';
  ```

- 通过查询信息模式中的元数据视图 `materialized_views` 来查看所有异步物化视图。更多信息，请参阅 [information_schema.materialized_views](../reference/information_schema/materialized_views.md)。

  ```SQL
  SELECT * FROM information_schema.materialized_views;
  ```

### 查看异步物化视图的定义

您可以通过 [SHOW CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_MATERIALIZED_VIEW.md) 查看用于创建异步物化视图的查询。

```SQL
SHOW CREATE MATERIALIZED VIEW order_mv;
```

### 查看异步物化视图的执行状态

您可以通过查询 [Information Schema](../reference/overview-pages/information_schema.md) 中的 [`tasks`](../reference/information_schema/tasks.md) 和 [`task_runs`](../reference/information_schema/task_runs.md) 来查看异步物化视图的执行（构建或刷新）状态。

以下示例检查最近创建的物化视图的执行状态：

1. 查看 `tasks` 表中最新任务的 `TASK_NAME`。

   ```Plain
   mysql> SELECT * FROM information_schema.tasks ORDER BY CREATE_TIME DESC LIMIT 1\G;
   *************************** 1. row ***************************
     TASK_NAME: mv-59299
   CREATE_TIME: 2022-12-12 17:33:51
     SCHEDULE: MANUAL
     DATABASE: ssb_1
   DEFINITION: INSERT OVERWRITE hive_mv_lineorder_flat_1 SELECT `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderkey`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_linenumber`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_custkey`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_partkey`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderpriority`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_ordtotalprice`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_revenue`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`p_mfgr`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`s_nation`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`c_city`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`c_nation`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderdate`
   FROM `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`
   WHERE `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderdate` = '1997-01-01'
   EXPIRE_TIME: NULL
   1 row in set (0.02 sec)
   ```

2. 使用找到的 `TASK_NAME` 查看 `task_runs` 表中的执行状态。

   ```Plain
   mysql> SELECT * FROM information_schema.task_runs WHERE task_name='mv-59299' ORDER BY CREATE_TIME \G;
   *************************** 1. row ***************************
       QUERY_ID: d9cef11f-7a00-11ed-bd90-00163e14767f
       TASK_NAME: mv-59299
     CREATE_TIME: 2022-12-12 17:39:19
     FINISH_TIME: 2022-12-12 17:39:22
           STATE: SUCCESS
       DATABASE: ssb_1
     DEFINITION: INSERT OVERWRITE hive_mv_lineorder_flat_1 SELECT `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderkey`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_linenumber`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_custkey`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_partkey`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderpriority`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_ordtotalprice`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_revenue`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`p_mfgr`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`s_nation`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`c_city`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`c_nation`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderdate`
   FROM `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`
   WHERE `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderdate` = '1997-01-01'
     EXPIRE_TIME: 2022-12-15 17:39:19
     ERROR_CODE: 0
   ERROR_MESSAGE: NULL
       PROGRESS: 100%
   2 rows in set (0.02 sec)
   ```

### 删除异步物化视图

您可以通过 [DROP MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/DROP_MATERIALIZED_VIEW.md) 删除异步物化视图。

```Plain
DROP MATERIALIZED VIEW order_mv;
```
```markdown
### 相关会话变量

以下变量控制异步物化视图的行为：

- `analyze_mv`：刷新后是否以及如何分析物化视图。有效值为空字符串（不分析）、`sample`（采样统计信息收集）和 `full`（完整统计信息收集）。默认值为 `sample`。
- `enable_materialized_view_rewrite`：是否启用物化视图的自动重写。有效值为 `true`（自 v2.5 起默认）和 `false`。