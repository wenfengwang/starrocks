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

### 查看异步材化视图的刷新历史

您可以通过查询数据库 `information_schema` 中的表 `task_runs` 来查看异步材化视图的刷新历史。在返回的所有信息中，可以关注以下字段：

- `CREATE_TIME` 和 `FINISH_TIME`: 刷新任务的开始和结束时间。
- `STATE`: 刷新任务的状态，包括 PENDING、RUNNING、FAILED 和 SUCCESS。
- `ERROR_MESSAGE`: 刷新任务失败的原因。

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

### 监视异步材化视图的资源消耗

您可以在刷新期间和刷新后监视和分析异步材化视图消耗的资源量。

#### 在刷新期间监视资源消耗

在刷新任务期间，您可以使用 [SHOW PROC '/current_queries'](../sql-reference/sql-statements/Administration/SHOW_PROC.md) 来监视其实时资源消耗。

在返回的所有信息中，可以关注以下字段：

- `ScanBytes`: 扫描的数据大小。
- `ScanRows`: 扫描的数据行数。

- `MemoryUsage`: 使用的内存大小。

- `CPUTime`: CPU 时间成本。

- `ExecTime`: 查询的执行时间。

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

#### 在刷新后分析资源消耗

在刷新任务完成后，可以通过查询结果来分析其资源消耗。

当异步材化视图刷新时，会执行一个 INSERT OVERWRITE 语句。您可以检查相应的查询结果来分析刷新任务消耗的时间和资源。


在返回的所有信息中，可以关注以下指标：

- `Total`: 查询消耗的总时间。

- `QueryCpuCost`: 查询的总 CPU 时间成本。CPU 时间成本被并行进程汇总。因此，此指标的值可能大于查询实际执行时间。

- `QueryMemCost`: 查询的总内存成本。

- 其他个别操作符的指标，如连接操作符和聚合操作符等。

有关如何检查查询结果并了解其他指标的详细信息，请参阅 [分析查询结果](../administration/query_profile.md)。


### 验证是否存在异步材化视图重写的查询

您可以通过查询计划来查看查询是否通过异步材化视图重写。

如果查询计划中的 `SCAN` 指标显示了相应材化视图的名称，那么该查询已经被材化视图重写。

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
| partitionRatio: 1/1, tabletRatio: 12/12 |
           1:c_custkey := 10:c_custkey           
+-----------------------------------------------------------------------------------+


如果禁用查询重写功能，StarRocks将采用常规查询计划。

示例2：

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

以下列出了在使用异步物化视图时可能遇到的一些常见问题以及相应的解决方案。

### 无法创建异步物化视图

如果无法创建异步物化视图，即CREATE MATERIALIZED VIEW语句无法执行，您可以查看以下几个方面：

- **检查是否错误地使用了用于同步物化视图的SQL语句。**

  StarRocks提供了两种不同的物化视图：同步物化视图和异步物化视图。

  创建同步物化视图的基本SQL语句如下：

  ```SQL
  CREATE MATERIALIZED VIEW <mv_name>
  AS <query>
  ```

  然而，创建异步物化视图的SQL语句包含更多的参数：

  ```SQL
  CREATE MATERIALIZED VIEW <mv_name>
  REFRESH ASYNC -- 异步物化视图的刷新策略。
  DISTRIBUTED BY HASH(<column>) -- 异步物化视图的数据分发策略。
  AS <query>
  ```

  除了SQL语句之外，两种物化视图的主要区别在于异步物化视图支持StarRocks提供的所有查询语法，而同步物化视图仅支持有限选择的聚合函数。

- **检查是否指定了正确的分区列。**

  在创建异步物化视图时，您可以指定分区策略，从而可以在更细粒度的级别上刷新物化视图。

  目前，StarRocks仅支持范围分区，并且仅支持引用查询语句中用于构建物化视图的SELECT表达式中的单个列。您可以使用date_trunc()函数来截断列以改变分区策略的粒度级别。请注意，不支持任何其他表达式。

- **检查是否具有创建物化视图所需的权限。**

  在创建异步物化视图时，您需要具有所有被查询的对象（表、视图、物化视图）的SELECT权限。当查询中使用UDFs时，还需要函数的USAGE权限。

### 物化视图刷新失败

如果物化视图刷新失败，即刷新任务的状态不是SUCCESS，您可以查看以下几个方面：

- **检查是否选择了不合适的刷新策略。**

  默认情况下，物化视图在创建后立即刷新。然而，在v2.5和较早的版本中，采用手动刷新策略的物化视图不会在创建后自动刷新。您必须使用REFRESH MATERIALIZED VIEW手动刷新它。

- **检查刷新任务是否超出了内存限制。**

  通常情况下，当异步物化视图涉及大规模聚合或连接计算时会耗尽内存资源。为解决此问题，您可以：

  - 为物化视图指定分区策略，每次刷新一个分区。
  - 在v3.1及以后的版本中，StarRocks支持将中间结果刷新到磁盘。执行以下语句以启用Spill to Disk：

    ```SQL
    SET enable_spill = true;
    ```

- **检查刷新任务是否超出了超时时长。**

  大规模物化视图可能因刷新任务超出超时时长而导致刷新失败。解决此问题，您可以：

  - 为物化视图指定分区策略，每次刷新一个分区。
  - 设置较长的超时时长。

从v3.0开始，您可以在创建物化视图或使用ALTER MATERIALIZED VIEW添加以下属性（会话变量）。

