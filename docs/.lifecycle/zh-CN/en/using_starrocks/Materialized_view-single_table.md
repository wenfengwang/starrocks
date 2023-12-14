---
displayed_sidebar: "Chinese"
---

# 同步物化视图

本主题描述了如何创建、使用和管理**同步物化视图（Rollup）**。

对于同步物化视图，基表中的所有更改都会同时更新到相应的同步物化视图中。同步物化视图的刷新是自动触发的。同步物化视图的维护和更新成本极低，适用于透明加速实时、单表聚合查询。

在StarRocks中，同步物化视图只能在来自[默认目录](../data_source/catalog/default_catalog.md)的单个基表上创建。它们本质上是查询加速的特殊索引，而不是像异步物化视图那样的物理表。

从v2.4开始，StarRocks提供了异步物化视图，支持在多个表上创建，并且支持更多的聚合操作符。有关**异步物化视图**的用法，请参阅[异步物化视图](../using_starrocks/Materialized_view.md)。

以下表格对比了StarRocks v2.5、v2.4中的异步物化视图（ASYNC MVs）和同步物化视图（SYNC MV）在其支持的特性方面的区别：

|                       | **单表聚合** | **多表连接** | **查询重写** | **刷新策略** | **基表** |
| --------------------- | ------------ | ------------ | ------------ | ------------ | -------- |
| **ASYNC MV** | 是 | 是 | 是 | <ul><li>异步刷新</li><li>手动刷新</li></ul> | 来自以下的多个表：<ul><li>默认目录</li><li>外部目录（v2.5）</li><li>现有物化视图（v2.5）</li><li>现有视图（v3.1）</li></ul> |
| **SYNC MV (Rollup)**  | 有限的[聚合函数选择](#聚合函数的对应关系) | 否 | 是 | 数据加载期间同步刷新 | 默认目录中的单个表 |

## 基本概念

- **基表**

  基表是物化视图的驱动表。

  对于StarRocks的同步物化视图，基表必须是来自[默认目录](../data_source/catalog/default_catalog.md)的单个本地表。StarRocks支持在重复键表、聚合表和唯一键表上创建同步物化视图。

- **刷新**

  同步物化视图在基表数据变化时会自动更新自身，无需手动触发刷新。

- **查询重写**

  查询重写意味着在对建立了物化视图的基表执行查询时，系统会自动判断物化视图中的预先计算结果是否可被重用于查询。如果可以重用，系统将直接从相关物化视图加载数据，避免耗费时间和资源的计算或连接。

  同步物化视图支持基于一些聚合操作符的查询重写。更多信息，请参阅[聚合函数的对应关系](#聚合函数的对应关系)。

## 准备工作

在创建同步物化视图之前，检查您的数据仓库是否符合通过同步物化视图进行查询加速的条件。例如，检查查询中是否重用了某些子查询语句。

以下示例基于包含每个交易的交易ID `record_id`、销售员ID `seller_id`、商店ID `store_id`、日期 `sale_date` 和销售金额 `sale_amt` 的表 `sales_records`。按照以下步骤创建表并向其插入数据：

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

该示例的业务场景要求经常分析不同商店的销售金额。因此，每个查询中都使用了`sum()`函数，消耗了大量计算资源。您可以运行查询来记录其时间，并通过使用EXPLAIN命令查看其查询概要。

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

可以观察到查询耗时约为0.02秒，并且未使用同步物化视图加速查询，因为查询概要中`rollup`字段的值为`sales_records`，即为基表。

## 创建同步物化视图

您可以使用[CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md)基于特定查询语句创建同步物化视图。

基于表`sales_records`和上述查询语句，以下示例创建了同步物化视图`store_amt`，用于分析每个商店的销售金额总和。

```SQL
CREATE MATERIALIZED VIEW store_amt AS
SELECT store_id, SUM(sale_amt)
FROM sales_records
GROUP BY store_id;
```

> **注意**
>
> - 在同步物化视图中使用聚合函数时，必须使用GROUP BY子句并在SELECT列表中至少指定一个GROUP BY列。
> - 同步物化视图不支持在多个列上使用一个聚合函数，不支持形如`sum(a+b)`的查询语句形式。
> - 同步物化视图不支持在一个列上使用多个聚合函数，不支持形如`select sum(a), min(a) from table`的查询语句形式。
> - 创建同步物化视图时不支持JOIN操作。
> - 使用ALTER TABLE DROP COLUMN在基表中删除特定列时，需要确保所有基表的同步物化视图不包含已删除列，否则无法执行删除操作。要删除使用了同步物化视图的列，需要先删除所有包含该列的同步物化视图，然后再删除列。
> - 为表创建过多的同步物化视图将影响数据加载效率。当数据加载到基表时，同步物化视图和基表的数据会同步更新。如果基表包含`n`个同步物化视图，那么加载数据到基表的效率与加载数据到`n`个表的效率大致相同。
- 当前，StarRocks 不支持同时创建多个同步物化视图。只有上一个同步物化视图完成后，才能创建新的同步物化视图。
- 共享数据的 StarRocks 集群不支持同步物化视图。

## 检查同步物化视图的构建状态

创建同步物化视图是一个异步操作。成功执行 CREATE MATERIALIZED VIEW 表示成功提交创建物化视图的任务。您可以通过在数据库中显示 [SHOW ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/SHOW_ALTER_MATERIALIZED_VIEW.md) 命令来查看同步物化视图的构建状态。

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

`RollupIndexName` 部分指示同步物化视图的名称，`State` 部分指示构建是否已完成。

## 直接查询同步物化视图

由于同步物化视图本质上是基表的索引而不是物理表，您只能使用提示 `[_SYNC_MV_]` 查询同步物化视图：

```SQL
-- 别忘了在提示中使用方括号[]。
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

> **注意**
>
> 当前，即使为同步物化视图列指定了别名，StarRocks 也会自动生成列名。

## 重写和加速使用同步物化视图的查询

您创建的同步物化视图包含根据查询语句预先计算的完整结果集。后续查询使用其中的数据。您可以运行与准备阶段相同的查询来测试查询时间。

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

可以观察到查询时间减少到 0.01 秒。

## 检查查询是否命中同步物化视图

再次执行 EXPLAIN 命令，查看查询是否命中同步物化视图。

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

可以观察到查询概要中的 `rollup` 部分的值现在是 `store_amt`，这是您建立的同步物化视图。这意味着此查询已命中同步物化视图。

## 显示同步物化视图

您可以执行 DESC <tbl_name> ALL 命令来检查表及其从属同步物化视图的架构。

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

在以下情况下，您需要删除同步物化视图：

- 您已经创建了错误的物化视图，并且需要在构建完成之前删除它。
- 您创建了过多的物化视图，导致负载性能急剧下降，并且其中一些物化视图是重复的。
- 相关查询的频率较低，且您可以容忍相对较高的查询延迟。

### 删除未完成的同步物化视图

您可以取消正在创建的同步物化视图，以删除正在创建的同步物化视图。首先，您需要通过[检查同步物化视图的构建状态](#检查同步物化视图的构建状态)来获取物化视图创建任务的任务 ID `JobID`。获取任务 ID 后，您需要使用 CANCEL ALTER 命令取消创建任务。

```Plain
CANCEL ALTER TABLE ROLLUP FROM sales_records (12090);
```

### 删除现有的同步物化视图

您可以使用 [DROP MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/DROP_MATERIALIZED_VIEW.md) 命令来删除现有的同步物化视图。

```SQL
DROP MATERIALIZED VIEW store_amt;
```

## 最佳实践

### 精确计数

以下示例基于一个广告业务分析表 `advertiser_view_record`，记录了广告被查看的日期 `click_time`、广告名称 `advertiser`、广告渠道 `channel` 和查看广告的用户 ID `user_id`。

```SQL
CREATE TABLE advertiser_view_record(
    click_time DATE,
    advertiser VARCHAR(10),
    channel VARCHAR(10),
    user_id INT
) distributed BY hash(click_time);
```

主要关注广告的 UV。

```SQL
SELECT advertiser, channel, count(distinct user_id)
FROM advertiser_view_record
GROUP BY advertiser, channel;
```

为加速精确计数，您可以根据此表创建一个同步物化视图，并使用 bitmap_union 函数预聚合数据。

```SQL
CREATE MATERIALIZED VIEW advertiser_uv AS
SELECT advertiser, channel, bitmap_union(to_bitmap(user_id))
FROM advertiser_view_record
按广告商、渠道进行分组;

```

创建同步物化视图后，后续查询中的子查询 `count(distinct user_id)` 将自动重写为 `bitmap_union_count (to_bitmap(user_id))`，以便命中同步物化视图。

### 近似计数唯一

再次以上述的 `advertiser_view_record` 表为例。为了加速近似计数唯一，您可以基于该表创建一个同步物化视图，并使用 [hll_union()](../sql-reference/sql-functions/aggregate-functions/hll_union.md) 函数对数据进行预聚合。

```SQL
CREATE MATERIALIZED VIEW advertiser_uv2 AS
SELECT advertiser, channel, hll_union(hll_hash(user_id))
FROM advertiser_view_record
GROUP BY advertiser, channel;
```

### 设置额外排序键

假设基表 `tableA` 包含列 `k1`、`k2` 和 `k3`，其中只有 `k1` 和 `k2` 是排序键。如果必须加速包含子查询 `where k3=x` 的查询，您可以创建一个同步物化视图，将 `k3` 作为第一列。

```SQL
CREATE MATERIALIZED VIEW k3_as_key AS
SELECT k3, k2, k1
FROM tableA
```

## 聚合函数对应关系

当使用同步物化视图执行查询时，原始查询语句将被自动重写并用于查询存储在同步物化视图中的中间结果。以下表格显示了原始查询中的聚合函数与用于构建同步物化视图的聚合函数之间的对应关系。您可以根据业务场景选择相应的聚合函数来构建同步物化视图。

| **原始查询中的聚合函数**           | **物化视图中使用的聚合函数** |
| ---------------------------------- | ----------------------------- |
| sum                                | sum                           |
| min                                | min                           |
| max                                | max                           |
| count                              | count                         |
| bitmap_union, bitmap_union_count, count(distinct) | bitmap_union          |
| hll_raw_agg, hll_union_agg, ndv, approx_count_distinct | hll_union           |