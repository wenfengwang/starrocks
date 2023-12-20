---
displayed_sidebar: English
---

# 您可以使用 CREATE MATERIALIZED VIEW 基于特定的查询语句创建物化视图。

基于表 goods、order_list 和上述查询语句，以下示例创建物化视图 order_mv 来分析每个订单的总计。物化视图设置为每天刷新一次。

注意

对于同步物化视图（Rollup）的场景和用法，请参阅[Synchronous materialized view (Rollup)](../using_starrocks/Materialized_view-single_table.md)。

## 您可以为异步物化视图设置与其基表不同的分区和分桶策略，但是在用于创建物化视图的查询语句中必须包含物化视图的分区键和分桶键。

异步物化视图支持较长时间跨度的动态分区策略。例如，如果基表按一天的间隔进行分区，可以将物化视图设置为按一个月的间隔进行分区。

目前，StarRocks 不支持使用列表分区策略创建异步物化视图，也不支持基于使用列表分区策略创建的表创建物化视图。

用于创建物化视图的查询语句不支持随机函数，包括 rand()、random()、uuid() 和 sleep()。

### 异步物化视图支持多种数据类型。更多信息，请参见 CREATE MATERIALIZED VIEW - 支持的数据类型。

默认情况下，执行 CREATE MATERIALIZED VIEW 语句会立即触发刷新任务，这可能会消耗一定比例的系统资源。如果您想推迟刷新任务，可以在 CREATE MATERIALIZED VIEW 语句中添加 REFRESH DEFERRED 参数。

关于异步物化视图的刷新机制

|单表聚合|多表join|查询重写|刷新策略|基表|
|---|---|---|---|---|
|ASYNC MV|是|是|是|异步刷新手动刷新|多个表来自：默认目录外部目录(v2.5)现有物化视图(v2.5)现有视图(v3.1)|
|SYNC MV (Rollup)|聚合函数选择有限|否|是|数据加载时同步刷新|默认目录单表|

### 您可以为异步物化视图的分区指定生存时间（TTL），从而减少物化视图占用的存储空间。

- **基本表**

  您可以指定数据更改不会自动触发相应物化视图刷新的基表。

  对于StarRocks的异步物化视图，基表可以是[默认目录](../data_source/catalog/default_catalog.md)中的StarRocks原生表，外部目录中的表（从v2.5开始支持），甚至是现有的异步物化视图（从v2.5开始支持）和视图（从v3.1开始支持）。StarRocks支持在所有[StarRocks表类型](../table_design/table_types/table_types.md)上创建异步物化视图。

- **刷新**

  注意

  为了防止完全刷新操作耗尽系统资源并导致任务失败，建议基于分区的基表创建分区物化视图。这样，当基表分区内发生数据更新时，只会刷新相应的物化视图分区，而不是刷新整个物化视图。更多信息，请参见带有物化视图的数据建模-分区建模。

  - 关于嵌套物化视图
  - StarRocks v2.5 支持创建嵌套的异步物化视图。您可以基于现有的异步物化视图构建异步物化视图。每个物化视图的刷新策略不会影响上层或下层的物化视图。目前，StarRocks 不限制嵌套层级的数量。在生产环境中，建议嵌套层数不超过三层。

- **关于外部目录物化视图**

  StarRocks 支持基于 Hive 目录（自 v2.5 起）、Hudi 目录（自 v2.5 起）、Iceberg 目录（自 v2.5 起）和 JDBC 目录（自 v3.0 起）构建异步物化视图。在外部目录上创建物化视图与在默认目录上创建异步物化视图类似，但有一些使用限制。更多信息，请参见使用物化视图加速数据湖查询。

  手动刷新异步物化视图

    > **注意**，您可以使用 `REFRESH MATERIALIZED VIEW` 刷新异步物化视图，无论其刷新策略如何。StarRocks v2.5 支持通过指定分区名称刷新异步物化视图的特定分区。StarRocks v3.1 支持同步调用刷新任务，并仅在任务成功或失败时返回 SQL 语句。
    > 在[JDBC目录](../data_source/catalog/jdbc_catalog.md)上创建的异步物化视图不支持查询重写。

## 直接查询异步物化视图

