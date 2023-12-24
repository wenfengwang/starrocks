---
displayed_sidebar: English
---

# 查询分析

如何优化查询性能是一个经常被问到的问题。慢查询会影响用户体验和集群性能。分析和优化查询性能非常重要。

您可以在 `fe/log/fe.audit.log` 中查看查询信息。每个查询都对应一个 `QueryID`，可以用来搜索查询的 `QueryPlan` 和 `Profile`。`QueryPlan` 是通过解析 SQL 语句生成的执行计划。`Profile` 是 BE 执行的结果，包含每个步骤消耗的时间和每个步骤处理的数据量等信息。

## 计划分析

在 StarRocks 中，一个 SQL 语句的生命周期可以分为三个阶段：查询解析、查询规划和查询执行。查询解析通常不是瓶颈，因为分析工作负载的 QPS 不高。

StarRocks 的查询性能取决于查询规划和查询执行。查询规划负责协调运算符（Join/Order/Aggregate），查询执行负责运行具体的操作。

查询计划为 DBA 提供了一个宏观的视角来访问查询信息。查询计划是查询性能的关键，也是 DBA 参考的重要资源。下面的代码片段以 `TPCDS query96` 为例，展示了如何查看查询计划。

~~~SQL
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
~~~

查询计划分为逻辑查询计划和物理查询计划两种类型。这里描述的是逻辑查询计划。对应于 `TPCDS query96.sql` 的查询计划如下所示。

~~~sql
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
~~~

查询 96 展示了一个涉及多个 StarRocks 概念的查询计划。

|名称|解释|
|--|--|
|avgRowSize|扫描的数据行的平均大小|
|cardinality|扫描的表中的数据行总数|
|colocate|表是否处于 colocate 模式|
|numNodes|要扫描的节点数|
|rollup|物化视图|
|preaggregation|预聚合|
|predicates|谓词，查询的过滤条件|

查询 96 的查询计划分为五个片段，编号从 0 到 4。可以逐个片段自下而上地阅读查询计划。


片段 4 负责扫描 `time_dim` 表并提前执行相关的查询条件（即 `time_dim.t_hour = 8 and time_dim.t_minute >= 30`）。这一步骤也被称为谓词下推。StarRocks 决定是否为聚合表启用 `PREAGGREGATION`。在上图中，`time_dim` 的预聚合被禁用。在这种情况下，将读取 `time_dim` 的所有维度列，如果表中有许多维度列，可能会对性能产生负面影响。如果 `time_dim` 表选择使用 `range partition` 进行数据划分，则查询计划中将命中多个分区，并自动过滤掉不相关的分区。如果存在物化视图，StarRocks 将根据查询自动选择物化视图。如果没有物化视图，查询将自动命中基表（例如，在上图中的 `rollup: time_dim`）。

扫描完成后，片段 4 结束。数据将传递到其他片段，如上图中的 EXCHANGE ID ： 09 所示，传递到标记为 9 的接收节点。

对于查询 96 的查询计划，片段 2、3 和 4 具有类似的功能，但它们负责扫描不同的表。具体来说，查询中的 `Order/Aggregation/Join` 操作在片段 1 中执行。

片段 1 使用 `BROADCAST` 方法执行 `Order/Aggregation/Join` 操作，即将小表广播到大表。如果两个表都很大，建议使用 `SHUFFLE` 方法。目前，StarRocks 仅支持 `HASH JOIN`。`colocate` 字段用于显示两个连接表的分区和桶化方式相同，因此可以在本地执行连接操作，而无需迁移数据。完成 Join 操作后，将执行上层的 `aggregation`、`order by` 和 `top-n` 操作。

通过删除特定的表达式（仅保留运算符），可以以更宏观的视图呈现查询计划，如下图所示。

![8-5](../assets/8-5.png)

## 查询提示

查询提示是明确建议查询优化器如何执行查询的指令或注释。目前，StarRocks 支持两种类型的提示：变量设置提示和连接提示。提示仅在单个查询中生效。

### 变量设置提示

可以通过在 SELECT 和 SUBMIT TASK 语句中，或者在其他语句（如 CREATE MATERIALIZED VIEW AS SELECT 和 CREATE VIEW AS SELECT）中包含的 SELECT 子句中使用 `SET_VAR` 提示的形式来设置一个或多个 [系统变量](../reference/System_variable.md)。

#### 语法

~~~SQL
[...] SELECT [/*+ SET_VAR(key=value [, key = value]*) */] ...
SUBMIT [/*+ SET_VAR(key=value [, key = value]*) */] TASK ...
~~~

#### 例子

通过设置系统变量 `streaming_preaggregation_mode` 和 `new_planner_agg_stage` 来提示聚合查询的聚合方法。

~~~SQL
SELECT /*+ SET_VAR (streaming_preaggregation_mode = 'force_streaming',new_planner_agg_stage = '2') */ SUM(sales_amount) AS total_sales_amount FROM sales_orders;
~~~

通过在 SUBMIT TASK 语句中设置系统变量 `query_timeout` 来提示查询的任务执行超时期限。

~~~SQL
SUBMIT /*+ SET_VAR(query_timeout=3) */ TASK AS CREATE TABLE temp AS SELECT count(*) AS cnt FROM tbl1;
~~~

