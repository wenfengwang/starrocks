---
displayed_sidebar: "Chinese"
---

# 异步物化视图

This article describes how to understand, create, use, and manage asynchronous materialized views in StarRocks. Starting from version 2.4, StarRocks supports asynchronous materialized views.

Compared to synchronous materialized views, asynchronous materialized views support multi-table associations and richer aggregation operators. Asynchronous materialized views can be refreshed through manual invocation or scheduled tasks, and support partial partition refresh, which greatly reduces the refresh cost. In addition, asynchronous materialized views support multiple query rewrite scenarios, achieving automatic and transparent query acceleration.

For scenarios and usage of synchronous materialized views (Rollup), see [Synchronous Materialized Views (Rollup)](../using_starrocks/Materialized_view-single_table.md).

## Background

Applications in data warehouse environments often perform complex queries based on multiple large tables, typically involving the association and aggregation of tens of billions of rows of data across multiple tables. Handling such queries typically consumes a large amount of system resources and time, resulting in extremely high query costs.

You can address the above issues with asynchronous materialized views in StarRocks. An asynchronous materialized view is a special physical table that stores precomputed results based on specific query statements of the base table. When you perform complex queries on the base table, StarRocks can directly reuse the precomputed results, avoiding redundant computation and thereby improving query performance. The higher the query frequency or the more complex the query statement, the more significant the performance gain will be.

Through asynchronous materialized views, you can also model the data warehouse to provide a unified data caliber to upper-level applications, shielding the underlying implementation and protecting the security of detailed data in the base table.

### Understanding StarRocks Materialized Views

Versions prior to StarRocks v2.4 provided a synchronous updated materialized view (Rollup) to provide better data freshness and lower refresh costs. However, synchronous materialized views have many limitations in scenarios. They can only be created based on a single base table and only support limited aggregation operators. Starting from version v2.4, support for asynchronous materialized views was added, which can be created based on multiple base tables and support richer aggregation operators.

The following table compares the supported features of asynchronous materialized views and synchronous materialized views (Rollup) in StarRocks from a feature perspective:

|                              | **Single Table Aggregation** | **Multi-Table Association** | **Query Rewrite** | **Refresh Strategy** | **Base Table** |
| ---------------------------- | ----------- | ---------- | ----------- | ---------- | -------- |
| **Asynchronous Materialized Views** | Yes | Yes | Yes | <ul><li>Asynchronous Refresh</li><li>manual Refresh</li></ul> | Supports constructing multiple tables. Base tables can come from: <ul><li>Default Catalog</li><li>External Catalog (v2.5)</li><li>Existing asynchronous materialized views (v2.5)</li><li>Existing views (v3.1)</li></ul> |
| **Synchronous Materialized Views (Rollup)** | Only partial aggregation functions | No | Yes | Import synchronous refresh | Only supports single table construction based on Default Catalog |

### Related Concepts

- **Base Table**

  The driving table for materialized views.

  For asynchronous materialized views in StarRocks, the base table can be an internal table in the [Default catalog](../data_source/catalog/default_catalog.md), a table in an external data directory (supported since version 2.5), or even an existing asynchronous materialized view (supported since v2.5) or view (supported since v3.1). StarRocks supports creating asynchronous materialized views on all [StarRocks table types](../table_design/table_types/table_types.md).

- **Refresh**

  After creating an asynchronous materialized view, the data in it only reflects the state of the base table at the time of creation. When the data in the base table changes, the asynchronous materialized view needs to be refreshed to update the data.

  Currently, StarRocks supports two asynchronous refresh strategies:

  - ASYNC: Asynchronous refresh. Whenever the data in the base table changes, the materialized view automatically triggers a refresh task based on the specified refresh interval.
  - MANUAL: Manually trigger refresh. The materialized view does not automatically refresh, and users need to manually maintain the refresh task.

- **Query Rewrite**

  Query rewrite refers to the system automatically determining whether the precomputed results in the materialized view can be reused to process the query on the base table with the materialized view constructed. If reuse is possible, the system directly reads the precomputed results from the relevant materialized view to avoid redundant computation, consuming system resources and time.

  Since version v2.5, StarRocks supports automatic and transparent query rewrite for asynchronous materialized views based on the SPJG type. SPJG type materialized views refer to materialized view plans that only contain Scan, Filter, Project, and Aggregate type operators.

  > **Note**
  >
  > Asynchronous materialized views built on tables based on the [JDBC Catalog](../data_source/catalog/jdbc_catalog.md) temporarily do not support query rewrite.

## Usage Scenarios

If your data warehouse environment has the following requirements, we recommend creating asynchronous materialized views:

- **Accelerating Duplicate Aggregated Queries**

  Suppose there are a large number of queries in your data warehouse environment that contain the same subqueries with aggregate functions, consuming a large amount of computing resources. You can create an asynchronous materialized view based on this subquery, calculate and save all the results of this subquery. After successful creation, the system will automatically rewrite the query statement to directly query the intermediate results in the asynchronous materialized view, reducing the load and accelerating the query.

- **Periodic Multi-Table Association Queries**

  Suppose you need to periodically associate multiple tables in the data warehouse to generate a new wide table. You can create asynchronous materialized views for these tables and set up periodic refresh rules to avoid manual scheduling of association tasks. After the asynchronous materialized view is successfully created, queries will directly return results based on the asynchronous materialized view, thereby avoiding delays caused by association operations.

- **Data Warehouse Hierarchies**

  If your base table contains a large amount of raw data and queries require complex ETL operations, you can implement data warehouse hierarchies through multiple layers of asynchronous materialized views. This can decompose complex queries into multiple simple queries, reducing redundant computations and helping maintenance personnel quickly locate problems. Additionally, data warehouse hierarchies can decouple raw data from statistical data, protecting sensitive original data.

