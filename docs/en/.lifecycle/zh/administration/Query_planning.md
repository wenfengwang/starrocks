---
displayed_sidebar: English
---

# 查询分析

如何优化查询性能是一个常见问题。缓慢的查询不仅会损害用户体验，也会影响集群性能。因此，分析并优化查询性能非常重要。

您可以在 `fe/log/fe.audit.log` 中查看查询信息。每个查询对应一个 `QueryID`，可用于搜索查询的 `QueryPlan` 和 `Profile`。`QueryPlan` 是 FE 通过解析 SQL 语句生成的执行计划。`Profile` 是 BE 执行结果，包含了每一步消耗的时间和每一步处理的数据量等信息。

## 计划分析

在 StarRocks 中，一个 SQL 语句的生命周期可以分为三个阶段：查询解析、查询规划和查询执行。查询解析通常不是性能瓶颈，因为分析型工作负载所需的 QPS 并不高。

StarRocks 中的查询性能由查询规划和查询执行决定。查询规划负责协调操作符（Join/Order/Aggregate），而查询执行负责执行具体的操作。

查询计划为 DBA 提供了一个宏观视角来访问查询信息。查询计划是查询性能的关键，也是 DBA 参考的重要资源。以下代码片段以 `TPCDS query96` 为例，展示如何查看查询计划。

```SQL
-- query96.sql
select  count(*)
from store_sales
    ,household_demographics
    ,time_dim
    , store
where ss_sold_time_sk = time_dim.t_time_sk
    and ss_hdemo_sk = household_demographics.hd_demo_sk
    and ss_store_sk = s_store_sk
    and time_dim.t_hour = 8
    and time_dim.t_minute >= 30
    and household_demographics.hd_dep_count = 5
    and store.s_store_name = 'ese'
order by count(*) limit 100;
```

有两种类型的查询计划——逻辑查询计划和物理查询计划。这里所讨论的查询计划指的是逻辑查询计划。`TPCDS query96.sql` 对应的查询计划如下所示。

