---
displayed_sidebar: English
---

# 同步物化视图

本主题描述了如何创建、使用和管理**同步物化视图（Rollup）**。

对于同步物化视图，基表中的所有更改都会同时更新到相应的同步物化视图中。同步物化视图的刷新是自动触发的。同步物化视图的维护和更新成本非常低，适用于透明加速实时单表聚合查询。

StarRocks中的同步物化视图只能在[默认目录](../data_source/catalog/default_catalog.md)中的单个基表上创建。它们本质上是用于查询加速的特殊索引，而不是像异步物化视图那样的物理表。

从v2.4开始，StarRocks提供了异步物化视图，支持在多个表上创建，并支持更多的聚合运算符。有关**异步物化视图**的用法，请参见[异步物化视图](../using_starrocks/Materialized_view.md)。

:::注意
目前，共享数据集群尚不支持同步物化视图。
:::

下表比较了StarRocks v2.5、v2.4中的异步物化视图（ASYNC MV）和同步物化视图（SYNC MV）在功能方面的支持：

|                       | **单表聚合** | **多表联接** | **查询重写** | **刷新策略** | **基表** |
| --------------------- | ---------------------------- | -------------------- | ----------------- | -------------------- | -------------- |
| **ASYNC MV** | 是 | 是 | 是 | <ul><li>异步刷新</li><li>手动刷新</li></ul> | 来自以下多个表：<ul><li>默认目录</li><li>外部目录（v2.5）</li><li>现有物化视图（v2.5）</li><li>现有视图（v3.1）</li></ul> |
| **SYNC MV（Rollup）**  | 聚合函数的选择受限 | 否 | 是 | 数据加载期间的同步刷新 | 默认目录中的单个表 |

## 基本概念

- **基表**

  基表是物化视图的驱动表。

  对于StarRocks的同步物化视图，基表必须是[默认目录](../data_source/catalog/default_catalog.md)中的单个原生表。StarRocks支持在Duplicate Key表、Aggregate表和Unique Key表上创建同步物化视图。

- **刷新**

  同步物化视图会在基表数据发生更改时自动更新，无需手动触发。

