---
displayed_sidebar: "Chinese"
---

# 查询分析

如何优化查询性能是一个经常被问到的问题。慢查询会影响用户体验以及集群性能。分析和优化查询性能非常重要。

您可以在`fe/log/fe.audit.log`中查看查询信息。每个查询都对应一个`QueryID`，可以用来搜索查询的`QueryPlan`和`Profile`。`QueryPlan`是FE解析SQL语句生成的执行计划。`Profile`是BE的执行结果，包含每个步骤消耗的时间以及每个步骤处理的数据量等信息。

## 计划分析

在StarRocks中，SQL语句的生命周期可以分为三个阶段：查询解析，查询规划和查询执行。查询解析通常不是瓶颈，因为分析工作负载的所需QPS并不高。

在StarRocks中，查询性能取决于查询规划和查询执行。查询规划负责协调操作符（Join/Order/Aggregate），而查询执行负责运行特定操作。

查询计划为DBA提供了宏观视角来访问查询信息。查询计划是查询性能的关键，也是DBA参考的好资源。下面的代码片段以`TPCDS query96`为例，展示了如何查看查询计划。

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

有两种类型的查询计划 – 逻辑查询计划和物理查询计划。这里描述的查询计划是指逻辑查询计划。对应于`TPCDS query96.sql`的查询计划如下所示。

~~~sql
+------------------------------------------------------------------------------+
| 执行计划详情                                                                   |
+------------------------------------------------------------------------------+
| 计划分片 0                                                                   |
|  输出表达式:<slot 11>                                                       |
|   分区: 无                                                                   |
|   结果输出节点                                                               |
|   12:合并-交换                                                               |
|      limit: 100                                                             |
|      tuple ids: 5                                                           |
|                                                                              |
| 计划分片 1                                                                  |
|  输出表达式:                                                               |
|   分区: 随机                                                                |
|   流数据输出节点                                                            |
|     交换标识: 12                                                           |
|     无分区                                                                   |
|                                                                              |
|   8:最大N                                                                   |
|   |  排序字段: <slot 11> ASC                                               |
|   |  偏移量: 0                                                              |
|   |  limit: 100                                                             |
|   |  tuple ids: 5                                                           |
|                                                                              |
|   7:聚合（更新完成）                                                        |
|   |  输出字段: count(*)                                                     |
|   |  分组字段:                                                              |
|   |  tuple ids: 4                                                           |
|                                                                              |
|   6:哈希连接                                                               |
|   |  连接操作: 内连接 （广播）                                             |
|   |  哈希谓词:                                                             |
|   |  同位标识: false, 原因: 左哈希连接节点无法同位                          |
|   |  等值连接的前提: `ss_store_sk` = `s_store_sk`                          |
|   |  tuple ids: 0 2 1 3                                                     |
|                                                                              |
|   |----11:交换                                                              |
|   |       tuple ids: 3                                                      |
|   |                                                                         |
|   4:哈希连接                                                               |
|   |  连接操作: 内连接 （广播）                                             |
|   |  哈希谓词:                                                             |
|   |  同位标识: false, 原因: 左哈希连接节点无法同位                          |
|   |  等值连接的前提: `ss_hdemo_sk`=`household_demographics`.`hd_demo_sk`   |
|   |  tuple ids: 0 2 1                                                       |
|   |                                                                         |
|   |----10:交换                                                              |
|   |       tuple ids: 1                                                      |
|   |                                                                         |
|   2:哈希连接                                                               |
|   |  连接操作: 内连接 （广播）                                             |
|   |  哈希谓词:                                                             |
|   |  同位标识: false, 原因: 表不在同一组                                   |
|   |  等值连接的前提: `ss_sold_time_sk` = `time_dim`.`t_time_sk`            |
|   |  tuple ids: 0 2                                                         |
|   |                                                                         |
|   |----9:交换                                                               |
|   |       tuple ids: 2                                                      |
|   |                                                                         |
|   0:Olap扫描节点                                                          |
|      表: store_sales                                                        |
|      预聚合: 关闭。原因: `ss_sold_time_sk` 是值列                           |
|      partitions=1/1                                                         |
|      滚动: store_sales                                                     |
|      自适应比例=0/0                                                         |
|      tablet 列表=                                                            |
|      基数=-1                                                               |
|      平均行大小=0.0                                                         |
|      numNodes=0                                                            |
|      tuple ids: 0                                                         |
|                                                                             |
| 计划分片 2                                                                |
|  输出表达式:                                                               |
|   分区: 随机                                                               |
|                                                                             |
|   流数据输出节点                                                           |
|     交换标识: 11                                                          |
|     无分区                                                                 |
|                                                                             |
|   5:Olap扫描节点                                                          |
|      表: store                                                           |
|      预聚合: 关闭。原因: 空                                              |
|      谓词: `store`.`s_store_name` = 'ese'                                |
|      partitions=1/1                                                       |
|      滚动: store                                                         |
|      自适应比例=0/0                                                       |
|      tablet 列表=                                                          |
|      基数=-1                                                             |
|      平均行大小=0.0                                                       |
|      numNodes=0                                                          |
|      tuple ids: 3                                                        |
|                                                                            |
| 计划分片 3                                                               |
|  输出表达式:                                                              |
|   分区: 随机                                                             |
|   流数据输出节点                                                         |
|     交换标识: 10                                                        |
|     无分区                                                                |
|                                                                            |
|   3:Olap扫描节点                                                         |
|      表: household_demographics                                         |
|      预聚合: 关闭。原因: 空                                           |
|      谓词: `household_demographics`.`hd_dep_count` = 5                |
|      partitions=1/1                                                    |
|      滚动: household_demographics                                    |
|      自适应比例=0/0                                                    |
|      tablet 列表=                                                       |
|      基数=-1                                                          |
|      平均行大小=0.0                                                    |
|      numNodes=0                                                       |
|      tuple ids: 1                                                     |
|                                                                           |
| 计划分片 4                                                              |
|  输出表达式:                                                             |
|   分区: 随机                                                            |
|   流数据输出节点                                                        |
|     交换标识: 09                                                       |
|     无分区                                                               |
|                                                                           |
|   1:Olap扫描节点                                                       |
|      表: time_dim                                                     |
|      预聚合: 关闭。原因: 空                                        |
|      谓词: `time_dim`.`t_hour` = 8, `time_dim`.`t_minute` >= 30     |
|      partitions=1/1                                                 |
|      滚动: time_dim                                                |
|      自适应比例=0/0                                                 |
|      tablet 列表=                                                    |
|      基数=-1                                                       |
|      平均行大小=0.0                                                 |
|      numNodes=0                                                    |
|      tuple ids: 2                                                  |
+-----------------------------------------------------------------------------+
128 行结果 (0.02 秒)
~~~