```sql
+------------------------------------------------------------------------------+
| Explain String                                                               |
+------------------------------------------------------------------------------+
| PLAN FRAGMENT 0                                                              |
|  OUTPUT EXPRS:<slot 11>                                                      |
|   PARTITION: UNPARTITIONED                                                   |
|   RESULT SINK                                                                |
|   12:MERGING-EXCHANGE                                                        |
|      limit: 100                                                              |
|      tuple ids: 5                                                            |
|                                                                              |
| PLAN FRAGMENT 1                                                              |
|  OUTPUT EXPRS:                                                               |
|   PARTITION: RANDOM                                                          |
|   STREAM DATA SINK                                                           |
|     EXCHANGE ID: 12                                                          |
|     UNPARTITIONED                                                            |
|                                                                              |
|   8:TOP-N                                                                    |
|   |  order by: <slot 11> ASC                                                 |
|   |  offset: 0                                                               |
|   |  limit: 100                                                              |
|   |  tuple ids: 5                                                            |
|   |                                                                          |
|   7:AGGREGATE (update finalize)                                              |
|   |  output: count(*)                                                        |
|   |  group by:                                                               |
|   |  tuple ids: 4                                                            |
|   |                                                                          |
|   6:HASH JOIN                                                                |
|   |  join op: INNER JOIN (BROADCAST)                                         |
|   |  hash predicates:                                                        |
|   |  colocate: false, reason: left hash join node cannot do colocate         |
|   |  equal join conjunct: `ss_store_sk` = `s_store_sk`                       |
|   |  tuple ids: 0 2 1 3                                                      |
|   |                                                                          |
|   |----11:EXCHANGE                                                           |
|   |       tuple ids: 3                                                       |
|   |                                                                          |
|   4:HASH JOIN                                                                |
|   |  join op: INNER JOIN (BROADCAST)                                         |
|   |  hash predicates:                                                        |
|   |  colocate: false, reason: left hash join node cannot do colocate         |
|   |  equal join conjunct: `ss_hdemo_sk`=`household_demographics`.`hd_demo_sk`|
|   |  tuple ids: 0 2 1                                                        |
|   |                                                                          |
|   |----10:EXCHANGE                                                           |
|   |       tuple ids: 1                                                       |
|   |                                                                          |
|   2:HASH JOIN                                                                |
|   |  join op: INNER JOIN (BROADCAST)                                         |
|   |  hash predicates:                                                        |
|   |  colocate: false, reason: table not in same group                        |
|   |  equal join conjunct: `ss_sold_time_sk` = `time_dim`.`t_time_sk`         |
|   |  tuple ids: 0 2                                                          |
|   |                                                                          |
|   |----9:EXCHANGE                                                            |
|   |       tuple ids: 2                                                       |
|   |                                                                          |
|   0:OlapScanNode                                                             |
|      TABLE: store_sales                                                      |
|      PREAGGREGATION: OFF. Reason: `ss_sold_time_sk` is a value column        |
|      partitions=1/1                                                          |
|      rollup: store_sales                                                     |
|      tabletRatio=0/0                                                         |
|      tabletList=                                                             |
|      cardinality=-1                                                          |
|      avgRowSize=0.0                                                          |
|      numNodes=0                                                              |
|      tuple ids: 0                                                            |
|                                                                              |
| PLAN FRAGMENT 2                                                              |
|  OUTPUT EXPRS:                                                               |
|   PARTITION: RANDOM                                                          |
|                                                                              |
|   STREAM DATA SINK                                                           |
|     EXCHANGE ID: 11                                                          |
|     UNPARTITIONED                                                            |
|                                                                              |
|   5:OlapScanNode                                                             |
|      TABLE: store                                                            |
|      PREAGGREGATION: OFF. Reason: null                                       |
|      PREDICATES: `store`.`s_store_name` = 'ese'                              |
|      partitions=1/1                                                          |
|      rollup: store                                                           |
|      tabletRatio=0/0                                                         |
|      tabletList=                                                             |
|      cardinality=-1                                                          |
|      avgRowSize=0.0                                                          |
|      numNodes=0                                                              |
|      tuple ids: 3                                                            |
|                                                                              |
| PLAN FRAGMENT 3                                                              |
|  OUTPUT EXPRS:                                                               |
|   PARTITION: RANDOM                                                          |
|   STREAM DATA SINK                                                           |
|     EXCHANGE ID: 10                                                          |
|     UNPARTITIONED                                                            |
|                                                                              |
|   3:OlapScanNode                                                             |
|      TABLE: household_demographics                                           |
|      PREAGGREGATION: OFF. Reason: null                                       |
|      PREDICATES: `household_demographics`.`hd_dep_count` = 5                 |
|      partitions=1/1                                                          |
|      rollup: household_demographics                                          |
|      tabletRatio=0/0                                                         |
|      tabletList=                                                             |
|      cardinality=-1                                                          |
|      avgRowSize=0.0                                                          |
|      numNodes=0                                                              |
|      tuple ids: 1                                                            |
|                                                                              |
| PLAN FRAGMENT 4                                                              |
|  OUTPUT EXPRS:                                                               |
|   PARTITION: RANDOM                                                          |
|   STREAM DATA SINK                                                           |
|     EXCHANGE ID: 09                                                          |
|     UNPARTITIONED                                                            |
|                                                                              |
|   1:OlapScanNode                                                             |
|      TABLE: time_dim                                                         |
|      PREAGGREGATION: OFF. Reason: null                                       |
|      PREDICATES: `time_dim`.`t_hour` = 8, `time_dim`.`t_minute` >= 30        |
|      partitions=1/1                                                          |
|      rollup: time_dim                                                        |
|      tabletRatio=0/0                                                         |
|      tabletList=                                                             |
|      cardinality=-1                                                          |
|      avgRowSize=0.0                                                          |
|      numNodes=0                                                              |
|      tuple ids: 2                                                            |
+------------------------------------------------------------------------------+
128 行在集合中 (0.02 秒)
```

查询 96 展示了一个涉及多个 StarRocks 概念的查询计划。

|名称|说明|
|---|---|
|avgRowSize|扫描数据行的平均大小|
|cardinality|扫描表中的数据行总数|
|colocate|表是否处于共置模式|
|numNodes|要扫描的节点数量|
|rollup|物化视图|
|preaggregation|预聚合|
|predicates|谓词，查询过滤器|

查询 96 的查询计划被分为五个片段，编号从 0 到 4。查询计划可以按照自下而上的方式逐个阅读。

```
```markdown
# 查询分析

如何优化查询性能是一个常见问题。缓慢的查询会影响用户体验以及集群性能。分析和优化查询性能非常重要。

您可以在 `fe/log/fe.audit.log` 中查看查询信息。每个查询对应一个 `QueryID`，可以用它来搜索查询的 `QueryPlan` 和 `Profile`。`QueryPlan` 是 FE 通过解析 SQL 语句生成的执行计划。`Profile` 是 BE 执行结果，包含每个步骤消耗的时间和每个步骤处理的数据量等信息。

## 计划分析

