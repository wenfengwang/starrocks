---
displayed_sidebar: English
---

# 显示动态分区表

## 描述

此语句用于显示数据库中配置了动态分区属性的所有分区表的状态。

## 语法

```sql
SHOW DYNAMIC PARTITION TABLES FROM <db_name>
```

此语句返回以下字段：

- TableName：表的名称。
- Enable：动态分区是否已启用。
- TimeUnit：分区的时间粒度。
- Start：动态分区的起始偏移量。
- End：动态分区的结束偏移量。
- Prefix：分区名称的前缀。
- Buckets：每个分区的存储桶数量。
- ReplicationNum：表的副本数。
- StartOf：动态分区的开始时间。
- LastUpdateTime：表的最后更新时间。
- LastSchedulerTime：表中数据的最后调度时间。
- State：表的状态。
- LastCreatePartitionMsg：最近一次分区创建操作的消息。
- LastDropPartitionMsg：最近一次分区删除操作的消息。

## 例子

显示数据库`db_test`中配置了动态分区属性的所有分区表的状态。

```sql
SHOW DYNAMIC PARTITION TABLES FROM db_test;
```