---
displayed_sidebar: English
---

# 重复键表

重复键表是 StarRocks 的默认模型。如果在创建表时未指定模型，则默认创建重复键表。

创建重复键表时，可以为该表定义排序键。如果筛选条件包含排序键列，StarRocks 可以快速筛选表中的数据，加快查询速度。重复键表允许您向表中追加新数据。但是，不允许修改表中的现有数据。

## 场景

重复键表适用于以下场景：

- 分析原始数据，例如原始日志和原始操作记录。
- 使用各种方法查询数据，不受预聚合方法的限制。
- 加载日志数据或时序数据。新数据以追加模式写入，现有数据不会被更新。

## 创建表

假设您要分析特定时间范围内的事件数据。在本示例中，创建一个名为 `detail` 的表，并将 `event_time` 和 `event_type` 定义为排序键列。

创建表的语句：

```SQL
CREATE TABLE IF NOT EXISTS detail (
    event_time DATETIME NOT NULL COMMENT "事件的日期时间",
    event_type INT NOT NULL COMMENT "事件类型",
    user_id INT COMMENT "用户ID",
    device_code INT COMMENT "设备编码",
    channel INT COMMENT ""
)
DUPLICATE KEY(event_time, event_type)
DISTRIBUTED BY HASH(user_id);
```

> **注意**
>
> - 创建表时，必须使用 `DISTRIBUTED BY HASH` 子句来指定分桶列。有关详细信息，请参阅 [分桶](../Data_distribution.md#design-partitioning-and-bucketing-rules)。
> - 从 v2.5.7 开始，StarRocks 可以在创建表或添加分区时自动设置存储桶的数量 (BUCKETS)。您不再需要手动设置存储桶的数量。有关详细信息，请参阅 [确定存储桶的数量](../Data_distribution.md#determine-the-number-of-buckets)。

## 使用说明

- 有关表的排序键，请注意以下几点：
  - 您可以使用 `DUPLICATE KEY` 关键字显式定义排序键中使用的列。

    > 注意：默认情况下，如果不指定排序键列，StarRocks 会使用**前三列**作为排序键列。

  - 在重复键表中，排序键可以包含部分或全部维度列。

- 您可以在创建表时创建位图索引和Bloomfilter索引等索引。

- 如果加载了两条相同的记录，则重复键表会将它们保留为两条记录，而不是合并为一条记录。

## 下一步操作

创建表后，您可以使用各种数据引入方法将数据加载到 StarRocks 中。有关 StarRocks 支持的数据加载方法的信息，请参见 [数据加载概述](../../loading/Loading_intro.md)。
> 注意：将数据加载到使用重复键表的表中时，只能向表中追加数据，无法修改表中的现有数据。
