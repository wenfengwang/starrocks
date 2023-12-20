---
displayed_sidebar: English
---

# 同步物化视图

本主题描述了如何创建、使用和管理**同步物化视图（Rollup）**。

对于同步物化视图，基表中的所有更改都会实时更新到相应的同步物化视图中。同步物化视图的刷新会自动触发。由于同步物化视图的维护和更新成本较低，它们非常适合用于透明地加速实时单表聚合查询。

在StarRocks中，**同步物化视图**只能在**[默认目录](../data_source/catalog/default_catalog.md)**的单个基表上创建。它们本质上是用于查询加速的特殊索引，而不是像**异步物化视图**那样的物理表。

从v2.4版本开始，StarRocks提供了异步物化视图，支持跨多表创建并支持更多的聚合操作。有关**异步物化视图**的使用，请参见[异步物化视图](../using_starrocks/Materialized_view.md)一节。

:::注意目前共享数据集群尚不支持同步物化视图。:::

下表比较了StarRocks v2.5、v2.4中的异步物化视图（ASYNC MVs）和同步物化视图（SYNC MV），从它们支持的功能特性角度出发：

|单表聚合|多表join|查询重写|刷新策略|基表|
|---|---|---|---|---|
|ASYNC MV|是|是|是|异步刷新手动刷新|多个表来自：默认目录外部目录(v2.5)现有物化视图(v2.5)现有视图(v3.1)|
|SYNC MV (Rollup)|聚合函数选择有限|否|是|数据加载时同步刷新|默认目录单表|

## 基本概念

- **基表**

  基表是物化视图的驱动表。

  对于StarRocks的同步物化视图，基表必须是[默认目录](../data_source/catalog/default_catalog.md)中的单个原生表。StarRocks支持在重复键表、聚合表和唯一键表上创建同步物化视图。

- **刷新**

  每当基表中的数据发生变化时，同步物化视图都会自动更新。您无需手动触发刷新。

