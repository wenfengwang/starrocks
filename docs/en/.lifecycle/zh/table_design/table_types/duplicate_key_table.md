---
displayed_sidebar: English
---

# Duplicate Key 表

Duplicate Key 表是 StarRocks 中的默认模型。如果在创建表时未指定模型，则默认创建 Duplicate Key 表。

创建 Duplicate Key 表时，您可以为该表定义排序键。如果过滤条件包含排序键列，StarRocks 可以快速过滤表中的数据以加速查询。Duplicate Key 表允许您向表中追加新数据。但是，它不允许您修改表中的现有数据。

## 应用场景

Duplicate Key 表适用于以下场景：

- 分析原始数据，例如原始日志和操作记录。
- 使用多种方法查询数据，不受预聚合方式的限制。
- 加载日志数据或时间序列数据。新数据以追加模式写入，现有数据不进行更新。

## 创建表

假设您要分析特定时间范围内的事件数据。在此示例中，创建一个名为 `detail` 的表，并将 `event_time` 和 `event_type` 定义为排序键列。

创建表的语句：

```SQL
CREATE TABLE IF NOT EXISTS detail (
    event_time DATETIME NOT NULL COMMENT "event datetime",
    event_type INT NOT NULL COMMENT "event type",
    user_id INT COMMENT "user id",
    device_code INT COMMENT "device code",
    channel INT COMMENT ""
)
DUPLICATE KEY(event_time, event_type)
DISTRIBUTED BY HASH(user_id);
```

> **注意**
- 创建表时，必须使用 `DISTRIBUTED BY HASH` 子句指定分桶列。有关详细信息，请参阅 [bucketing](../Data_distribution.md#design-partitioning-and-bucketing-rules)。
- 从 v2.5.7 版本开始，StarRocks 在创建表或添加分区时可以自动设置存储桶（BUCKETS）的数量。您不再需要手动设置存储桶的数量。有关详细信息，请参阅 [determine the number of buckets](../Data_distribution.md#determine-the-number-of-buckets)。

## 使用须知

- 关于表的排序键，请注意以下几点：
-   您可以使用 `DUPLICATE KEY` 关键字显式定义用于排序键的列。

        > 注意：默认情况下，如果您不指定排序键列，StarRocks 会使用**前三个**列作为排序键列。

-   在 Duplicate Key 表中，排序键可以由部分或全部维度列组成。

- 您可以在创建表时创建索引，如 BITMAP 索引和 Bloomfilter 索引。

- 如果加载了两条相同的记录，Duplicate Key 表会将它们保留为两条记录，而不是合并为一条记录。

## 下一步

创建表后，您可以使用各种数据摄取方法将数据加载到 StarRocks 中。有关 StarRocks 支持的数据摄取方法的信息，请参阅 [数据加载概述](../../loading/Loading_intro.md)。
> 注意：当您将数据加载到使用 Duplicate Key 表的表中时，您只能追加数据到该表。您无法修改表中的现有数据。