在 StarRocks 中，一个 SQL 语句的生命周期可以分为三个阶段：查询解析、查询计划和查询执行。查询解析通常不是瓶颈，因为分析型工作负载所需的 QPS 并不高。

StarRocks 中的查询性能由查询计划和查询执行决定。查询计划负责协调操作符（Join/Order/Aggregate），查询执行负责运行具体操作。

查询计划为 DBA 提供了一个宏观视角来访问查询信息。查询计划是查询性能的关键，也是 DBA 参考的好资源。以下代码片段使用 `TPCDS query96` 作为示例展示如何查看查询计划。

```SQL
-- query96.sql
select  count(*)
from store_sales
    ,household_demographics
    ,time_dim
    , store
where ss_sold_time_sk = time_dim.t_time_sk
    and ss_hdemo_sk = household_demographics.hd_demo_sk
    and ss_store_sk = s_store_sk
    and time_dim.t_hour = 8
    and time_dim.t_minute >= 30
    and household_demographics.hd_dep_count = 5
    and store.s_store_name = 'ese'
order by count(*) limit 100;
```

有两种类型的查询计划 - 逻辑查询计划和物理查询计划。这里描述的查询计划指的是逻辑查询计划。`TPCDS query96.sql` 对应的查询计划如下所示。

```sql
-- 省略了查询计划的 SQL 输出，因为它与原文档相同 --
```

查询 96 展示了一个涉及多个 StarRocks 概念的查询计划。

|名称|解释|
|---|---|
|avgRowSize|扫描数据行的平均大小|
|cardinality|扫描表中数据行的总数|
|colocate|表是否处于共置模式|
|numNodes|要扫描的节点数量|
|rollup|物化视图|
|preaggregation|预聚合|
|predicates|谓词，查询过滤器|

查询 96 的查询计划分为五个片段，编号从 0 到 4。查询计划可以自下而上逐个阅读。

片段 4 负责扫描 `time_dim` 表并提前执行相关查询条件（即 `time_dim.t_hour = 8` 且 `time_dim.t_minute >= 30`）。这一步骤也称为谓词下推。StarRocks 决定是否为聚合表启用 `PREAGGREGATION`。在前面的图中，`time_dim` 的预聚合被禁用。在这种情况下，会读取 `time_dim` 的所有维度列，如果表中的维度列很多，这可能会对性能产生负面影响。如果 `time_dim` 表选择范围分区进行数据划分，查询计划中会命中多个分区，不相关的分区会被自动过滤掉。如果存在物化视图，StarRocks 会根据查询自动选择物化视图。如果没有物化视图，查询会自动命中基表（例如前图中的 `rollup: time_dim`）。

扫描完成后，片段 4 结束。数据将被传递到其他片段，如前图中的 `EXCHANGE ID: 09` 所示，传递到标记为 9 的接收节点。

对于查询 96 的查询计划，片段 2、3 和 4 具有相似的功能，但它们负责扫描不同的表。具体来说，查询中的 `Order/Aggregation/Join` 操作是在片段 1 中执行的。

片段 1 使用 `BROADCAST` 方法执行 `Order/Aggregation/Join` 操作，即将小表广播到大表。如果两个表都很大，我们建议您使用 `SHUFFLE` 方法。目前，StarRocks 只支持 `HASH JOIN`。`colocate` 字段用于显示两个连接表是否以相同的方式进行分区和分桶，以便可以在本地进行连接操作，而无需迁移数据。当 Join 操作完成后，将执行上层的 `aggregation`、`order by` 和 `top-n` 操作。

通过移除具体表达式（只保留操作符），可以以更宏观的视角展示查询计划，如下图所示。

![8-5](../assets/8-5.png)

## 查询提示

查询提示是明确建议查询优化器如何执行查询的指令或注释。目前，StarRocks 支持两种类型的提示：变量设置提示和连接提示。提示仅在单个查询内生效。

### 变量设置提示

您可以在 SELECT 和 SUBMIT TASK 语句中，或在其他语句中包含的 SELECT 子句中，以语法 `/*+ SET_VAR(var_name = value) */` 的形式使用 `SET_VAR` 提示来设置一个或多个[系统变量](../reference/System_variable.md)，例如在创建物化视图 **AS SELECT** 和创建视图 **AS SELECT** 时。

#### 语法

```SQL
[...] SELECT [/*+ SET_VAR(key=value [, key = value]*) */] ...
SUBMIT [/*+ SET_VAR(key=value [, key = value]*) */] TASK ...
```

#### 示例

通过设置系统变量 `streaming_preaggregation_mode` 和 `new_planner_agg_stage` 来提示聚合查询的聚合方法。