查询96显示了涉及几个StarRocks概念的查询计划。

|名称|解释|
|--|--|
|平均行大小|扫描数据行的平均大小|
|基数|扫描表中的数据行总数|
|同位|表是否处于同位模式|
|numNodes|要扫描的节点数|
|滚动|物化视图|
|预聚合|预聚合|
|谓词|谓词，即查询过滤条件|

查询96的查询计划分为五个分片，编号从0到4。可以逐个从下到上阅读查询计划。

Fragment 4 is responsible for scanning the `time_dim` table and executing the related query condition (i.e. `time_dim.t_hour = 8 and time_dim.t_minute >= 30`) in advance. This step is also known as predicate pushdown. StarRocks decides whether to enable `PREAGGREGATION` for aggregation tables. In the previous figure, preaggregation of `time_dim` is disabled. In this case, all dimension columns of `time_dim` are read, which may negatively affect performance if there are many dimension columns in the table. If the `time_dim` table selects `range partition` for data division, several partitions will be hit in the query plan and irrelevant partitions will be automatically filtered out. If there is a materialized view, StarRocks will automatically select the materialized view based on the query. If there is no materialized view, the query will automatically hit the base table (for example, `rollup: time_dim` in the previous figure).

When the scan is complete, Fragment 4 ends. Data will be passed to other fragments, as indicated by EXCHANGE ID : 09 in the previous figure, to the receiving node labeled 9.

For the query plan of Query 96, Fragment 2, 3, and 4 have similar functions but they are responsible for scanning different tables. Specifically, the `Order/Aggregation/Join` operations in the query are performed in Fragment 1.

Fragment 1 uses the `BROADCAST` method to perform `Order/Aggregation/Join` operations i, that is, to broadcast the small table to the large table. If both tables are large, we recommend that you use the `SHUFFLE` method. Currently, StarRocks only supports `HASH JOIN`. The `colocate` field is used to show that the two joined tables are partitioned and bucketed in the same way, so that the join operation can be performed locally without migrating the data. When the Join operation is complete, the upper-level `aggregation`, `order by`, and `top-n` operations will be performed.

By removing the specific expressions (only keep the operators), the query plan can be presented in a more macroscopic view, as shown in the following figure.

![8-5](../assets/8-5.png)

## 查询提示

查询提示是明确建议查询优化器如何执行查询的指令或注释。目前，StarRocks支持两种类型的提示：变量设置提示和连接提示。提示只在单个查询中生效。

### 变量设置提示

您可以通过在SELECT和SUBMIT TASK语句中使用形式为语法`/*+ SET_VAR(var_name = value) */`的`SET_VAR`提示来设置一个或多个[系统变量](../reference/System_variable.md)，或者在其他语句中包括的SELECT子句中使用`SET_VAR`提示，例如创建MATERIALIZED VIEW AS SELECT和CREATE VIEW AS SELECT。

#### 语法

~~~SQL
[...] SELECT [/*+ SET_VAR(key=value [, key = value]*) */] ...
SUBMIT [/*+ SET_VAR(key=value [, key = value]*) */] TASK ...
~~~

#### 示例

通过设置系统变量`streaming_preaggregation_mode`和`new_planner_agg_stage`来提示聚合查询的聚合方法。

~~~SQL
SELECT /*+ SET_VAR (streaming_preaggregation_mode = 'force_streaming',new_planner_agg_stage = '2') */ SUM(sales_amount) AS total_sales_amount FROM sales_orders;
~~~

