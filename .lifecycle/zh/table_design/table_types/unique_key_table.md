---
displayed_sidebar: English
---

# 唯一键表

在创建表时，可以定义主键列和度量列。这样，查询会返回具有相同主键的记录组中最新的记录。与重复键表相比，唯一键表简化了数据的加载过程，更好地支持实时和频繁的数据更新。

## 适用场景

唯一键表适合于需要频繁实时更新数据的业务场景。例如，在电商场景中，每天可能会产生数亿订单，订单状态的变化也非常频繁。

## 原理

唯一键表可以看作是一种特殊的聚合键表，在这种表中，度量列指定了REPLACE聚合函数，用以返回具有相同主键的记录组中最新的记录。

当您向使用唯一键表的表中加载数据时，数据会被分成多个批次加载。每个批次都会被分配一个版本号。因此，具有相同主键的记录可能会有多个版本，查询时会返回最新的版本（即，版本号最大的记录）。

如下表所示，ID是主键列，value是度量列，_version记录了StarRocks内部生成的数据版本号。在本例中，ID为1的记录被分为两个批次，其版本号分别为1和2进行加载；ID为2的记录被分为三个批次，其版本号分别为3、4和5进行加载。

|ID|值|_版本|
|---|---|---|
|1|100|1|
|1|101|2|
|2|100|3|
|2|101|4|
|2|102|5|

当查询ID为1的记录时，会返回版本号最大的最新记录，本例中为2。当查询ID为2的记录时，会返回版本号最大的最新记录，本例中为5。下表展示了两次查询返回的记录：

|ID|值|
|---|---|
|1|101|
|2|102|

## 创建表

在电商场景中，通常需要按日期收集和分析订单状态。在这个例子中，创建一个名为orders的表来存储订单信息，将经常用于过滤订单条件的create_time和order_id定义为主键列，并将其他两列order_state和total_price定义为度量列。这样一来，订单的状态更新可以实时反映，并且可以快速过滤，以加快查询速度。

创建表的语句如下：

```SQL
CREATE TABLE IF NOT EXISTS orders (
    create_time DATE NOT NULL COMMENT "create time of an order",
    order_id BIGINT NOT NULL COMMENT "id of an order",
    order_state INT COMMENT "state of an order",
    total_price BIGINT COMMENT "price of an order"
)
UNIQUE KEY(create_time, order_id)
DISTRIBUTED BY HASH(order_id);
```

> **注意**
- 创建表时，必须使用 `DISTRIBUTED BY HASH` 子句来指定分桶列。详细信息请参阅[bucketing](../Data_distribution.md#design-partitioning-and-bucketing-rules)。
- 从v2.5.7版本开始，StarRocks在创建表或添加分区时可以自动设置存储桶（BUCKETS）的数量，无需手动设置桶数量。详细信息请参阅[determine the number of buckets](../Data_distribution.md#determine-the-number-of-buckets)。

## 使用注意事项

- 关于表的主键，请注意以下几点：

  - 主键使用UNIQUE KEY关键字定义。
  - 主键必须创建在有唯一性约束且列名不可更改的列上。
  - 主键的设计必须恰当：
    - 在运行查询时，主键列在多个数据版本聚合之前进行过滤，而度量列在聚合之后进行过滤。因此，我们建议您识别经常用作过滤条件的列，并将这些列定义为主键列。这样，可以在多版本数据聚合之前开始过滤数据，从而提高查询性能。
    - 在聚合过程中，StarRocks会比较所有的主键列，这个过程非常耗时，可能会降低查询性能。因此，不应定义过多的主键列。如果某列很少作为查询条件使用，我们建议不要将其定义为主键列。

- 在创建表时，不能在表的度量列上创建BITMAP索引或Bloom Filter索引。

- 唯一键表不支持物化视图。

## 下一步要做的事情

创建表后，您可以使用多种数据导入方法将数据加载到StarRocks中。有关StarRocks支持的数据导入方法，请参阅[数据导入](../../loading/Loading_intro.md)。

- 当您向使用唯一键表的表中加载数据时，只能更新表的所有列。例如，当您更新前面提到的orders表时，必须更新所有列，包括create_time、order_id、order_state和total_price。
- 当您从使用唯一键表的表中查询数据时，StarRocks需要聚合多个数据版本的记录。在这种情况下，数据版本数量庞大会降低查询性能。因此，我们建议您设定一个合适的数据加载频率，既能满足实时数据分析的需求，又能避免产生过多的数据版本。如果您需要分钟级别的数据，可以将数据加载频率设定为每分钟一次，而不是每秒一次。