在创建物化视图时，通过在 SELECT 子句中设置系统变量 `query_timeout` 来提示查询的执行超时期限。

~~~SQL
CREATE MATERIALIZED VIEW mv 
PARTITION BY dt 
DISTRIBUTED BY HASH(`key`) 
BUCKETS 10 
REFRESH ASYNC 
AS SELECT /*+ SET_VAR(query_timeout=500) */ * from dual;
~~~

### 连接提示

对于多表联接查询，优化器通常会选择最佳的联接执行方法。在特殊情况下，可以使用连接提示向优化器显式建议联接执行方法，或禁用“联接重新排序”。目前，联接提示支持建议将 Shuffle Join、Broadcast Join、Bucket Shuffle Join 或 Colocate Join 作为联接执行方法。使用连接提示时，优化程序不会执行“联接重新排序”。因此，您需要选择较小的表作为正确的表。此外，在建议将 [Colocate Join](../using_starrocks/Colocate_join.md) 或 Bucket Shuffle Join 作为联接执行方法时，请确保联接表的数据分布满足这些联接执行方法的要求。否则，建议的联接执行方法将无法生效。

#### 语法

~~~SQL
... JOIN { [BROADCAST] | [SHUFFLE] | [BUCKET] | [COLOCATE] | [UNREORDER]} ...
~~~

> **注意**
>
> 联接提示不区分大小写。

#### 例子

- Shuffle Join

  如果需要在执行 Join 操作之前将表 A 和表 B 中具有相同存储桶键值的数据行随机分发到同一台计算机上，则可以将联接执行方法提示为 Shuffle Join。

  ~~~SQL
  select k1 from t1 join [SHUFFLE] t2 on t1.k1 = t2.k2 group by t2.k2;
  ~~~

- Broadcast Join
  
  如果表 A 是大表，表 B 是小表，则可以将联接执行方法提示为 Broadcast Join。将表 B 的数据全量广播到表 A 的数据所在的机器上，然后进行 Join 操作。与 Shuffle Join 相比，Broadcast Join 节省了对表 A 数据进行洗牌的成本。

  ~~~SQL
  select k1 from t1 join [BROADCAST] t2 on t1.k1 = t2.k2 group by t2.k2;
  ~~~

- Bucket Shuffle Join
  
  如果联接查询中的 Join equijoin 表达式包含表 A 的分桶键，尤其是当表 A 和 B 都是大表时，可以将联接执行方法提示为 Bucket Shuffle Join。根据表 A 的数据分布，将表 B 的数据洗牌到表 A 的数据所在的机器上，然后进行 Join 操作。与 Broadcast Join 相比，Bucket Shuffle Join 显著减少了数据传输量，因为表 B 的数据全局只洗牌一次。

  ~~~SQL
  select k1 from t1 join [BUCKET] t2 on t1.k1 = t2.k2 group by t2.k2;
  ~~~

- Colocate Join
  
  如果表 A 和表 B 属于创建表时指定的同一 Colocation Group，则表 A 和表 B 中具有相同分桶键值的数据行将分布在同一个 BE 节点上。当联接查询中的 Join equijoin 表达式包含表 A 和表 B 的分桶键时，可以将联接执行方法提示为 Colocate Join。具有相同键值的数据直接在本地联接，减少节点间数据传输时间，提高查询性能。

  ~~~SQL
  select k1 from t1 join [COLOCATE] t2 on t1.k1 = t2.k2 group by t2.k2;
  ~~~

### 查看联接执行方法

使用 `EXPLAIN` 命令可查看实际的联接执行方法。如果返回结果显示联接执行方法与联接提示匹配，则表示联接提示有效。

~~~SQL
EXPLAIN select k1 from t1 join [COLOCATE] t2 on t1.k1 = t2.k2 group by t2.k2;
~~~

![8-9](../assets/8-9.png)

## SQL 指纹

SQL 指纹用于优化慢查询，提高系统资源利用率。StarRocks 使用 SQL 指纹功能对慢查询日志（`fe.audit.log.slow_query`）中的 SQL 语句进行规范化处理，将 SQL 语句分类为不同的类型，并计算每种 SQL 类型的 MD5 哈希值来识别慢查询。MD5 哈希值由字段 `Digest` 指定。

~~~SQL
2021-12-27 15:13:39,108 [慢查询] |客户端=172.26.xx.xxx:54956|用户=root|数据库=default_cluster:test|状态=EOF|时间=2469|扫描字节数=0|扫描行数=0|返回行数=6|StmtId=3|QueryId=824d8dc0-66e4-11ec-9fdc-00163e04d4c2|是否查询=true|feIp=172.26.92.195|语句=select count(*) from test_basic group by id_bigint|摘要=51390da6b57461f571f0712d527320f4
~~~

SQL语句规范化会将语句文本转换为更规范的格式，并仅保留重要的语句结构。

- 保留对象标识符，例如数据库和表名称。

- 将常量转换为问号(?)。

- 删除注释并格式化空格。

例如，以下两条SQL语句在规范化后属于同一类型。

- 规范化前的SQL语句

~~~SQL
SELECT * FROM orders WHERE customer_id=10 AND quantity>20



SELECT * FROM orders WHERE customer_id = 20 AND quantity > 100
~~~

- 规范化后的SQL语句

~~~SQL
SELECT * FROM orders WHERE customer_id=? AND quantity>?
~~~