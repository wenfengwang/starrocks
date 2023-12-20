---
displayed_sidebar: English
---

# 显示动态分区表

## 描述

此语句用于展示数据库中所有配置了动态分区属性的分区表的状态。

## 语法

```sql
SHOW DYNAMIC PARTITION TABLES FROM <db_name>
```

此语句返回以下字段：

- TableName：表的名称。
- Enable：是否启用了动态分区。
- TimeUnit：分区的时间粒度。
- Start：动态分区的起始偏移。
- End：动态分区的结束偏移。
- Prefix：分区名称的前缀。
- Buckets：每个分区的桶数量。
- ReplicationNum：表的副本数量。
- LastUpdateTime：表最后更新的时间。
- LastSchedulerTime：表数据最后调度的时间。
- State：表的状态。
- LastCreatePartitionMsg：最新分区创建操作的消息。
- LastDropPartitionMsg：最新分区删除操作的消息。

## 示例

显示`db_test`数据库中配置了动态分区属性的所有分区表的状态。

```sql
SHOW DYNAMIC PARTITION TABLES FROM db_test;
```