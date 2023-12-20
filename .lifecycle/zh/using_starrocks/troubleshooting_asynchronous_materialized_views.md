---
displayed_sidebar: English
---

# 异步物化视图的故障排查

本主题介绍如何检查您的异步物化视图并解决在使用过程中遇到的问题。

> **注意**
> 以下展示的某些功能仅从 StarRocks v3.1 版本开始支持。

## 检验异步物化视图

为了全面了解您正在使用的异步物化视图，您可以首先检查它们的工作状态、刷新历史和资源消耗情况。

### 检查异步物化视图的工作状态

您可以通过 [SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md) 命令来检查异步物化视图的工作状态。在所有返回的信息中，您应当重点关注以下几个字段：

- is_active：物化视图是否处于活跃状态。只有处于活跃状态的物化视图才能用于查询加速和改写。
- last_refresh_state：最后一次刷新的状态，可能的值包括 PENDING、RUNNING、FAILED 和 SUCCESS。
- last_refresh_error_message：最后一次刷新失败的原因（如果物化视图状态不是活跃的）。
- rows：物化视图中的数据行数。请注意，这个值可能与物化视图的实际行数不同，因为更新可能会被延迟。

关于返回的其他字段的详细信息，请参见[SHOW MATERIALIZED VIEWS - 返回结果](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md#returns)。

示例：

```Plain
MySQL > SHOW MATERIALIZED VIEWS LIKE 'mv_pred_2'\G
***************************[ 1. row ]***************************
id                                   | 112517
database_name                        | ssb_1g
name                                 | mv_pred_2
refresh_type                         | ASYNC
is_active                            | true
inactive_reason                      | <null>
partition_type                       | UNPARTITIONED
task_id                              | 457930
task_name                            | mv-112517
last_refresh_start_time              | 2023-08-04 16:46:50
last_refresh_finished_time           | 2023-08-04 16:46:54
last_refresh_duration                | 3.996
last_refresh_state                   | SUCCESS
last_refresh_force_refresh           | false
last_refresh_start_partition         |
last_refresh_end_partition           |
last_refresh_base_refresh_partitions | {}
last_refresh_mv_refresh_partitions   |
last_refresh_error_code              | 0
last_refresh_error_message           |
rows                                 | 0
text                                 | CREATE MATERIALIZED VIEW `mv_pred_2` (`lo_quantity`, `lo_revenue`, `sum`)
DISTRIBUTED BY HASH(`lo_quantity`, `lo_revenue`) BUCKETS 2
REFRESH ASYNC
PROPERTIES (
"replication_num" = "1",
"storage_medium" = "HDD"
)
AS SELECT `lineorder`.`lo_quantity`, `lineorder`.`lo_revenue`, sum(`lineorder`.`lo_tax`) AS `sum`
FROM `ssb_1g`.`lineorder`
WHERE `lineorder`.`lo_linenumber` = 1
GROUP BY 1, 2;

1 row in set
Time: 0.003s
```

### 查看异步物化视图的刷新历史

您可以通过查询 information_schema 数据库中的 task_runs 表来查看异步物化视图的刷新历史。在所有返回的信息中，您应当重点关注以下字段：

- CREATE_TIME 和 FINISH_TIME：刷新任务的开始和结束时间。
- STATE：刷新任务的状态，可能的值包括 PENDING、RUNNING、FAILED 和 SUCCESS。
- ERROR_MESSAGE：刷新任务失败的原因。

示例：

```Plain
MySQL > SELECT * FROM information_schema.task_runs WHERE task_name ='mv-112517' \G
***************************[ 1. row ]***************************
QUERY_ID      | 7434cee5-32a3-11ee-b73a-8e20563011de
TASK_NAME     | mv-112517
CREATE_TIME   | 2023-08-04 16:46:50
FINISH_TIME   | 2023-08-04 16:46:54
STATE         | SUCCESS
DATABASE      | ssb_1g
EXPIRE_TIME   | 2023-08-05 16:46:50
ERROR_CODE    | 0
ERROR_MESSAGE | <null>
PROGRESS      | 100%
EXTRA_MESSAGE | {"forceRefresh":false,"mvPartitionsToRefresh":[],"refBasePartitionsToRefreshMap":{},"basePartitionsToRefreshMap":{}}
PROPERTIES    | {"FORCE":"false"}
***************************[ 2. row ]***************************
QUERY_ID      | 72dd2f16-32a3-11ee-b73a-8e20563011de
TASK_NAME     | mv-112517
CREATE_TIME   | 2023-08-04 16:46:48
FINISH_TIME   | 2023-08-04 16:46:53
STATE         | SUCCESS
DATABASE      | ssb_1g
EXPIRE_TIME   | 2023-08-05 16:46:48
ERROR_CODE    | 0
ERROR_MESSAGE | <null>
PROGRESS      | 100%
EXTRA_MESSAGE | {"forceRefresh":true,"mvPartitionsToRefresh":["mv_pred_2"],"refBasePartitionsToRefreshMap":{},"basePartitionsToRefreshMap":{"lineorder":["lineorder"]}}
PROPERTIES    | {"FORCE":"true"}
```

### 监控异步物化视图的资源消耗

您可以在刷新过程中和刷新完成后监控和分析异步物化视图消耗的资源。

#### 监控刷新期间的资源消耗

在刷新任务进行时，您可以使用 [SHOW PROC '/current_queries'](../sql-reference/sql-statements/Administration/SHOW_PROC.md) 来监控实时资源消耗。

在所有返回的信息中，您应当重点关注以下字段：

- ScanBytes：扫描的数据量大小。
- ScanRows：扫描的数据行数。
- MemoryUsage：使用的内存大小。
- CPUTime：CPU 时间消耗。
- ExecTime：查询的执行时间。

示例：

```Plain
MySQL > SHOW PROC '/current_queries'\G
***************************[ 1. row ]***************************
StartTime     | 2023-08-04 17:01:30
QueryId       | 806eed7d-32a5-11ee-b73a-8e20563011de
ConnectionId  | 0
Database      | ssb_1g
User          | root
ScanBytes     | 70.981 MB
ScanRows      | 6001215 rows
MemoryUsage   | 73.748 MB
DiskSpillSize | 0.000
CPUTime       | 2.515 s
ExecTime      | 2.583 s
```

#### 分析刷新后的资源消耗

在刷新任务完成后，您可以通过查询分析来分析其资源消耗。

当异步物化视图自行刷新时，会执行一个 INSERT OVERWRITE 语句。您可以查看相应的查询分析来分析刷新任务消耗的时间和资源。

在所有返回的信息中，您应当重点关注以下指标：

- Total：查询消耗的总时间。
- QueryCpuCost：查询的总 CPU 时间消耗。CPU 时间消耗是针对并发处理过程进行累加的。因此，这个指标的值可能会大于查询的实际执行时间。
- QueryMemCost：查询的总内存消耗。
- 其他针对各个操作符的指标，如连接操作符和聚合操作符。

有关如何检查查询分析及理解其他指标的详细信息，请参阅[分析查询分析](../administration/query_profile.md)。

### 验证查询是否被异步物化视图改写

您可以通过使用[EXPLAIN](../sql-reference/sql-statements/Administration/EXPLAIN.md)查看查询计划，来检查查询是否可以被异步物化视图改写。

如果查询计划中的 SCAN 指标显示了相应物化视图的名称，则说明查询已被物化视图改写。

示例 1：

```Plain
MySQL > SHOW CREATE TABLE mv_agg\G
***************************[ 1. row ]***************************
Materialized View        | mv_agg
Create Materialized View | CREATE MATERIALIZED VIEW `mv_agg` (`c_custkey`)
DISTRIBUTED BY RANDOM
REFRESH ASYNC
PROPERTIES (
"replication_num" = "1",
"replicated_storage" = "true",
"storage_medium" = "HDD"
)
AS SELECT `customer`.`c_custkey`
FROM `ssb_1g`.`customer`
GROUP BY `customer`.`c_custkey`;

MySQL > EXPLAIN LOGICAL SELECT `customer`.`c_custkey`
                      -> FROM `ssb_1g`.`customer`
                      -> GROUP BY `customer`.`c_custkey`;
+-----------------------------------------------------------------------------------+
| Explain String                                                                    |
+-----------------------------------------------------------------------------------+
| - Output => [1:c_custkey]                                                         |
|     - SCAN [mv_agg] => [1:c_custkey]                                              |
|             Estimates: {row: 30000, cpu: ?, memory: ?, network: ?, cost: 15000.0} |
|             partitionRatio: 1/1, tabletRatio: 12/12                               |
|             1:c_custkey := 10:c_custkey                                           |
+-----------------------------------------------------------------------------------+
```

如果您禁用了查询改写功能，StarRocks 将采用常规查询计划。

示例 2：

```Plain
MySQL > SET enable_materialized_view_rewrite = false;
MySQL > EXPLAIN LOGICAL SELECT `customer`.`c_custkey`
                      -> FROM `ssb_1g`.`customer`
                      -> GROUP BY `customer`.`c_custkey`;
+---------------------------------------------------------------------------------------+
| Explain String                                                                        |
+---------------------------------------------------------------------------------------+
| - Output => [1:c_custkey]                                                             |
|     - AGGREGATE(GLOBAL) [1:c_custkey]                                                 |
|             Estimates: {row: 15000, cpu: ?, memory: ?, network: ?, cost: 120000.0}    |
|         - SCAN [mv_bitmap] => [1:c_custkey]                                           |
|                 Estimates: {row: 60000, cpu: ?, memory: ?, network: ?, cost: 30000.0} |
|                 partitionRatio: 1/1, tabletRatio: 12/12                               |
+---------------------------------------------------------------------------------------+
```

## 诊断并解决问题

这里我们列举了在使用异步物化视图时可能遇到的一些常见问题及其相应的解决方案。

### 创建异步物化视图失败

如果您未能成功创建异步物化视图，即 CREATE MATERIALIZED VIEW 语句无法执行，您可以考虑以下几个方面：

- **检查是否误用了同步物化视图的 SQL 语句。**

  StarRocks 提供了两种不同的物化视图：同步物化视图和异步物化视图。

  创建同步物化视图的基本 SQL 语句如下所示：

  ```SQL
  CREATE MATERIALIZED VIEW <mv_name> 
  AS <query>
  ```

  然而，创建异步物化视图的 SQL 语句则包含更多参数：

  ```SQL
  CREATE MATERIALIZED VIEW <mv_name> 
  REFRESH ASYNC -- The refresh strategy of the asynchronous materialized view.
  DISTRIBUTED BY HASH(<column>) -- The data distribution strategy of the asynchronous materialized view.
  AS <query>
  ```

  除了 SQL 语句之外，两种物化视图的主要区别在于，异步物化视图支持 StarRocks 提供的所有查询语法，而同步物化视图仅支持有限选择的聚合函数。

- **检查您是否指定了正确的 `Partition By` 列。**

  在创建异步物化视图时，您可以指定分区策略，从而可以更精细地刷新物化视图。

  当前，StarRocks 仅支持范围分区，并且只支持引用查询语句中 SELECT 表达式的单个列来构建物化视图。您可以使用 date_trunc() 函数来截断列，以改变分区策略的粒度级别。请注意，不支持其他任何表达式。

- **检查您是否拥有创建物化视图所需的权限**。

  在创建异步物化视图时，您需要对所有查询对象（表、视图、物化视图）具有 SELECT 权限。如果查询中使用了 UDF，您还需要对这些函数具有 USAGE 权限。

### 物化视图刷新失败

如果物化视图刷新失败，即刷新任务的状态不是 SUCCESS，您可以考虑以下几个方面：

- **检查是否采用了不适当的刷新策略。**

  默认情况下，物化视图在创建后会立即刷新。但在 v2.5 及之前的版本中，采用 MANUAL 刷新策略的物化视图在创建后不会自动刷新。您必须使用 REFRESH MATERIALIZED VIEW 命令手动进行刷新。

- **检查刷新任务是否超出了内存限制。**

  通常，当异步物化视图涉及大规模聚合或连接计算，可能会耗尽内存资源。为了解决这个问题，您可以：

-   为物化视图指定分区策略，逐个分区进行刷新。
-   为刷新任务启用磁盘溢出功能。从 v3.1 版本开始，StarRocks 支持在刷新物化视图时将中间结果溢出到磁盘。执行以下语句以启用磁盘溢出功能：

    ```SQL
    SET enable_spill = true;
    ```

- **检查刷新任务是否超过了超时时长。**

  大规模物化视图可能因为刷新任务超过了超时时长而刷新失败。为了解决这个问题，您可以：

  - 为物化视图指定分区策略，逐个分区进行刷新。
  - 设置更长的超时时长。

从 v3.0 版本开始，您可以在创建物化视图时或通过使用 ALTER MATERIALIZED VIEW 命令来定义以下属性（会话变量）。

示例：

```SQL
-- Define the properties when creating the materialized view
CREATE MATERIALIZED VIEW mv1 
REFRESH ASYNC
PROPERTIES ( 'session.enable_spill'='true' )
AS <query>;

-- Add the properties.
ALTER MATERIALIZED VIEW mv2 
    SET ('session.enable_spill' = 'true');
```

### 物化视图状态不活跃

如果物化视图无法改写查询或刷新，并且其 is_active 状态为 false，可能是因为基表架构发生了变化。为了解决这个问题，您可以通过执行以下语句手动将物化视图状态设置为活跃：

```SQL
ALTER MATERIALIZED VIEW mv1 ACTIVE;
```

如果将物化视图状态设置为活跃后没有生效，您可能需要删除并重新创建物化视图。

### 物化视图刷新任务占用了过多资源

如果您发现刷新任务占用了过多的系统资源，可以考虑以下几个方面：

- **检查是否创建了过大的物化视图**。

  如果您连接了过多的表，导致计算量巨大，刷新任务将占用大量资源。为了解决这个问题，您需要评估物化视图的大小并重新规划。

- **检查是否设置了不必要的频繁刷新间隔。**

  如果您采用了固定间隔刷新策略，可以通过设置较低的刷新频率来解决问题。如果刷新任务是由基表数据变化触发的，频繁的数据加载也可能导致这个问题。为了解决这个问题，您需要为物化视图定义一个合适的刷新策略。

- **检查**物化视图是否进行了分区。

  未分区的物化视图刷新成本可能很高，因为 StarRocks 每次都会刷新整个物化视图。为了解决这个问题，您需要为物化视图指定一个分区策略，每次只刷新一个分区。

如果需要停止占用过多资源的刷新任务，您可以：

- 将物化视图状态设置为非活跃，从而停止所有的刷新任务：

  ```SQL
  ALTER MATERIALIZED VIEW mv1 INACTIVE;
  ```

- 使用 SHOW PROCESSLIST 和 KILL 命令终止正在运行的刷新任务：

  ```SQL
  -- Get the ConnectionId of the running refresh task.
  SHOW PROCESSLIST;
  -- Terminate the running refresh task.
  KILL QUERY <ConnectionId>;
  ```

### 物化视图无法改写查询

如果您的物化视图未能改写相关查询，您可以考虑以下几个方面：

- **检查物化视图和查询是否匹配。**

  - StarRocks 使用基于结构的匹配技术而不是基于文本的匹配技术来匹配物化视图和查询。因此，并不能保证看起来相似的查询就一定能被物化视图改写。
  - 物化视图只能改写 SPJG（选择/投影/连接/聚合）类型的查询。不支持涉及窗口函数、嵌套聚合或连接加聚合的查询。
  - 物化视图无法改写涉及外连接中复杂连接谓词的查询。例如，在类似 A LEFT JOIN B ON A.dt > '2023-01-01' AND A.id = B.id 的情况下，我们建议您在 WHERE 子句中指定 JOIN ON 子句的谓词。

  有关物化视图查询改写限制的更多信息，请参阅[Query rewrite with materialized views - Limitations](./query_rewrite_with_materialized_views.md#limitations)。

- **检查物化视图状态是否为活跃。**

  StarRocks 在改写查询前会检查物化视图的状态。只有当物化视图状态为活跃时，查询才能被改写。为了解决这个问题，您可以通过执行以下语句手动将物化视图状态设置为活跃：

  ```SQL
  ALTER MATERIALIZED VIEW mv1 ACTIVE;
  ```

- **检查物化视图是否满足数据一致性的要求。**

  StarRocks 会检查物化视图和基表数据的一致性。默认情况下，只有当物化视图中的数据是最新的时候，查询才能被改写。为了解决这个问题，您可以：

  - 向物化视图添加 PROPERTIES('query_rewrite_consistency'='LOOSE') 来禁用一致性检查。
  - 添加 PROPERTIES('mv_rewrite_staleness_second'='5') 来容忍一定程度的数据不一致。如果最后一次刷新发生在这个时间间隔之前，无论基表数据是否发生变化，查询都可以被改写。

- **检查物化视图的查询语句是否缺少输出列。**

  为了改写范围查询和点查询，您必须在物化视图查询语句的 SELECT 表达式中指定用作过滤条件的列。您需要检查物化视图的 SELECT 语句，确保它包括了查询的 WHERE 和 ORDER BY 子句中引用的列。

示例 1：物化视图 mv1 使用了嵌套聚合，因此它无法用于改写查询。

```SQL
CREATE MATERIALIZED VIEW mv1 REFRESH ASYNC AS
select count(distinct cnt) 
from (
    select c_city, count(*) cnt 
    from customer 
    group by c_city
) t;
```

示例 2：物化视图 mv2 使用了连接加聚合，因此它无法用于改写查询。为了解决这个问题，您可以先创建一个带有聚合的物化视图，然后基于它创建一个带有连接的嵌套物化视图。

```SQL
CREATE MATERIALIZED VIEW mv2 REFRESH ASYNC AS
select *
from (
    select lo_orderkey, lo_custkey, p_partkey, p_name
    from lineorder
    join part on lo_partkey = p_partkey
) lo
join (
    select c_custkey
    from customer
    group by c_custkey
) cust
on lo.lo_custkey = cust.c_custkey;
```

示例 3：物化视图 mv3 无法改写类似 SELECT c_city, sum(tax) FROM tbl WHERE dt='2023-01-01' AND c_city = 'xxx' 的查询模式，因为谓词引用的列不在 SELECT 表达式中。

```SQL
CREATE MATERIALIZED VIEW mv3 REFRESH ASYNC AS
SELECT c_city, sum(tax) FROM tbl GROUP BY c_city;
```

为了解决这个问题，您可以按以下方式创建物化视图：

```SQL
CREATE MATERIALIZED VIEW mv3 REFRESH ASYNC AS
SELECT dt, c_city, sum(tax) FROM tbl GROUP BY dt, c_city;
```