```SQL
SELECT /*+ SET_VAR (streaming_preaggregation_mode = 'force_streaming', new_planner_agg_stage = '2') */ SUM(sales_amount) AS total_sales_amount FROM sales_orders;
```

通过在 SUBMIT TASK 语句中设置系统变量 `query_timeout` 来提示查询的任务执行超时时间。

```SQL
SUBMIT /*+ SET_VAR(query_timeout=3) */ TASK AS CREATE TABLE temp AS SELECT count(*) AS cnt FROM tbl1;
```

在创建物化视图时，通过在 SELECT 子句中设置系统变量 `query_timeout` 来提示查询的执行超时时间。

```SQL
CREATE MATERIALIZED VIEW mv 
PARTITION BY dt 
DISTRIBUTED BY HASH(`key`) 
BUCKETS 10 
REFRESH ASYNC 
AS SELECT /*+ SET_VAR(query_timeout=500) */ * FROM dual;
```

### 连接提示

对于多表连接查询，优化器通常会选择最优的连接执行方法。在特殊情况下，您可以使用连接提示向优化器明确建议连接执行方法或禁用连接重排序。目前，连接提示支持建议 Shuffle Join、Broadcast Join、Bucket Shuffle Join 或 Colocate Join 作为连接执行方法。使用连接提示时，优化器不会执行连接重排序。因此您需要选择较小的表作为右表。另外，当建议使用 [Colocate Join](../using_starrocks/Colocate_join.md) 或 Bucket Shuffle Join 作为连接执行方式时，请确保连接表的数据分布满足这些连接执行方式的要求。否则，建议的连接执行方法无法生效。

#### 语法

```SQL
... JOIN { [BROADCAST] | [SHUFFLE] | [BUCKET] | [COLOCATE] | [UNREORDER] } ...
```

> **注意**
> Join Hint 不区分大小写。

#### 示例

- Shuffle Join

  如果您需要在 Join 操作之前将 A 表和 B 表中具有相同分桶键值的数据行 shuffle 到同一台机器上，可以将连接执行方法提示为 Shuffle Join。

  ```SQL
  SELECT k1 FROM t1 JOIN [SHUFFLE] t2 ON t1.k1 = t2.k2 GROUP BY t2.k2;
  ```

- Broadcast Join

  如果 A 表是大表，B 表是小表，您可以将连接执行方法提示为 Broadcast Join。B 表的数据将完全广播到 A 表数据所在的机器上，然后进行 Join 操作。与 Shuffle Join 相比，Broadcast Join 节省了 A 表数据 shuffle 的成本。

  ```SQL
  SELECT k1 FROM t1 JOIN [BROADCAST] t2 ON t1.k1 = t2.k2 GROUP BY t2.k2;
  ```

- Bucket Shuffle Join

  如果连接查询中的 Join 等值连接表达式包含 A 表的分桶键，特别是当 A 表和 B 表都是大表时，您可以将连接执行方法提示为 Bucket Shuffle Join。根据 A 表的数据分布，将 B 表的数据 shuffle 到 A 表数据所在的机器上，然后进行 Join 操作。与 Broadcast Join 相比，Bucket Shuffle Join 显著减少了数据传输，因为 B 表的数据只在全局范围内进行了一次 shuffle。

  ```SQL
  SELECT k1 FROM t1 JOIN [BUCKET] t2 ON t1.k1 = t2.k2 GROUP BY t2.k2;
  ```

- Colocate Join

  如果 A 表和 B 表属于建表时指定的同一个 Colocation Group，A 表和 B 表中具有相同分桶键值的数据行分布在同一个 BE 节点上。当连接查询中的 Join 等值连接表达式包含 A 表和 B 表的分桶键时，您可以将连接执行方法提示为 Colocate Join。相同键值的数据直接在本地进行 Join，减少了节点间数据传输的时间，提高了查询性能。

  ```SQL
  SELECT k1 FROM t1 JOIN [COLOCATE] t2 ON t1.k1 = t2.k2 GROUP BY t2.k2;
  ```

### 查看连接执行方法

使用 `EXPLAIN` 命令可以查看实际的连接执行方法。如果返回结果显示连接执行方法与连接提示匹配，则说明连接提示有效。

```SQL
EXPLAIN SELECT k1 FROM t1 JOIN [COLOCATE] t2 ON t1.k1 = t2.k2 GROUP BY t2.k2;
```

![8-9](../assets/8-9.png)

## SQL 指纹

