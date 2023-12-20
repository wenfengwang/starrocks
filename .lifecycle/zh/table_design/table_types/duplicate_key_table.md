---
displayed_sidebar: English
---

# 重复键表

重复键表是StarRocks中的默认模型。如果在创建表时没有指定模型，系统会默认创建一个重复键表。

当您创建一个重复键表时，可以为该表定义一个排序键。如果过滤条件包含了排序键的列，StarRocks能够快速地从表中过滤数据，以此加快查询速度。重复键表允许您向表中追加新数据，但不支持修改表中已经存在的数据。

## 适用场景

重复键表适合以下几种场景：

- 分析原始数据，比如原始日志和操作记录。
- 使用多种方法查询数据，不受预先聚合方式的限制。
- 加载日志数据或时间序列数据。新数据以追加模式写入，而现有数据不进行更新。

## 创建表格

假设您想分析特定时间范围内的事件数据。在这个例子中，创建一个名为“detail”的表，并定义“event_time”和“event_type”作为排序键列。

创建表的语句如下：

```SQL
CREATE TABLE IF NOT EXISTS detail (
    event_time DATETIME NOT NULL COMMENT "datetime of event",
    event_type INT NOT NULL COMMENT "type of event",
    user_id INT COMMENT "id of user",
    device_code INT COMMENT "device code",
    channel INT COMMENT ""
)
DUPLICATE KEY(event_time, event_type)
DISTRIBUTED BY HASH(user_id);
```

> **注意事项**
- 在创建表时，您必须使用`DISTRIBUTED BY HASH`子句来指定分桶列。更多详细信息，请参见[分桶](../Data_distribution.md#design-partitioning-and-bucketing-rules)的文档。
- 从v2.5.7版本开始，StarRocks可以自动设置表或添加分区时的桶（BUCKETS）数量，无需您手动设置。详细信息请参阅[determine the number of buckets](../Data_distribution.md#determine-the-number-of-buckets)。

## 使用须知

- 关于表的排序键，请注意以下几点：
-   您可以使用“DUPLICATE KEY”关键字来明确指定哪些列作为排序键。

        > 注：默认情况下，如果您没有指定排序键列，StarRocks会将**前三**列作为排序键列。

-   在重复键表中，排序键可以由部分或全部维度列组成。

- 您可以在创建表的同时创建像BITMAP索引和Bloomfilter索引这样的索引。

- 如果有两条完全相同的记录被加载，重复键表会将它们保留为两条独立的记录，而不是合并为一条。

## 下一步

在创建表之后，您可以使用多种数据导入方法将数据加载到**StarRocks**中。关于**StarRocks**支持的数据加载方法的信息，请参阅[数据加载概览](../../loading/Loading_intro.md)。
> 注意：当您向使用重复键表的表中加载数据时，只能追加数据到表中，不能修改表中已存在的数据。