示例：

```SQL
-- 在创建物化视图时定义属性
CREATE MATERIALIZED VIEW mv1
REFRESH ASYNC
PROPERTIES ('session.enable_spill'='true')
AS <query>;

-- 添加属性。
ALTER MATERIALIZED VIEW mv2
    SET ('session.enable_spill' = 'true');
```

### 物化视图状态不是活动的

如果物化视图无法重写查询或刷新，且物化视图的状态`is_active`是`false`，这可能是由基表结构更改造成的。为解决此问题，您可以通过执行以下语句手动将物化视图状态设置为活动：

```SQL
ALTER MATERIALIZED VIEW mv1 ACTIVE;
```

如果将物化视图状态设置为活动没有生效，您需要删除物化视图并重新创建。

### 物化视图刷新任务使用了过多的资源

如果发现刷新任务正在使用过多系统资源，您可以查看以下几个方面：

- **检查是否创建了过大的物化视图。**

  如果连接了太多的表导致了大量计算，刷新任务将占用大量资源。解决此问题，您需要评估物化视图的大小并重新规划。

- **检查是否设置了不必要频繁的刷新间隔。**

  如果采用了固定间隔刷新策略，可以设置较低的刷新频率来解决问题。如果刷新任务是由基表数据更改触发的，过于频繁地加载数据也会导致此问题。解决此问题，您需要为物化视图定义适当的刷新策略。

- **检查物化视图是否分区。**

  未分区的物化视图刷新成本高，因为StarRocks每次都会刷新整个物化视图。为解决此问题，您需要为物化视图指定分区策略，每次刷新一个分区。

要停止占用过多资源的刷新任务，您可以：

- 将物化视图状态设置为非活动状态，以停止其所有刷新任务：

  ```SQL
  ALTER MATERIALIZED VIEW mv1 INACTIVE;
  ```

- 使用SHOW PROCESSLIST和KILL终止运行中的刷新任务：

  ```SQL
  -- 获取运行中刷新任务的ConnectionId。
  SHOW PROCESSLIST;
  -- 终止运行中的刷新任务。
  KILL QUERY <ConnectionId>;
  ```

### 物化视图无法重写查询

如果您的物化视图无法重写相关查询，您可以查看以下几个方面：

- **检查物化视图和查询是否匹配。**

  - StarRocks根据基于结构的匹配技术而不是基于文本的匹配来匹配物化视图和查询。因此，并不保证查询可以被重写，仅因为它与物化视图相似。
  - 物化视图仅能重写SPJG（选择/投影/连接/聚合）类型的查询。不支持涉及窗口函数、嵌套聚合或连接加聚合的查询。
  - 物化视图不能重写涉及外连接中复杂连接谓词的查询。例如，在`A LEFT JOIN B ON A.dt > '2023-01-01' AND A.id = B.id`等情况下，建议您将`JOIN ON`子句中的谓词改为`WHERE`子句中。

  有关物化视图查询重写限制的更多信息，请参见[物化视图查询重写的限制](./query_rewrite_with_materialized_views.md#limitations)。

- **检查物化视图状态是否活动。**

  StarRocks在重写查询之前会检查物化视图的状态。只有在物化视图状态为活动时才能重写查询。解决此问题，您可以通过执行以下语句手动将物化视图状态设置为活动：

  ```SQL
  ALTER MATERIALIZED VIEW mv1 ACTIVE;
  ```

- **检查物化视图是否符合数据一致性要求。**

  StarRocks检查物化视图中的数据与基表数据的一致性。默认情况下，只有在物化视图中的数据是最新的时，才能重写查询。解决此问题，您可以：

- 向物化视图添加`PROPERTIES('query_rewrite_consistency'='LOOSE')`以禁用一致性检查。
- 向物化视图添加`PROPERTIES('mv_rewrite_staleness_second'='5')`以容忍一定程度的数据不一致。如果最后一次刷新在此时间间隔之前，不管基表中的数据是否发生变化，查询都可以重写。

- **检查物化视图的查询语句是否缺少输出列。**

  要重写范围和点查询，必须在物化视图查询语句的SELECT表达式中指定用作过滤谓词的列。您需要检查物化视图的SELECT语句，确保它包括查询的WHERE和ORDER BY子句中引用的列。

示例1：物化视图`mv1`使用了嵌套聚合。因此，它无法用于重写查询。

```SQL
CREATE MATERIALIZED VIEW mv1 REFRESH ASYNC AS
select count(distinct cnt) 
from (
    select c_city, count(*) cnt 
    from customer 
    group by c_city
) t;
```

示例2：物化视图`mv2`使用了连接加聚合。因此，它无法用于重写查询。要解决这个问题，您可以创建一个带有聚合的物化视图，然后基于前一个物化视图创建一个基于连接的嵌套物化视图。

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

示例3：物化视图`mv3`无法重写`SELECT c_city, sum(tax) FROM tbl WHERE dt='2023-01-01' AND c_city = 'xxx'`这种模式的查询，因为谓词引用的列不在SELECT表达式中。

```SQL
CREATE MATERIALIZED VIEW mv3 REFRESH ASYNC AS
SELECT c_city, sum(tax) FROM tbl GROUP BY c_city;
```

要解决这个问题，您可以如下创建物化视图：

```SQL
CREATE MATERIALIZED VIEW mv3 REFRESH ASYNC AS
SELECT dt, c_city, sum(tax) FROM tbl GROUP BY dt, c_city;
```