通过在SUBMIT TASK语句中设置系统变量`query_timeout`来提示查询的任务执行超时时间。

~~~SQL
SUBMIT /*+ SET_VAR(query_timeout=3) */ TASK AS CREATE TABLE temp AS SELECT count(*) AS cnt FROM tbl1;
~~~

通过在创建物化视图时设置系统变量`query_timeout`来提示查询的执行超时时间。

~~~SQL
CREATE MATERIALIZED VIEW mv 
PARTITION BY dt 
DISTRIBUTED BY HASH(`key`) 
BUCKETS 10 
REFRESH ASYNC 
AS SELECT /*+ SET_VAR(query_timeout=500) */ * from dual;
~~~

### 连接提示

对于多表连接查询，优化器通常会选择最佳的连接执行方法。在特殊情况下，可以使用连接提示来明确建议优化器连接执行方法或禁用连接重排序。目前，连接提示支持建议Shuffle Join、Broadcast Join、Bucket Shuffle Join或Colocate Join作为连接执行方法。当使用连接提示时，优化器不会执行连接重排序。因此，您需要将较小的表选择为右表。此外，在建议[Colocate Join](../using_starrocks/Colocate_join.md)或Bucket Shuffle Join作为连接执行方法时，请确保连接表的数据分布符合这些连接执行方法的要求。否则，建议的连接执行方法将无法生效。

#### 语法

~~~SQL
... JOIN { [BROADCAST] | [SHUFFLE] | [BUCKET] | [COLOCATE] | [UNREORDER]} ...
~~~

> **注意**
>
> 连接提示不区分大小写。

#### 示例

- Shuffle Join

  如果您需要在执行Join操作之前从表A和B中来自相同bucketing key值的数据行上随机分配数据行，则可以将连接执行方法提示为Shuffle Join。

  ~~~SQL
  select k1 from t1 join [SHUFFLE] t2 on t1.k1 = t2.k2 group by t2.k2;
  ~~~

- Broadcast Join

  如果表A是一个大表，表B是一个小表，可以将连接执行方法提示为Broadcast Join。表B的数据完全广播到包含表A数据的机器上，然后执行Join操作。与Shuffle Join相比，Broadcast Join节省了表A数据洗牌的成本。

  ~~~SQL
  select k1 from t1 join [BROADCAST] t2 on t1.k1 = t2.k2 group by t2.k2;
  ~~~

- Bucket Shuffle Join

  如果连接查询的Join等值Join表达式包含表A的bucketing key，尤其是当表A和B都是大表时，可以将连接执行方法提示为Bucket Shuffle Join。表B的数据根据表A的数据分布根据数据分布洗牌到包含表A数据的机器上，然后执行Join操作。与Broadcast Join相比，Bucket Shuffle Join只需要全局洗牌表B数据一次，因此显著减少了数据传输。

  ~~~SQL
  select k1 from t1 join [BUCKET] t2 on t1.k1 = t2.k2 group by t2.k2;
  ~~~

- Colocate Join

  如果表A和B属于同一个指定的Colocation Group，该Colocation Group在表创建时指定，则可以将连接执行方法提示为Colocate Join。具有相同键值的数据行直接在本地连接，减少了节点间数据传输的时间，提高了查询性能。

  ~~~SQL
  select k1 from t1 join [COLOCATE] t2 on t1.k1 = t2.k2 group by t2.k2;
  ~~~

### 查看连接执行方法

使用`EXPLAIN`命令查看实际的连接执行方法。如果返回的结果显示连接执行方法与连接提示匹配，则表示连接提示生效。

~~~SQL
EXPLAIN select k1 from t1 join [COLOCATE] t2 on t1.k1 = t2.k2 group by t2.k2;
~~~

![8-9](../assets/8-9.png)

## SQL指纹

SQL指纹用于优化慢查询并改善系统资源利用率。StarRocks使用SQL指纹功能将慢查询日志(`fe.audit.log.slow_query`)中的SQL语句标准化，将SQL语句分类为不同类型，并计算每种SQL类型的MD5哈希值以识别慢查询。MD5哈希值由`Digest`字段指定。

~~~SQL
2021-12-27 15:13:39,108 [slow_query] |Client=172.26.xx.xxx:54956|User=root|Db=default_cluster:test|State=EOF|Time=2469|ScanBytes=0|ScanRows=0|ReturnRows=6|StmtId=3|QueryId=824d8dc0-66e4-11ec-9fdc-00163e04d4c2|IsQuery=true|feIp=172.26.92.195|Stmt=select count(*) from test_basic group by id_bigint|Digest=51390da6b57461f571f0712d527320f4
~~~

SQL语句标准化将语句文本转换为更加标准化的格式，并仅保留重要的语句结构。

- 保留对象标识，例如数据库和表名。 

- 将常量转换为问号(?)。

- 删除注释并格式化空格。

例如，在标准化之前的两个SQL语句属于相同类型。
~~~SQL
SELECT * FROM orders WHERE customer_id=10 AND quantity>20
~~~
- 标准化后的SQL语句
~~~SQL
SELECT * FROM orders WHERE customer_id=10 AND quantity>20
~~~