您创建的异步物化视图本质上是一个包含根据查询语句预先计算结果的完整数据集的物理表。因此，在物化视图首次刷新后，您可以直接查询物化视图。

- **加速使用重复聚合函数的查询**

  您可以直接查询异步物化视图，但结果可能与对其基表进行查询的结果不一致。

- **多表的常规JOIN**

  StarRocks v2.5 支持基于 SPJG 类型的异步物化视图的自动和透明的查询重写。SPJG 类型的物化视图查询重写包括单表查询重写、连接查询重写、聚合查询重写、Union 查询重写和基于嵌套物化视图的查询重写。更多信息，请参见使用物化视图进行查询重写。

- **目前，StarRocks 支持重写在默认目录或外部目录（如 Hive 目录、Hudi 目录或 Iceberg 目录）上创建的异步物化视图的查询。在查询默认目录中的数据时，StarRocks 通过排除与基表数据不一致的物化视图来确保重写查询和原始查询之间的结果强一致性。当物化视图中的数据过期时，物化视图将不会被用作候选物化视图。在查询外部目录中的数据时，StarRocks 无法确保结果的强一致性，因为 StarRocks 无法感知外部目录中的数据更改。有关基于外部目录创建的异步物化视图的更多信息，请参见使用物化视图加速数据湖查询。**

  注意

- **在 JDBC 目录中的基表上创建的异步物化视图不支持查询重写**

  管理异步物化视图

使用 ALTER MATERIALIZED VIEW 可以更改异步物化视图的属性。

- [数据建模](./data_modeling_with_materialized_views.md)启用非活动的物化视图。
- [重命名异步物化视图](./query_rewrite_with_materialized_views.md)。
- [数据湖查询加速](./data_lake_query_acceleration_with_materialized_views.md)，将异步物化视图的刷新间隔更改为2天。

## 显示异步物化视图

您可以使用 SHOW MATERIALIZED VIEWS 或查询 Information Schema 中的系统元数据视图来查看数据库中的异步物化视图。

- 检查数据库中的所有异步物化视图。

- 检查特定的异步物化视图。

  - 通过匹配名称检查特定的异步物化视图。
  - 通过查询信息模式中的元数据视图 materialized_views 来检查所有异步物化视图。有关更多信息，请参见 information_schema.materialized_views。
  - 检查异步物化视图的定义
  - 您可以使用 SHOW CREATE MATERIALIZED VIEW 检查用于创建异步物化视图的查询。

- 检查异步物化视图的执行状态
- 您可以通过查询 Information Schema 中的 tasks 和 task_runs 来检查异步物化视图的执行（构建或刷新）状态。

### 以下示例检查最近创建的物化视图的执行状态：

通过查询 tasks 表中最近任务的 TASK_NAME 来检查执行状态。

- 使用您找到的 TASK_NAME 在 task_runs 表中检查执行状态。
- 删除异步物化视图

您可以使用 DROP MATERIALIZED VIEW 删除异步物化视图。

相关会话变量

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

以下变量控制异步物化视图的行为：

analyze_mv：是否以及如何在刷新后分析物化视图。有效值为空字符串（不分析）、sample（采样统计收集）和full（完整统计收集）。默认值为 sample。

```SQL
SELECT
    order_id,
    sum(goods.price) as total
FROM order_list INNER JOIN goods ON goods.item_id1 = order_list.item_id2
GROUP BY order_id;
```

### enable_materialized_view_rewrite：是否启用物化视图的自动重写。有效值为 true（v2.5 起默认值）和 false。

您可以使用[CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md)基于特定查询语句来创建物化视图。