SQL 指纹用于优化慢查询，提高系统资源利用率。StarRocks 利用 SQL 指纹特性对慢查询日志（`fe.audit.log.slow_query`）中的 SQL 语句进行规范化，将 SQL 语句分为不同类型，并计算每种 SQL 类型的 MD5 哈希值来识别慢查询。MD5 哈希值由字段 `Digest` 指定。

```SQL
2021-12-27 15:13:39,108 [slow_query] |Client=172.26.xx.xxx:54956|User=root|Db=default_cluster:test|State=EOF|Time=2469|ScanBytes=0|ScanRows=0|ReturnRows=6|StmtId=3|QueryId=824d8dc0-66e4-11ec-9fdc-00163e04d4c2|IsQuery=true|feIp=172.26.92.195|Stmt=select count(*) from test_basic group by id_bigint|Digest=51390da6b57461f571f0712d527320f4
```

SQL 语句规范化将语句文本转换为更规范的格式，并仅保留重要的语句结构。

- 保留对象标识符，如数据库和表名。

- 将常量转换为问号（?）。

- 删除注释并格式化空格。

例如，以下两个 SQL 语句在规范化后属于同一类型。

- 规范化前的 SQL 语句

```SQL
SELECT * FROM orders WHERE customer_id=10 AND quantity>20



SELECT * FROM orders WHERE customer_id = 20 AND quantity > 100
```

- 规范化后的 SQL 语句

```SQL
SELECT * FROM orders WHERE customer_id=? AND quantity>?
```
```
# 查询分析

如何优化查询性能是一个经常被问到的问题。慢查询会影响用户体验和集群性能。因此，分析和优化查询性能非常重要。

您可以在 `fe/log/fe.audit.log` 中查看查询信息。每个查询对应一个 `QueryID`，可以用来搜索查询的 `QueryPlan` 和 `Profile`。`QueryPlan` 是由 FE 解析 SQL 语句生成的执行计划。`Profile` 是 BE 的执行结果，包含每个步骤消耗的时间和每个步骤处理的数据量等信息。

## 计划分析

在 StarRocks 中，一个 SQL 语句的生命周期可以分为三个阶段：查询解析、查询规划和查询执行。查询解析通常不是瓶颈，因为分析型工作负载的所需 QPS 不高。

StarRocks 中的查询性能由查询规划和查询执行决定。查询规划负责协调运算符（Join/Order/Aggregate），查询执行负责运行具体的操作。

查询计划为 DBA 提供了宏观的查询信息。查询计划是查询性能的关键，也是 DBA 参考的重要资源。下面的代码片段以 `TPCDS query96` 为例，展示了如何查看查询计划。

```SQL
-- query96.sql
select  count(*)
from store_sales
    ,household_demographics
    ,time_dim
    , store
where ss_sold_time_sk = time_dim.t_time_sk
    and ss_hdemo_sk = household_demographics.hd_demo_sk
    and ss_store_sk = s_store_sk
    and time_dim.t_hour = 8
    and time_dim.t_minute >= 30
    and household_demographics.hd_dep_count = 5
    and store.s_store_name = 'ese'