- **查询重写**

  查询重写是指在基表上执行查询时，系统会自动判断物化视图中预先计算的结果是否可用于查询。如果可用，系统将直接从相关物化视图加载数据，避免耗时和资源的计算或连接。

  同步物化视图支持基于某些聚合运算符的查询重写。更多信息，请参见[聚合函数的对应关系](#correspondence-of-aggregate-functions)。

## 准备工作

在创建同步物化视图之前，请检查数据仓库是否符合通过同步物化视图进行查询加速的条件，例如，检查查询是否重用了某些子查询语句。

以下示例基于表`sales_records`，其中包含每个交易的交易ID`record_id`、销售人员ID`seller_id`、商店ID`store_id`、日期`sale_date`和销售金额`sale_amt`。按以下步骤创建表并插入数据：

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

该示例的业务场景需要频繁分析不同商店的销售额。因此，每个查询都使用`sum()`函数，消耗大量计算资源。您可以运行查询以记录其时间，并使用EXPLAIN命令查看其查询配置文件。

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

可以看到，查询耗时约为0.02秒，并且查询配置文件中的`rollup`字段的值为`sales_records`，因此未使用同步物化视图加速查询。

## 创建同步物化视图

您可以使用[CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md)基于特定查询语句创建同步物化视图。

根据表`sales_records`和上述查询语句，以下示例创建了同步物化视图`store_amt`，用于分析每个商店的销售金额总和。

```SQL
CREATE MATERIALIZED VIEW store_amt AS
SELECT store_id, SUM(sale_amt)
FROM sales_records
GROUP BY store_id;
```

> **注意**
>
> - 在同步物化视图中使用聚合函数时，必须使用GROUP BY子句，并在SELECT列表中指定至少一个GROUP BY列。
> - 同步物化视图不支持在多个列上使用一个聚合函数。不支持形式的查询语句`sum(a+b)`。
> - 同步物化视图不支持在一列上使用多个聚合函数。不支持形式的查询语句`select sum(a), min(a) from table`。
> - 创建同步物化视图时不支持JOIN。
> - 使用ALTER TABLE DROP COLUMN删除基表中的特定列时，需要确保基表的所有同步物化视图都不包含已删除的列，否则无法执行删除操作。若要删除同步物化视图中使用的列，需要首先删除包含该列的所有同步物化视图，然后删除该列。
> - 为表创建过多的同步物化视图会影响数据加载效率。当数据加载到基表时，同步物化视图和基表中的数据会同步更新。如果基表中包含`n`个同步物化视图，则将数据加载到基表中的效率与将数据加载到`n`个表中的效率大致相同。
> - 目前，StarRocks不支持同时创建多个同步物化视图。只有在前一个同步物化视图完成后，才能创建新的同步物化视图。
> - 共享数据的StarRocks集群不支持同步物化视图。

## 检查同步物化视图的构建状态

创建同步物化视图是一种异步操作。成功执行CREATE MATERIALIZED VIEW表示成功提交了创建物化视图的任务。您可以通过[SHOW ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/SHOW_ALTER_MATERIALIZED_VIEW.md)在数据库中查看同步物化视图的构建状态。

```Plain

MySQL > SHOW CREATE MATERIALIZED VIEW\G
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

`RollupIndexName` 部分表示同步物化视图的名称，`State` 部分表示建设是否已完成。

## 直接查询同步物化视图

由于同步物化视图本质上是基表的索引而不是物理表，因此只能使用提示`[_SYNC_MV_]`来查询同步物化视图：

```SQL
-- 不要忽略提示中的方括号 []。
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
> 目前，StarRocks 会自动为同步物化视图中的列生成名称，即使您为它们指定了别名。

## 重写和加速查询使用同步物化视图

您创建的同步物化视图包含根据查询语句预先计算的完整结果集。后续查询将使用其中的数据。您可以运行与准备工作时相同的查询来测试查询时间。

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

再次执行 EXPLAIN 命令以检查查询是否命中同步物化视图。

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

可以观察到查询配置文件中 `rollup` 部分的值现在是 `store_amt`，这是您构建的同步物化视图。这意味着此查询已命中同步物化视图。

## 显示同步物化视图

您可以执行 DESC \<tbl_name\> ALL 来检查表及其从属同步物化视图的架构。

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

- 您创建了错误的物化视图，需要在建设完成之前将其删除。
- 您创建了太多的物化视图，导致负载性能大幅下降，并且一些物化视图是重复的。
- 涉及的查询频率较低，您可以容忍相对较高的查询延迟。

### 删除未完成的同步物化视图

您可以通过取消正在进行的创建任务来删除正在创建的同步物化视图。首先，您需要通过[检查同步物化视图的建设状态](#check-the-building-status-of-a-synchronous-materialized-view)来获取物化视图创建任务的作业 ID `JobID`。获取作业 ID 后，需要使用 CANCEL ALTER 命令取消创建任务。

```Plain
CANCEL ALTER TABLE ROLLUP FROM sales_records (12090);
```

### 删除现有的同步物化视图

可以使用 [DROP MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/DROP_MATERIALIZED_VIEW.md) 命令删除现有的同步物化视图。

```SQL
DROP MATERIALIZED VIEW store_amt;
```

## 最佳实践

### 精确计数不同

以下示例基于广告业务分析表 `advertiser_view_record`，该表记录了广告被观看的日期 `click_time`、广告的名称 `advertiser`、广告的渠道 `channel`以及查看该广告的用户ID `user_id`。

```SQL
CREATE TABLE advertiser_view_record(
    click_time DATE,
    advertiser VARCHAR(10),
    channel VARCHAR(10),
    user_id INT
) distributed BY hash(click_time);
```

分析主要集中在广告的紫外线上。

```SQL
SELECT advertiser, channel, count(distinct user_id)
FROM advertiser_view_record
GROUP BY advertiser, channel;
```

为了加速精确计数的区分，您可以基于此表创建同步物化视图，并使用 bitmap_union 函数对数据进行预聚合。

```SQL
CREATE MATERIALIZED VIEW advertiser_uv AS
SELECT advertiser, channel, bitmap_union(to_bitmap(user_id))
FROM advertiser_view_record
GROUP BY advertiser, channel;
```

创建同步物化视图后，后续查询中的子查询将自动重写为 `bitmap_union_count (to_bitmap(user_id))` 以便它们可以命中同步物化视图。

### 近似计数不同

再次使用上表 `advertiser_view_record` 作为示例。为了加速近似计数的区分，您可以基于此表创建同步物化视图，并使用 [hll_union（）](../sql-reference/sql-functions/aggregate-functions/hll_union.md) 函数对数据进行预聚合。

```SQL
CREATE MATERIALIZED VIEW advertiser_uv2 AS
SELECT advertiser, channel, hll_union(hll_hash(user_id))
FROM advertiser_view_record
GROUP BY advertiser, channel;
```

### 设置额外的排序键

假设基表 `tableA` 包含列 `k1`、`k2` 和 `k3`，其中只有 `k1` 和 `k2` 是排序键。如果必须加速包含子查询 `where k3=x` 的查询，您可以创建以 `k3` 为第一列的同步物化视图。

```SQL
CREATE MATERIALIZED VIEW k3_as_key AS
SELECT k3, k2, k1
FROM tableA
```

## 聚合函数的对应关系

使用同步物化视图执行查询时，原始查询语句将自动重写并用于查询存储在同步物化视图中的中间结果。下表显示了原始查询中的聚合函数与用于构造同步物化视图的聚合函数之间的对应关系。您可以根据业务场景选择对应的聚合函数构建同步物化视图。

| **原始查询中的聚合函数**           | **物化视图的聚合函数** |
| ------------------------------------------------------ | ----------------------------------------------- |
| 和                                                    | 和 |
| 最小值                                                    | 最小值                                             |
| 最大值                                                    | 最大值                                             |
| 计数                                                  | 计数                                           |
| bitmap_union、bitmap_union_count、count(distinct)      | bitmap_union                                    |
| hll_raw_agg、hll_union_agg、ndv、approx_count_distinct | hll_union                                       |