以下示例基于 goods 表、order_list 表和上述查询语句，创建了名为 order_mv 的物化视图以分析每个订单的总计。该物化视图设定为每隔一天自动刷新一次。

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
- 在创建异步物化视图时，您必须指定物化视图的数据分布策略或刷新策略，或者两者都要指定。
- 您可以为异步物化视图设置与基表不同的分区和分桶策略，但在用于创建物化视图的查询语句中必须包含物化视图的分区键和分桶键。
- 异步物化视图支持更长时间跨度的动态分区策略。例如，如果基表是按照每天分区，那么您可以设置物化视图按照每月分区。
- 目前，StarRocks 不支持使用列表分区策略创建异步物化视图，也不支持基于已使用列表分区策略创建的表来创建异步物化视图。
- 用于创建物化视图的查询语句不支持随机函数，包括 rand()、random()、uuid() 和 sleep()。
- 异步物化视图支持多种数据类型。更多信息，请参见 [CREATE MATERIALIZED VIEW - 支持的数据类型](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md#supported-data-types)。
- 默认情况下，执行 CREATE MATERIALIZED VIEW 语句会立即触发刷新任务，这可能会占用一定比例的系统资源。如果您希望延迟刷新任务，可以在 CREATE MATERIALIZED VIEW 语句中添加 REFRESH DEFERRED 参数。

- **关于异步物化视图的刷新机制**

  目前，StarRocks 支持两种按需刷新策略：MANUAL（手动刷新）和 ASYNC（异步刷新）。

  在 StarRocks v2.5 中，异步物化视图进一步支持多种异步刷新机制，这有助于控制刷新成本并提高刷新成功率：

  - 如果一个物化视图有许多大型分区，每次刷新可能会消耗大量资源。在 v2.5 版本中，StarRocks 支持分拆刷新任务。您可以指定最大刷新分区数，StarRocks 将分批进行刷新，每批的大小不超过您指定的最大分区数。这一功能确保了大型异步物化视图能够稳定刷新，提高了数据建模的稳定性和鲁棒性。
  - 您可以为异步物化视图的分区设置生存时间（TTL），以减少物化视图占用的存储空间。
  - 您可以指定刷新范围，仅刷新最新的几个分区，以减少刷新的负担。
  - 您可以指定基表的数据变更不会自动触发相应物化视图的刷新。
  - 您可以为刷新任务分配资源组。

  更多信息，请参阅 [CREATE MATERIALIZED VIEW - 参数](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md#parameters)中的**PROPERTIES**部分。您也可以使用 [ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md)命令修改现有异步物化视图的机制。

    > **警告**
    > 为防止全量刷新操作耗尽系统资源并导致任务失败，建议基于分区基表创建分区物化视图。这样当基表分区内的数据发生更新时，只有对应的物化视图分区会被刷新，而不是整个物化视图。更多信息，请参阅 [使用物化视图进行数据建模 - 分区建模](./data_modeling_with_materialized_views.md#partitioned-modeling)。

- **关于嵌套物化视图**

  从 StarRocks v2.5 版本开始，支持创建嵌套的异步物化视图。您可以基于现有的异步物化视图构建新的异步物化视图。每个物化视图的刷新策略不会影响上层或下层物化视图。目前，StarRocks 不限制嵌套层数。在生产环境中，我们建议嵌套层数不要超过三层。

- **关于外部目录物化视图**

  StarRocks 支持基于 Hive Catalog（自 v2.5 起）、Hudi Catalog（自 v2.5 起）、Iceberg Catalog（自 v2.5 起）和 JDBC Catalog（自 v3.0 起）创建异步物化视图。在外部目录上创建物化视图的过程与在默认目录上创建异步物化视图类似，但存在一些使用限制。更多信息，请参阅 [使用物化视图加速数据湖查询](./data_lake_query_acceleration_with_materialized_views.md)。

## 手动刷新异步物化视图

无论其刷新策略如何，您都可以通过 [REFRESH MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/REFRESH_MATERIALIZED_VIEW.md) 命令手动刷新异步物化视图。从 StarRocks v2.5 版本开始，支持通过指定分区名称来刷新异步物化视图的特定分区。从 StarRocks v3.1 版本开始，支持同步调用刷新任务，并且 SQL 语句只有在任务成功或失败后才返回。

```SQL
-- Refresh the materialized view via an asynchronous call (default).
REFRESH MATERIALIZED VIEW order_mv;
-- Refresh the materialized view via a synchronous call.
REFRESH MATERIALIZED VIEW order_mv WITH SYNC MODE;
```

您可以使用[CANCEL REFRESH MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/CANCEL_REFRESH_MATERIALIZED_VIEW.md)命令取消通过异步调用提交的刷新任务。

## 直接查询异步物化视图

您创建的异步物化视图本质上是一个包含了根据查询语句预计算结果的物理表。因此，物化视图首次刷新后，您就可以直接对其进行查询。

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
> 您可以直接查询异步物化视图，但其结果可能与直接查询基表得到的结果不一致。

## 使用异步物化视图重写和加速查询

从 StarRocks v2.5 版本开始，支持基于 SPJG 类型的异步物化视图进行自动和透明的查询重写。SPJG 类型的物化视图查询重写包括单表查询重写、联接查询重写、聚合查询重写、并集查询重写以及查询重写基于嵌套物化视图。更多信息，请参阅 [Query Rewrite with Materialized Views](./query_rewrite_with_materialized_views.md)。

目前，StarRocks支持重写在默认目录或外部目录（例如Hive目录、Hudi目录或Iceberg目录）上创建的异步物化视图的查询。在查询数据湖中的数据时，StarRocks通过排除与基表数据不一致的物化视图，确保重写查询与原始查询之间的结果强一致性。当物化视图中的数据过期时，该物化视图不会被作为候选物化视图使用。在查询外部目录中的数据时，由于StarRocks无法感知外部目录中的数据变化，因此无法保证结果的强一致性。更多关于异步物化视图的信息，请参阅[使用物化视图加速数据湖查询](./data_lake_query_acceleration_with_materialized_views.md)。

> **注意**
> 在 JDBC 目录中创建的**基表**上的**异步物化视图**不支持**查询重写**。

## 管理异步物化视图

### 更改异步物化视图

您可以使用[ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md)命令更改异步物化视图的属性：

- 启用处于非活动状态的物化视图。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv ACTIVE;
  ```

- 重命名异步物化视图。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv RENAME order_total;
  ```

- 将异步物化视图的刷新间隔更改为两天。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv REFRESH ASYNC EVERY(INTERVAL 2 DAY);
  ```

### 显示异步物化视图

您可以使用[SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)命令或查询信息模式（Information Schema）中的系统元数据视图来查看数据库中的异步物化视图：

- 查看数据库中的所有异步物化视图。

  ```SQL
  SHOW MATERIALIZED VIEWS;
  ```

- 查看特定的异步物化视图。

  ```SQL
  SHOW MATERIALIZED VIEWS WHERE NAME = "order_mv";
  ```

- 通过匹配名称来查看特定的异步物化视图。

  ```SQL
  SHOW MATERIALIZED VIEWS WHERE NAME LIKE "order%";
  ```

- 通过查询 Information Schema 中的元数据视图 `materialized_views` 来查看所有异步物化视图。更多信息，请参阅 [information_schema.materialized_views](../reference/information_schema/materialized_views.md)。

  ```SQL
  SELECT * FROM information_schema.materialized_views;
  ```

### 检查异步物化视图的定义

您可以通过[SHOW CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_MATERIALIZED_VIEW.md)命令检查用于创建异步物化视图的查询。

```SQL
SHOW CREATE MATERIALIZED VIEW order_mv;
```

### 检查异步物化视图的执行状态

您可以通过查询 [tasks](../reference/information_schema/tasks.md) 和 [task_runs](../reference/information_schema/task_runs.md) 来查看异步物化视图的执行（构建或刷新）状态：在 [Information Schema](../reference/overview-pages/information_schema.md)。

以下示例检查了最近创建的物化视图的执行状态：

1. 查看 tasks 表中最新任务的 TASK_NAME。

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

2. 使用您找到的 TASK_NAME 在 task_runs 表中查看执行状态。

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

### 删除异步物化视图

您可以通过[DROP MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/DROP_MATERIALIZED_VIEW.md)命令删除异步物化视图。

```Plain
DROP MATERIALIZED VIEW order_mv;
```

### 相关会话变量

以下变量控制异步物化视图的行为：

- analyze_mv：刷新后是否以及如何分析物化视图。有效值为一个空字符串（不进行分析）、sample（采样统计信息收集）和 full（全面统计信息收集）。默认值为 sample。
- 启用物化视图自动改写：是否启用物化视图的自动改写功能。有效值为 true（自 v2.5 起为默认值）和 false。
