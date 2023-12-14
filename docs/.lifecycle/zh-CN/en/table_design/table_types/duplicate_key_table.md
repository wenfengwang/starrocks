---
displayed_sidebar: "Chinese"
---

# 重复键表

重复键表是StarRocks中的默认模型。如果在创建表时没有指定模型，则默认创建重复键表。

创建重复键表时，可以为该表定义一个排序键。如果过滤条件包含排序键列，则StarRocks可以快速地从表中过滤数据以加速查询。重复键表允许将新数据附加到表中。但是，它不允许修改表中的现有数据。

## 场景

重复键表适用于以下场景：

- 分析原始数据，如原始日志和原始操作记录。
- 使用各种方法查询数据，而不受预聚合方法的限制。
- 加载日志数据或时间序列数据。新数据以追加模式写入，现有数据不会被更新。

## 创建表

假设要分析特定时间范围内的事件数据。在本示例中，创建一个名为`detail`的表，并将`event_time`和`event_type`定义为排序键列。

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
> - 创建表时，必须使用`DISTRIBUTED BY HASH`子句指定分桶列。有关详细信息，请参见[分桶](../Data_distribution.md#design-partitioning-and-bucketing-rules)。
> - 从v2.5.7开始，StarRocks在创建表或添加分区时可以自动设置分桶的数量（BUCKETS）。不再需要手动设置分桶的数量。有关详细信息，请参见[确定分桶的数量](../Data_distribution.md#determine-the-number-of-buckets)。

## 使用说明

- 请注意有关表的排序键的以下要点：
  - 可以使用`DUPLICATE KEY`关键字显式定义排序键中使用的列。

    > 注意：默认情况下，如果未指定排序键列，则StarRocks使用**前三个**列作为排序键列。

  - 在重复键表中，排序键可以由一些或所有维度列组成。

- 可以在表创建时创建位图索引和布隆过滤器索引。

- 如果加载两个相同记录，重复键表会将它们保留为两条记录，而不是一条记录。

## 接下来的操作

创建表后，可以使用各种数据摄入方法将数据加载到StarRocks中。有关StarRocks支持的数据摄入方法的信息，请参见[数据加载概述](../../loading/Loading_intro.md)。
> 注意：当将数据加载到使用重复键表的表中时，只能将数据附加到表中。不能修改表中的现有数据。