order by count(*) limit 100;
```

查询计划分为两种类型：逻辑查询计划和物理查询计划。这里描述的查询计划指的是逻辑查询计划。对应于 `TPCDS query96.sql` 的查询计划如下所示。

```sql
+------------------------------------------------------------------------------+
| Explain String                                                               |
+------------------------------------------------------------------------------+
| PLAN FRAGMENT 0                                                              |
|  OUTPUT EXPRS:<slot 11>                                                      |
|   PARTITION: UNPARTITIONED                                                   |
|   RESULT SINK                                                                |
|   12:MERGING-EXCHANGE                                                        |
|      limit: 100                                                              |
|      tuple ids: 5                                                            |
|                                                                              |
| PLAN FRAGMENT 1                                                              |
|  OUTPUT EXPRS:                                                               |
|   PARTITION: RANDOM                                                          |
|   STREAM DATA SINK                                                           |
|     EXCHANGE ID: 12                                                          |
|     UNPARTITIONED                                                            |
|                                                                              |
|   8:TOP-N                                                                    |
|   |  order by: <slot 11> ASC                                                 |
|   |  offset: 0                                                               |
|   |  limit: 100                                                              |
|   |  tuple ids: 5                                                            |
|   |                                                                          |
|   7:AGGREGATE (update finalize)                                              |
|   |  output: count(*)                                                        |
|   |  group by:                                                               |
|   |  tuple ids: 4                                                            |
|   |                                                                          |
|   6:HASH JOIN                                                                |
|   |  join op: INNER JOIN (BROADCAST)                                         |
|   |  hash predicates:                                                        |
|   |  colocate: false, reason: left hash join node can not do colocate        |
|   |  equal join conjunct: `ss_store_sk` = `s_store_sk`                       |
|   |  tuple ids: 0 2 1 3                                                      |
|   |                                                                          |
|   |----11:EXCHANGE                                                           |
|   |       tuple ids: 3                                                       |
|   |                                                                          |
|   4:HASH JOIN                                                                |
|   |  join op: INNER JOIN (BROADCAST)                                         |
|   |  hash predicates:                                                        |
|   |  colocate: false, reason: left hash join node can not do colocate        |
|   |  equal join conjunct: `ss_hdemo_sk`=`household_demographics`.`hd_demo_sk`|
|   |  tuple ids: 0 2 1                                                        |
|   |                                                                          |
|   |----10:EXCHANGE                                                           |
|   |       tuple ids: 1                                                       |
|   |                                                                          |
|   2:HASH JOIN                                                                |
|   |  join op: INNER JOIN (BROADCAST)                                         |
|   |  hash predicates:                                                        |
|   |  colocate: false, reason: table not in same group                        |
|   |  equal join conjunct: `ss_sold_time_sk` = `time_dim`.`t_time_sk`         |
|   |  tuple ids: 0 2                                                          |
|   |                                                                          |
|   |----9:EXCHANGE                                                            |
|   |       tuple ids: 2                                                       |
|   |                                                                          |
|   0:OlapScanNode                                                             |
|      TABLE: store_sales                                                      |
|      PREAGGREGATION: OFF. Reason: `ss_sold_time_sk` is value column          |
|      partitions=1/1                                                          |
|      rollup: store_sales                                                     |
|      tabletRatio=0/0                                                         |
|      tabletList=                                                             |
|      cardinality=-1                                                          |
|      avgRowSize=0.0                                                          |
|      numNodes=0                                                              |
|      tuple ids: 0                                                            |
|                                                                              |
| PLAN FRAGMENT 2                                                              |
|  OUTPUT EXPRS:                                                               |
|   PARTITION: RANDOM                                                          |
|                                                                              |
|   STREAM DATA SINK                                                           |
|     EXCHANGE ID: 11                                                          |
|     UNPARTITIONED                                                            |
|                                                                              |
|   5:OlapScanNode                                                             |
|      TABLE: store                                                            |
|      PREAGGREGATION: OFF. Reason: null                                       |
|      PREDICATES: `store`.`s_store_name` = 'ese'                              |
|      partitions=1/1                                                          |
|      rollup: store                                                           |
|      tabletRatio=0/0                                                         |
|      tabletList=                                                             |
|      cardinality=-1                                                          |
|      avgRowSize=0.0                                                          |
|      numNodes=0                                                              |
|      tuple ids: 3                                                            |
|                                                                              |
| PLAN FRAGMENT 3                                                              |
|  OUTPUT EXPRS:                                                               |
|   PARTITION: RANDOM                                                          |
|   STREAM DATA SINK                                                           |
|     EXCHANGE ID: 10                                                          |
|     UNPARTITIONED                                                            |
|                                                                              |
|   3:OlapScanNode                                                             |
|      TABLE: household_demographics                                           |
|      PREAGGREGATION: OFF. Reason: null                                       |
|      PREDICATES: `household_demographics`.`hd_dep_count` = 5                 |
|      partitions=1/1                                                          |
|      rollup: household_demographics                                          |
|      tabletRatio=0/0                                                         |
|      tabletList=                                                             |
|      cardinality=-1                                                          |
|      avgRowSize=0.0                                                          |
|      numNodes=0                                                              |
|      tuple ids: 1                                                            |
|                                                                              |
| PLAN FRAGMENT 4                                                              |
|  OUTPUT EXPRS:                                                               |
|   PARTITION: RANDOM                                                          |
|   STREAM DATA SINK                                                           |
|     EXCHANGE ID: 09                                                          |
|     UNPARTITIONED                                                            |
|                                                                              |
|   1:OlapScanNode                                                             |
|      TABLE: time_dim                                                         |
|      PREAGGREGATION: OFF. Reason: null                                       |
|      PREDICATES: `time_dim`.`t_hour` = 8, `time_dim`.`t_minute` >= 30        |
|      partitions=1/1                                                          |
|      rollup: time_dim                                                        |
|      tabletRatio=0/0                                                         |
|      tabletList=                                                             |
|      cardinality=-1                                                          |
|      avgRowSize=0.0                                                          |
|      numNodes=0                                                              |
|      tuple ids: 2                                                            |
+------------------------------------------------------------------------------+
128 rows in set (0.02 sec)
```

查询 96 展示了一个涉及多个 StarRocks 概念的查询计划。

| 名称 | 解释 |
|---|---|
| avgRowSize | 扫描数据行的平均大小 |
| cardinality | 扫描表中的数据行总数 |
| colocate | 表是否处于 colocate 模式 |
| numNodes | 要扫描的节点数 |
| rollup | 材料化视图 |
| preaggregation | 预聚合 |
| predicates | 谓词，即查询过滤条件 |

查询 96 的查询计划分为五个片段，编号从 0 到 4。可以逐个自下而上地阅读查询计划。

片段 4 负责扫描 `time_dim` 表，并提前执行相关的查询条件（即 `time_dim.t_hour = 8 and time_dim.t_minute >= 30`）。这一步也称为谓词下推。StarRocks 决定是否为聚合表启用 `PREAGGREGATION`。在上图中，`time_dim` 的预聚合被禁用。在这种情况下，将读取 `time_dim` 的所有维度列，如果表中有许多维度列，可能会对性能产生负面影响。如果 `time_dim` 表选择 `range partition` 进行数据划分，查询计划中将命中多个分区，并自动过滤掉无关的分区。如果存在材料化视图，StarRocks 将根据查询自动选择材料化视图。如果没有材料化视图，查询将自动命中基表（例如，上图中的 `rollup: time_dim`）。

扫描完成后，片段 4 结束。数据将传递给其他片段，如上图中的 EXCHANGE ID : 09 所示，传递给标记为 9 的接收节点。

对于查询 96 的查询计划，片段 2、3 和 4 具有类似的功能，但负责扫描不同的表。具体而言，查询中的 `Order/Aggregation/Join` 操作在片段 1 中执行。

片段 1 使用 `BROADCAST` 方法执行 `Order/Aggregation/Join` 操作，即将小表广播到大表。如果两个表都很大，建议使用 `SHUFFLE` 方法。目前，StarRocks 仅支持 `HASH JOIN`。`colocate` 字段用于显示两个连接表以相同的方式进行分区和分桶，以便可以在本地执行连接操作，而无需迁移数据。完成 Join 操作后，将执行上层的 `aggregation`、`order by` 和 `top-n` 操作。

通过删除具体的表达式（仅保留运算符），可以以更宏观的方式呈现查询计划，如下图所示。

![8-5](../assets/8-5.png)

## 查询提示

查询提示是指令或注释，明确建议查询优化器如何执行查询。目前，StarRocks 支持两种类型的提示：变量设置提示和连接提示。提示仅在单个查询中生效。

### 变量设置提示

您可以使用 `SET_VAR` 提示以 `/*+ SET_VAR(var_name = value) */` 的语法形式在 SELECT 和 SUBMIT TASK 语句中设置一个或多个 [系统变量](../reference/System_variable.md)，或者在其他语句中包含的 SELECT 子句中设置。例如 CREATE MATERIALIZED VIEW AS SELECT 和 CREATE VIEW AS SELECT。

#### 语法

```SQL
[...] SELECT [/*+ SET_VAR(key=value [, key = value]*) */] ...
SUBMIT [/*+ SET_VAR(key=value [, key = value]*) */] TASK ...
```

#### 示例

通过设置系统变量 `streaming_preaggregation_mode` 和 `new_planner_agg_stage`，使用 `SET_VAR` 提示为聚合查询提示聚合方法。

```SQL
SELECT /*+ SET_VAR (streaming_preaggregation_mode = 'force_streaming',new_planner_agg_stage = '2') */ SUM(sales_amount) AS total_sales_amount FROM sales_orders;
```

在 SUBMIT TASK 语句中使用系统变量 `query_timeout`，为查询的任务执行超时时间设置提示。

```SQL
SUBMIT /*+ SET_VAR(query_timeout=3) */ TASK AS CREATE TABLE temp AS SELECT count(*) AS cnt FROM tbl1;
```

在创建材料化视图时，在 SELECT 子句中使用系统变量 `query_timeout`，为查询的执行超时时间设置提示。

```SQL
CREATE MATERIALIZED VIEW mv 
PARTITION BY dt 
DISTRIBUTED BY HASH(`key`) 
BUCKETS 10 
REFRESH ASYNC 
AS SELECT /*+ SET_VAR(query_timeout=500) */ * from dual;
```

### 连接提示

对于多表连接查询，优化器通常会选择最优的连接执行方法。在特殊情况下，您可以使用连接提示来明确建议优化器的连接执行方法或禁用连接重排序。目前，连接提示支持建议 Shuffle Join、Broadcast Join、Bucket Shuffle Join 或 Colocate Join 作为连接执行方法。使用连接提示时，优化器不执行连接重排序。因此，您需要将较小的表选择为右表。此外，当将 [Colocate Join](../using_starrocks/Colocate_join.md) 或 Bucket Shuffle Join 作为连接执行方法时，请确保连接表的数据分布满足这些连接执行方法的要求。否则，建议的连接执行方法将无法生效。

#### 语法

```SQL
... JOIN { [BROADCAST] | [SHUFFLE] | [BUCKET] | [COLOCATE] | [UNREORDER]} ...
```

> **注意**
> 连接提示不区分大小写。

#### 示例

- Shuffle Join

  如果需要在执行 Join 操作之前将具有相同分桶键值的表 A 和表 B 的数据行洗牌到同一台机器上，可以将连接执行方法提示为 Shuffle Join。

  ```SQL
  select k1 from t1 join [SHUFFLE] t2 on t1.k1 = t2.k2 group by t2.k2;
  ```

- Broadcast Join

  如果表 A 是一个大表，而表 B 是一个小表，可以将连接执行方法提示为 Broadcast Join。表 B 的数据将完全广播到表 A 的数据所在的机器上，然后执行 Join 操作。与 Shuffle Join 相比，Broadcast Join 可以节省洗牌表 A 数据的成本。

  ```SQL
  select k1 from t1 join [BROADCAST] t2 on t1.k1 = t2.k2 group by t2.k2;
  ```

- Bucket Shuffle Join

  如果连接查询的 Join 等值连接表达式包含表 A 的分桶键，特别是当表 A 和表 B 都是大表时，可以将连接执行方法提示为 Bucket Shuffle Join。根据表 A 的数据分布，将表 B 的数据根据表 A 的数据分布洗牌到表 A 的数据所在的机器上，然后执行 Join 操作。与 Broadcast Join 相比，Bucket Shuffle Join 显著减少了数据传输，因为表 B 的数据只需全局洗牌一次。

  ```SQL
  select k1 from t1 join [BUCKET] t2 on t1.k1 = t2.k2 group by t2.k2;
  ```

- Colocate Join

  如果表 A 和表 B 属于同一个指定的 Colocation Group（在表创建时指定），具有相同分桶键值的数据行分布在同一个 BE 节点上。当连接查询中的 Join 等值连接表达式包含表 A 和表 B 的分桶键时，可以将连接执行方法提示为 Colocate Join。具有相同键值的数据直接在本地进行连接，减少了节点间数据传输的时间，提高了查询性能。

  ```SQL
  select k1 from t1 join [COLOCATE] t2 on t1.k1 = t2.k2 group by t2.k2;
  ```

### 查看连接执行方法

使用 `EXPLAIN` 命令查看实际的连接执行方法。如果返回的结果显示连接执行方法与连接提示相匹配，则表示连接提示生效。

```SQL
EXPLAIN select k1 from t1 join [COLOCATE] t2 on t1.k1 = t2.k2 group by t2.k2;
```

![8-9](../assets/8-9.png)

## SQL 指纹

SQL 指纹用于优化慢查询并提高系统资源利用率。StarRocks 使用 SQL 指纹功能将慢查询日志（`fe.audit.log.slow_query`）中的 SQL 语句进行规范化，将 SQL 语句分类为不同的类型，并计算每个 SQL 类型的 MD5 哈希值以识别慢查询。MD5 哈希值由字段 `Digest` 指定。

```SQL
2021-12-27 15:13:39,108 [slow_query] |Client=172.26.xx.xxx:54956|User=root|Db=default_cluster:test|State=EOF|Time=2469|ScanBytes=0|ScanRows=0|ReturnRows=6|StmtId=3|QueryId=824d8dc0-66e4-11ec-9fdc-00163e04d4c2|IsQuery=true|feIp=172.26.92.195|Stmt=select count(*) from test_basic group by id_bigint|Digest=51390da6b57461f571f0712d527320f4
```

SQL 语句规范化将语句文本转换为更加规范化的格式，并仅保留重要的语句结构。

- 保留对象标识符，例如数据库和表名称。

- 将常量转换为问号 (?)。

- 删除注释并格式化空格。

例如，下面两条 SQL 语句规范化后属于同一类型。

- 规范化之前的 SQL 语句

```SQL
SELECT * FROM orders WHERE customer_id=10 AND quantity>20



SELECT * FROM orders WHERE customer_id = 20 AND quantity > 100
```

- 规范化后的 SQL 语句

```SQL
SELECT * FROM orders WHERE customer_id=? AND quantity>?