- **查询改写**

  查询改写指的是在对建立了物化视图的基表执行查询时，系统自动判断物化视图中的预计算结果是否可以重用于查询。如果可以重用，系统将直接从相关的物化视图加载数据，避免进行耗时和资源消耗的计算或联接。

  同步物化视图支持基于某些聚合操作符的查询改写。更多信息，请参见[聚合函数的对应关系](#correspondence-of-aggregate-functions)。

## 准备工作

在创建同步物化视图之前，请检查您的数据仓库是否适合通过同步物化视图进行查询加速。例如，检查查询是否重用了特定的子查询语句。

以下示例基于sales_records表，包含每笔交易的交易ID（record_id）、销售人员ID（seller_id）、商店ID（store_id）、日期（sale_date）和销售金额（sale_amt）。按照以下步骤创建表并插入数据：

```SQL
CREATE TABLE sales_records(
    record_id INT,
    seller_id INT,
    store_id INT,
    sale_date DATE,
    sale_amt BIGINT
) DISTRIBUTED BY HASH(record_id);

INSERT INTO sales_records
VALUES
    (001,01,1,"2022-03-13",8573),
    (002,02,2,"2022-03-14",6948),
    (003,01,1,"2022-03-14",4319),
    (004,03,3,"2022-03-15",8734),
    (005,03,3,"2022-03-16",4212),
    (006,02,2,"2022-03-17",9515);
```

本例的业务场景要求频繁分析不同商店的销售金额。因此，每次查询都会使用sum()函数，消耗大量的计算资源。您可以运行查询来记录其耗时，并使用EXPLAIN命令查看查询计划。

```Plain
MySQL > SELECT store_id, SUM(sale_amt)
FROM sales_records
GROUP BY store_id;
+----------+-----------------+
| store_id | sum(`sale_amt`) |
+----------+-----------------+
|        2 |           16463 |
|        3 |           12946 |
|        1 |           12892 |
+----------+-----------------+
3 rows in set (0.02 sec)

MySQL > EXPLAIN SELECT store_id, SUM(sale_amt)
FROM sales_records
GROUP BY store_id;
+-----------------------------------------------------------------------------+
| Explain String                                                              |
+-----------------------------------------------------------------------------+
| PLAN FRAGMENT 0                                                             |
|  OUTPUT EXPRS:3: store_id | 6: sum                                          |
|   PARTITION: UNPARTITIONED                                                  |
|                                                                             |
|   RESULT SINK                                                               |
|                                                                             |
|   4:EXCHANGE                                                                |
|                                                                             |
| PLAN FRAGMENT 1                                                             |
|  OUTPUT EXPRS:                                                              |
|   PARTITION: HASH_PARTITIONED: 3: store_id                                  |
|                                                                             |
|   STREAM DATA SINK                                                          |
|     EXCHANGE ID: 04                                                         |
|     UNPARTITIONED                                                           |
|                                                                             |
|   3:AGGREGATE (merge finalize)                                              |
|   |  output: sum(6: sum)                                                    |
|   |  group by: 3: store_id                                                  |
|   |                                                                         |
|   2:EXCHANGE                                                                |
|                                                                             |
| PLAN FRAGMENT 2                                                             |
|  OUTPUT EXPRS:                                                              |
|   PARTITION: RANDOM                                                         |
|                                                                             |
|   STREAM DATA SINK                                                          |
|     EXCHANGE ID: 02                                                         |
|     HASH_PARTITIONED: 3: store_id                                           |
|                                                                             |
|   1:AGGREGATE (update serialize)                                            |
|   |  STREAMING                                                              |
|   |  output: sum(5: sale_amt)                                               |
|   |  group by: 3: store_id                                                  |
|   |                                                                         |
|   0:OlapScanNode                                                            |
|      TABLE: sales_records                                                   |
|      PREAGGREGATION: ON                                                     |
|      partitions=1/1                                                         |
|      rollup: sales_records                                                  |
|      tabletRatio=10/10                                                      |
|      tabletList=12049,12053,12057,12061,12065,12069,12073,12077,12081,12085 |
|      cardinality=1                                                          |
|      avgRowSize=2.0                                                         |
|      numNodes=0                                                             |
+-----------------------------------------------------------------------------+
45 rows in set (0.00 sec)
```

可以观察到查询大约耗时0.02秒，并且没有使用同步物化视图来加速查询，因为查询计划中rollup字段的值是sales_records，即基表。

## 创建同步物化视图

您可以使用[CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md)语句来创建基于特定查询语句的同步物化视图。

基于sales_records表和上述查询语句，以下示例创建了名为store_amt的同步物化视图，用于分析每个商店的销售金额总和。

```SQL
CREATE MATERIALIZED VIEW store_amt AS
SELECT store_id, SUM(sale_amt)
FROM sales_records
GROUP BY store_id;
```

> **警告**
- 在同步物化视图中使用聚合函数时，您必须使用GROUP BY子句，并在SELECT列表中至少指定一个GROUP BY列。
- 同步物化视图不支持在多个列上使用单个聚合函数。形如sum(a+b)的查询语句不被支持。
- 同步物化视图不支持在单列上使用多个聚合函数。形如select sum(a), min(a) from table的查询语句不被支持。
- 在创建同步物化视图时不支持使用JOIN。
- 在使用ALTER TABLE DROP COLUMN删除基表中的特定列时，您需要确保所有同步物化视图都不包含该列，否则无法执行删除操作。要删除同步物化视图中使用的列，您需要先删除所有包含该列的同步物化视图，然后才能删除该列。
- 为表创建过多的同步物化视图会影响数据加载效率。当数据加载到基表时，同步物化视图和基表中的数据会同步更新。如果基表包含n个同步物化视图，那么加载数据到基表的效率大约相当于加载数据到n个表中。
- 目前，StarRocks不支持同时创建多个同步物化视图。只有在上一个同步物化视图创建完成后，才能创建新的同步物化视图。
- 共享数据的StarRocks集群不支持同步物化视图。

## 检查同步物化视图的构建状态

创建同步物化视图是一个异步操作。执行CREATE MATERIALIZED VIEW成功意味着提交创建物化视图的任务。您可以通过[SHOW ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/SHOW_ALTER_MATERIALIZED_VIEW.md)命令查看数据库中同步物化视图的构建状态。

```Plain
MySQL > SHOW ALTER MATERIALIZED VIEW\G
*************************** 1. row ***************************
          JobId: 12090
      TableName: sales_records
     CreateTime: 2022-08-25 19:41:10
   FinishedTime: 2022-08-25 19:41:39
  BaseIndexName: sales_records
RollupIndexName: store_amt
       RollupId: 12091
  TransactionId: 10
          State: FINISHED
            Msg: 
       Progress: NULL
        Timeout: 86400
1 row in set (0.00 sec)
```

RollupIndexName部分指示了同步物化视图的名称，State部分显示了构建是否完成。

## 直接查询同步物化视图

由于同步物化视图本质上是基表的索引，而不是实体表，您只能使用提示[_SYNC_MV_]来查询同步物化视图：

```SQL
-- Do not omit the brackets [] in the hint.
MySQL > SELECT * FROM store_amt [_SYNC_MV_];
+----------+----------+
| store_id | sale_amt |
+----------+----------+
|        2 |     6948 |
|        3 |     8734 |
|        1 |     4319 |
|        2 |     9515 |
|        3 |     4212 |
|        1 |     8573 |
+----------+----------+
```

> **警告**
> 目前，StarRocks自动生成列名称，即使您为同步物化视图中的列指定了别名。

## 使用同步物化视图改写和加速查询

您创建的同步物化视图包含与查询语句相符的预计算完整结果集。后续查询将使用这些数据。您可以运行与准备阶段相同的查询，以测试查询时间。

```Plain
MySQL > SELECT store_id, SUM(sale_amt)
FROM sales_records
GROUP BY store_id;
+----------+-----------------+
| store_id | sum(`sale_amt`) |
+----------+-----------------+
|        2 |           16463 |
|        3 |           12946 |
|        1 |           12892 |
+----------+-----------------+
3 rows in set (0.01 sec)
```

可以观察到查询时间缩短至0.01秒。

## 检查查询是否命中同步物化视图

再次执行EXPLAIN命令，以检查查询是否命中了同步物化视图。

```Plain
MySQL > EXPLAIN SELECT store_id, SUM(sale_amt) FROM sales_records GROUP BY store_id;
+-----------------------------------------------------------------------------+
| Explain String                                                              |
+-----------------------------------------------------------------------------+
| PLAN FRAGMENT 0                                                             |
|  OUTPUT EXPRS:3: store_id | 6: sum                                          |
|   PARTITION: UNPARTITIONED                                                  |
|                                                                             |
|   RESULT SINK                                                               |
|                                                                             |
|   4:EXCHANGE                                                                |
|                                                                             |
| PLAN FRAGMENT 1                                                             |
|  OUTPUT EXPRS:                                                              |
|   PARTITION: HASH_PARTITIONED: 3: store_id                                  |
|                                                                             |
|   STREAM DATA SINK                                                          |
|     EXCHANGE ID: 04                                                         |
|     UNPARTITIONED                                                           |
|                                                                             |
|   3:AGGREGATE (merge finalize)                                              |
|   |  output: sum(6: sum)                                                    |
|   |  group by: 3: store_id                                                  |
|   |                                                                         |
|   2:EXCHANGE                                                                |
|                                                                             |
| PLAN FRAGMENT 2                                                             |
|  OUTPUT EXPRS:                                                              |
|   PARTITION: RANDOM                                                         |
|                                                                             |
|   STREAM DATA SINK                                                          |
|     EXCHANGE ID: 02                                                         |
|     HASH_PARTITIONED: 3: store_id                                           |
|                                                                             |
|   1:AGGREGATE (update serialize)                                            |
|   |  STREAMING                                                              |
|   |  output: sum(5: sale_amt)                                               |
|   |  group by: 3: store_id                                                  |
|   |                                                                         |
|   0:OlapScanNode                                                            |
|      TABLE: sales_records                                                   |
|      PREAGGREGATION: ON                                                     |
|      partitions=1/1                                                         |
|      rollup: store_amt                                                      |
|      tabletRatio=10/10                                                      |
|      tabletList=12092,12096,12100,12104,12108,12112,12116,12120,12124,12128 |
|      cardinality=6                                                          |
|      avgRowSize=2.0                                                         |
|      numNodes=0                                                             |
+-----------------------------------------------------------------------------+
45 rows in set (0.00 sec)
```

可以观察到，查询计划中rollup部分的值现在是store_amt，这是您创建的同步物化视图。这意味着该查询已经命中了同步物化视图。

## 展示同步物化视图

您可以执行DESC \<tbl_name\> ALL命令，以检查表及其从属的同步物化视图的架构。

```Plain
MySQL > DESC sales_records ALL;
+---------------+---------------+-----------+--------+------+-------+---------+-------+
| IndexName     | IndexKeysType | Field     | Type   | Null | Key   | Default | Extra |
+---------------+---------------+-----------+--------+------+-------+---------+-------+
| sales_records | DUP_KEYS      | record_id | INT    | Yes  | true  | NULL    |       |
|               |               | seller_id | INT    | Yes  | true  | NULL    |       |
|               |               | store_id  | INT    | Yes  | true  | NULL    |       |
|               |               | sale_date | DATE   | Yes  | false | NULL    | NONE  |
|               |               | sale_amt  | BIGINT | Yes  | false | NULL    | NONE  |
|               |               |           |        |      |       |         |       |
| store_amt     | AGG_KEYS      | store_id  | INT    | Yes  | true  | NULL    |       |
|               |               | sale_amt  | BIGINT | Yes  | false | NULL    | SUM   |
+---------------+---------------+-----------+--------+------+-------+---------+-------+
8 rows in set (0.00 sec)
```

## 删除同步物化视图

在以下情况下，您可能需要删除同步物化视图：

- 您创建了错误的物化视图，并需要在构建完成之前将其删除。
- 您创建了过多的物化视图，导致加载性能大幅下降，且有些物化视图是多余的。
- 涉及的查询频率较低，您可以接受较高的查询延迟。

### 删除未完成的同步物化视图

您可以通过[取消正在进行的创建任务](#check-the-building-status-of-a-synchronous-materialized-view)来删除正在创建中的同步物化视图。首先，您需要通过检查物化视图的构建状态来获取物化视图创建任务的作业ID `JobID`。获取作业ID后，您需要使用CANCEL ALTER命令来取消创建任务。

```Plain
CANCEL ALTER TABLE ROLLUP FROM sales_records (12090);
```

### 删除现有的同步物化视图

您可以使用[DROP MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/DROP_MATERIALIZED_VIEW.md)命令来删除现有的同步物化视图。

```SQL
DROP MATERIALIZED VIEW store_amt;
```

## 最佳实践

### 精确计数去重

以下示例基于广告业务分析表advertiser_view_record，记录了广告被查看的日期（click_time）、广告的名称（advertiser）、广告渠道（channel）以及查看用户的ID（user_id）。

```SQL
CREATE TABLE advertiser_view_record(
    click_time DATE,
    advertiser VARCHAR(10),
    channel VARCHAR(10),
    user_id INT
) distributed BY hash(click_time);
```

分析主要关注广告的独立访客（UV）。

```SQL
SELECT advertiser, channel, count(distinct user_id)
FROM advertiser_view_record
GROUP BY advertiser, channel;
```

为了加速精确的去重计数，您可以基于此表创建同步物化视图，并使用bitmap_union函数预聚合数据。

```SQL
CREATE MATERIALIZED VIEW advertiser_uv AS
SELECT advertiser, channel, bitmap_union(to_bitmap(user_id))
FROM advertiser_view_record
GROUP BY advertiser, channel;
```

创建同步物化视图后，后续查询中的子查询count(distinct user_id)将自动改写为bitmap_union_count(to_bitmap(user_id))，从而可以命中同步物化视图。

### 近似计数去重

再次以`advertiser_view_record`表为例。为了加速近似的去重计数，您可以创建一个同步物化视图，基于这个表，并使用[hll_union()](../sql-reference/sql-functions/aggregate-functions/hll_union.md)函数预聚合数据。

```SQL
CREATE MATERIALIZED VIEW advertiser_uv2 AS
SELECT advertiser, channel, hll_union(hll_hash(user_id))
FROM advertiser_view_record
GROUP BY advertiser, channel;
```

### 设置额外的排序键

假设基表tableA包含列k1、k2和k3，其中k1和k2是排序键。如果包含子查询where k3=x的查询需要加速，您可以创建一个以k3作为第一列的同步物化视图。

```SQL
CREATE MATERIALIZED VIEW k3_as_key AS
SELECT k3, k2, k1
FROM tableA
```

## 聚合函数的对应关系

当使用同步物化视图执行查询时，原始查询语句会自动改写并用于查询同步物化视图中存储的中间结果。下表展示了原始查询中聚合函数与构建同步物化视图时使用的聚合函数之间的对应关系。您可以根据业务场景选择相应的聚合函数来构建同步物化视图。

|原始查询中的聚合函数|物化视图的聚合函数|
|---|---|
|总和|总和|
|分钟|分钟|
|最大|最大|
|计数|计数|
|bitmap_union，bitmap_union_count，计数（不同）|bitmap_union|
|hll_raw_agg、hll_union_agg、ndv、approx_count_distinct|hll_union|
