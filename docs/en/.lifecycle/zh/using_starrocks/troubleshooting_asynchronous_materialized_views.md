---
displayed_sidebar: English
---

# 异步物化视图故障排除

本主题描述了如何检查异步物化视图并解决在使用它们时遇到的问题。

> **注意**
>
> 下面展示的部分功能仅在 StarRocks v3.1 以上版本支持。

## 检查异步物化视图

本节介绍如何检查异步物化视图的工作状态、刷新历史记录以及解决在使用它们时遇到的问题。

### 检查异步物化视图的工作状态

您可以使用 [SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md) 来检查异步物化视图的工作状态。在返回的所有信息中，您可以重点关注以下字段：

- `is_active`：物化视图的状态是否为活动状态。只有活动的物化视图才能用于查询加速和重写。
- `last_refresh_state`：上次刷新的状态，包括 PENDING、RUNNING、FAILED 和 SUCCESS。
- `last_refresh_error_message`：上次刷新失败的原因（如果物化视图状态不是活动状态）。
- `rows`：物化视图中的数据行数。请注意，此值可能与物化视图的实际行数不同，因为更新可以被延迟。

有关返回的其他字段的详细信息，请参阅 [SHOW MATERIALIZED VIEWS - Returns](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md#returns)。

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

### 查看异步物化视图的刷新历史记录

您可以通过查询 `information_schema` 数据库中的 `task_runs` 表来查看异步物化视图的刷新历史记录。在返回的所有信息中，您可以重点关注以下字段：

- `CREATE_TIME` 和 `FINISH_TIME`：刷新任务的开始和结束时间。
- `STATE`：刷新任务的状态，包括 PENDING、RUNNING、FAILED 和 SUCCESS。
- `ERROR_MESSAGE`：刷新任务失败的原因。

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

### 监视异步物化视图的资源消耗

您可以在刷新期间和之后监视和分析异步物化视图所消耗的资源。

#### 监视刷新期间的资源消耗

在刷新任务期间，您可以使用 [SHOW PROC '/current_queries'](../sql-reference/sql-statements/Administration/SHOW_PROC.md) 来监视其实时资源消耗。在返回的所有信息中，您可以重点关注以下字段：

- `ScanBytes`：扫描的数据大小。
- `ScanRows`：扫描的数据行数。
- `MemoryUsage`：使用的内存大小。
- `CPUTime`：CPU 时间成本。
- `ExecTime`：查询的执行时间。

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

在刷新任务后，您可以通过查询配置文件来分析其资源消耗情况。

在异步物化视图刷新自身时，将执行 INSERT OVERWRITE 语句。您可以查看对应的查询配置文件，分析刷新任务所消耗的时间和资源。

在返回的所有信息中，您可以关注以下指标：

- `Total`：查询消耗的总时间。
- `QueryCpuCost`：查询的总 CPU 时间成本。CPU 时间成本会被并发进程聚合。因此，此指标的值可能大于查询的实际执行时间。
- `QueryMemCost`：查询的总内存成本。
- 其他运算符的指标，例如联接运算符和聚合运算符。

有关如何检查查询配置文件和了解其他指标的详细信息，请参阅 [分析查询配置文件](../administration/query_profile.md)。

### 验证查询是否被异步物化视图重写

您可以使用 [EXPLAIN](../sql-reference/sql-statements/Administration/EXPLAIN.md) 来检查查询计划中是否可以使用异步物化视图重写查询。如果查询计划中的指标 `SCAN` 显示对应物化视图的名称，则表示查询已被物化视图重写。

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

如果关闭查询重写功能，StarRocks 将采用常规查询计划。

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

## 诊断和解决问题

以下是您在使用异步物化视图时可能遇到的一些常见问题，以及相应的解决方案。

### 无法创建异步物化视图

如果无法创建异步物化视图，即无法执行 CREATE MATERIALIZED VIEW 语句，您可以查看以下几个方面：

- **检查是否错误地使用了同步物化视图的 SQL 语句。**
  
  StarRocks 提供两种不同的物化视图：同步物化视图和异步物化视图。

  用于创建同步物化视图的基本 SQL 语句如下：

  ```SQL
  CREATE MATERIALIZED VIEW <mv_name> 
  AS <query>
  ```

  然而，用于创建异步物化视图的 SQL 语句包含更多参数：

  ```SQL
  CREATE MATERIALIZED VIEW <mv_name> 
  REFRESH ASYNC -- 异步物化视图的刷新策略。
  DISTRIBUTED BY HASH(<column>) -- 异步物化视图的数据分布策略。
  AS <query>
  ```

  除了 SQL 语句之外，两个物化视图的主要区别在于异步物化视图支持 StarRocks 提供的所有查询语法，但同步物化视图仅支持有限的聚合函数选择。

- **检查是否已指定正确的 `Partition By` 列。**

  创建异步物化视图时，可以指定分区策略，从而可以在更精细的粒度级别上刷新物化视图。

  目前，StarRocks 仅支持范围分区，并且仅支持在构建物化视图的查询语句中引用 SELECT 表达式中的单个列。您可以使用 date_trunc() 函数截断列以更改分区策略的粒度级别。请注意，不支持任何其他表达式。

- **检查是否具有创建物化视图所需的必要权限。**

  创建异步物化视图时，需要查询的所有对象（表、视图、物化视图）的 SELECT 权限。在查询中使用 UDF 时，还需要函数的 USAGE 权限。

### 物化视图刷新失败

如果物化视图无法成功刷新，即刷新任务的状态不是 SUCCESS，您可以查看以下几个方面：

- **检查是否采用了不适当的刷新策略。**

  默认情况下，物化视图在创建后会立即刷新。但是，在 v2.5 及更早版本中，采用 MANUAL 刷新策略的物化视图在创建后不会刷新。您必须使用 REFRESH MATERIALIZED VIEW 手动刷新它。

- **检查刷新任务是否超出内存限制。**

  通常，当异步物化视图涉及大规模聚合或联接计算耗尽内存资源时。要解决此问题，您可以：

  - 为物化视图指定分区策略，以便每次刷新一个分区。
  - 启用“溢出到磁盘”功能以解决刷新任务。从 v3.1 开始，StarRocks 支持在刷新物化视图时将中间结果溢出到磁盘。执行以下语句以启用 Spill to Disk：

    ```SQL
    SET enable_spill = true;
    ```

- **检查刷新任务是否超出超时持续时间。**

  大规模物化视图可能因刷新任务超出超时持续时间而无法成功刷新。要解决此问题，您可以：

  - 为物化视图指定分区策略，以便每次刷新一个分区。
  - 设置更长的超时持续时间。

从 v3.0 开始，您可以在创建物化视图时或使用 ALTER MATERIALIZED VIEW 添加以下属性（会话变量）。

示例：

```SQL
-- 创建物化视图时定义属性
CREATE MATERIALIZED VIEW mv1 
REFRESH ASYNC
PROPERTIES ( 'session.enable_spill'='true' )
AS <query>;

-- 添加属性。
ALTER MATERIALIZED VIEW mv2 
    SET ('session.enable_spill' = 'true');
```

### 物化视图状态非活动

如果物化视图无法重写查询或刷新，并且物化视图的状态 `is_active` 为 `false`，可能是基表架构更改的结果。要解决此问题，可以通过执行以下语句手动将物化视图状态设置为活动状态：

```SQL
ALTER MATERIALIZED VIEW mv1 ACTIVE;
```

如果将物化视图状态设置为活动状态未生效，则需要删除该物化视图并重新创建。

### 物化视图刷新任务使用过多资源

如果发现刷新任务占用了过多的系统资源，可以查看以下几个方面：

- **检查是否创建了超大的物化视图。**

  如果联接了太多表导致大量计算，刷新任务将占用大量资源。要解决此问题，您需要评估物化视图的大小并重新规划。

- **检查是否设置了不必要的频繁刷新间隔。**

  如果采用固定间隔刷新策略，可以设置较低的刷新频率来解决问题。如果刷新任务是由基表中的数据更改触发的，则加载数据过于频繁也会导致此问题。若要解决此问题，需要为物化视图定义适当的刷新策略。

- **检查物化视图是否分区。**

  未分区的物化视图刷新成本可能很高，因为 StarRocks 每次都会刷新整个物化视图。要解决此问题，您需要为物化视图指定分区策略，以便每次刷新一个分区。

要停止占用过多资源的刷新任务，可以：

- 将物化视图状态设置为非活动状态，以便停止其所有刷新任务：

  ```SQL
  ALTER MATERIALIZED VIEW mv1 INACTIVE;
  ```

- 使用 SHOW PROCESSLIST 和 KILL 终止正在运行的刷新任务：

  ```SQL
  -- 获取正在运行的刷新任务的 ConnectionId。
  SHOW PROCESSLIST;
  -- 终止正在运行的刷新任务。
  KILL QUERY <ConnectionId>;
  ```

### 物化视图无法重写查询

如果您的物化视图无法重写相关查询，可以查看以下几个方面：

- **检查物化视图和查询是否匹配。**

  - StarRocks 使用基于结构的匹配技术而不是基于文本的匹配技术来匹配物化视图和查询。因此，不能保证查询可以仅仅因为它看起来与物化视图相似而重写。
  - 物化视图只能重写 SPJG（Select/Projection/Join/Aggregation）类型的查询。不支持涉及窗口函数、嵌套聚合或联接加聚合的查询。
  - 物化视图无法重写涉及外部联接中复杂联接谓词的查询。例如，在类似 的情况下，我们建议您在子句`A LEFT JOIN B ON A.dt > '2023-01-01' AND A.id = B.id`中指定子句中的`JOIN ON`谓词 `WHERE` 。

  有关物化视图查询重写的限制的详细信息，请参阅使用物化视图重写查询 - 限制。

- **检查物化视图状态是否为活动状态。**

  StarRocks 在重写查询之前会检查物化视图的状态。仅当物化视图状态为活动状态时，才能重写查询。要解决此问题，可以通过执行以下语句手动将物化视图状态设置为活动状态：

  ```SQL
  ALTER MATERIALIZED VIEW mv1 ACTIVE;
  ```

- **检查物化视图是否满足数据一致性要求。**

  StarRocks 会检查物化视图和基表数据的一致性。默认情况下，仅当物化视图中的数据是最新的时，才能重写查询。要解决此问题，您可以：

  - 添加到 `PROPERTIES('query_rewrite_consistency'='LOOSE')` 物化视图以禁用一致性检查。
  - 添加 `PROPERTIES('mv_rewrite_staleness_second'='5')` 以容忍一定程度的数据不一致。如果上次刷新在此时间间隔之前，则无论基表中的数据是否更改，都可以重写查询。

- **检查物化视图的查询语句是否缺少输出列。**

  要重写范围和点查询，必须在物化视图的查询语句的 SELECT 表达式中指定用作筛选谓词的列。您需要检查物化视图的 SELECT 语句，以确保它包含查询的 WHERE 和 ORDER BY 子句中引用的列。

示例 1：物化视图 `mv1` 使用嵌套聚合。因此，它不能用于重写查询。

```SQL
CREATE MATERIALIZED VIEW mv1 REFRESH ASYNC AS
select count(distinct cnt) 
from (
    select c_city, count(*) cnt 
    from customer 
    group by c_city
) t;
```

示例 2：物化视图 `mv2` 使用联接加聚合。因此，它不能用于重写查询。要解决这个问题，您可以创建一个具有聚合的物化视图，然后基于前一个具有连接的嵌套物化视图。

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
示例 3：物化视图 `mv3` 无法按照`SELECT c_city, sum(tax) FROM tbl WHERE dt='2023-01-01' AND c_city = 'xxx'`的模式重写查询，因为谓词引用的列不在SELECT表达式中。

```SQL
CREATE MATERIALIZED VIEW mv3 REFRESH ASYNC AS
SELECT c_city, sum(tax) FROM tbl GROUP BY c_city;
```

要解决这个问题，您可以按以下方式创建物化视图：

```SQL
CREATE MATERIALIZED VIEW mv3 REFRESH ASYNC AS
SELECT dt, c_city, sum(tax) FROM tbl GROUP BY dt, c_city;