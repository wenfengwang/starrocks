---
displayed_sidebar: English
---

# 展示动态分区表

## 说明

此语句用于展示数据库中已配置动态分区属性的所有分区表的状态。

## 语法

```sql
SHOW DYNAMIC PARTITION TABLES FROM <db_name>
```

此语句将返回以下字段：

- TableName：表名。
- Enable：动态分区是否已启用。
- TimeUnit：分区的时间粒度。
- Start：动态分区的起始偏移。
- End：动态分区的结束偏移。
- Prefix：分区名的前缀。
- Buckets：每个分区的桶数量。
- ReplicationNum：表的副本数量。
- StartOf
- LastUpdateTime：表最后更新的时间。
- LastSchedulerTime：表数据最后调度的时间。
- State：表的当前状态。
- LastCreatePartitionMsg：最新分区创建操作的信息。
- LastDropPartitionMsg：最新分区删除操作的信息。

## 示例

展示在db_test数据库中已配置动态分区属性的所有分区表的状态。

```sql
SHOW DYNAMIC PARTITION TABLES FROM db_test;
```