- **Data Lake Acceleration**

  Querying data lakes may become slow due to network latency and throughput limitations of object storage. You can improve query performance by building asynchronous materialized views on top of the data lake. In addition, StarRocks can intelligently rewrite queries to use existing materialized views, eliminating the hassle of manual query modifications.

```markdown
      + {R}
      + {R}
    + {R}
  + {R}
```
```markdown
      + {T}
      + {T}
    + {T}
  + {T}
```
StarRocks支持使用Hive Catalog（从v2.5开始）、Hudi Catalog（从v2.5开始）、Iceberg Catalog（从v2.5开始）和JDBC Catalog（从v3.0开始）构建异步物化视图。外部数据目录中的物化视图的创建方式与普通异步物化视图相同，但有使用限制。有关详细信息，请参阅[使用物化视图加速数据湖查询](./data_lake_query_acceleration_with_materialized_views.md)。

## 手动刷新异步物化视图

您可以使用[REFRESH MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/REFRESH_MATERIALIZED_VIEW.md)命令手动刷新指定的异步物化视图。在StarRocks v2.5版本中，异步物化视图支持手动刷新部分分区。在v3.1版本中，StarRocks支持同步调用刷新任务。

```SQL
-- 异步调用刷新任务。
REFRESH MATERIALIZED VIEW order_mv;
-- 同步调用刷新任务。
REFRESH MATERIALIZED VIEW order_mv WITH SYNC MODE;
```

您可以使用[CANCEL REFRESH MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/CANCEL_REFRESH_MATERIALIZED_VIEW.md)取消异步调用的刷新任务。

## 直接查询异步物化视图

异步物化视图本质上是一个物理表，其中存储了根据特定查询语句预先计算的完整结果集。在物化视图第一次刷新后，您即可直接查询物化视图。

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

> **说明**
>
> 您可以直接查询异步物化视图，但由于异步刷新机制，其结果可能与您从基表上查询的结果不一致。

## 使用异步物化视图改写加速查询

StarRocks v2.5版本支持SPJG类型的异步物化视图查询的自动透明改写。其查询改写包括单表改写，Join改写，聚合改写，Union改写和嵌套物化视图的改写。有关详细内容，请参考[物化视图查询改写](./query_rewrite_with_materialized_views.md)。

目前，StarRocks支持基于Default catalog、Hive catalog、Hudi catalog和Iceberg catalog的异步物化视图的查询改写。当查询Default catalog数据时，StarRocks通过排除数据与基表不一致的物化视图，来保证改写之后的查询与原始查询结果的强一致性。当物化视图数据过期时，不会作为候选物化视图。在查询外部目录数据时，由于StarRocks无法感知外部目录分区中的数据变化，因此不保证结果的强一致性。有关基于外部目录的异步物化视图，请参考[使用物化视图加速数据湖查询](./data_lake_query_acceleration_with_materialized_views.md)。

> **注意**
>
> 基于JDBC Catalog表构建的异步物化视图暂不支持查询改写。

## 管理异步物化视图

### 修改异步物化视图

您可以使用[ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md)命令修改异步物化视图属性。

- 启用被禁用的异步物化视图（将物化视图的状态设置为Active）。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv ACTIVE;
  ```

- 修改异步物化视图名称为 `order_total`。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv RENAME order_total;
  ```

- 修改异步物化视图的最大刷新间隔为2天。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv REFRESH ASYNC EVERY(INTERVAL 2 DAY);
  ```

### 查看异步物化视图

您可以使用[SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)或查询信息模式中的系统元数据视图来查看数据库中的异步物化视图。

- 查看当前数据仓库内所有异步物化视图。

  ```SQL
  SHOW MATERIALIZED VIEWS;
  ```

- 查看特定异步物化视图。

  ```SQL
  SHOW MATERIALIZED VIEWS WHERE NAME = "order_mv";
  ```

- 通过名称匹配查看异步物化视图。

  ```SQL
  SHOW MATERIALIZED VIEWS WHERE NAME LIKE "order%";
  ```

- 通过信息模式中的系统元数据视图`materialized_views`查看所有异步物化视图。有关详细内容，请参考[information_schema.materialized_views](../reference/information_schema/materialized_views.md)。

  ```SQL
  SELECT * FROM information_schema.materialized_views;
  ```

### 查看异步物化视图创建语句

您可以使用[SHOW CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_MATERIALIZED_VIEW.md)命令查看异步物化视图创建语句。

```SQL
SHOW CREATE MATERIALIZED VIEW order_mv;
```

### 查看异步物化视图的执行状态

您可以通过查询StarRocks的[信息模式](../reference/overview-pages/information_schema.md)中的[`tasks`](../reference/information_schema/tasks.md)和[`task_runs`](../reference/information_schema/task_runs.md)元数据视图来查看异步物化视图的执行（构建或刷新）状态。

以下示例查看最新创建的异步物化视图的执行状态：

1. 查看`tasks`表中最新任务的`TASK_NAME`。

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

您可以通过 [DROP MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/DROP_MATERIALIZED_VIEW.md) 命令删除已创建的异步物化视图。

```SQL
DROP MATERIALIZED VIEW order_mv;
```

### 相关 Session 变量

以下变量控制物化视图的行为：

- `analyze_mv`：刷新后是否以及如何分析物化视图。有效值为空字符串（即不分析）、`sample`（抽样采集）或 `full`（全量采集）。默认为 `sample`。
- `enable_materialized_view_rewrite`：是否开启物化视图的自动改写。有效值为 `true`（自 2.5 版本起为默认值）和 `false`。