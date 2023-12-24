---
displayed_sidebar: English
---

# 显示 DELETE

## 描述

该语句用于显示在当前数据库中成功执行的历史 DELETE 任务，这些任务是针对重复键表执行的。有关数据删除的详细信息，请参阅 [DELETE](DELETE.md)。

## 语法

```sql
SHOW DELETE [FROM <db_name>]
```

`db_name`：数据库名称，可选。如果未指定该参数，则默认使用当前数据库。

返回字段：

- TableName：删除数据的表名。
- PartitionName：删除数据的分区名。如果该表是非分区表，则显示 `*`。
- CreateTime：DELETE 任务创建的时间。
- DeleteCondition：指定的 DELETE 条件。
- State：DELETE 任务的状态。

## 例子

显示 `database` 中所有历史的 DELETE 任务。

```sql
SHOW DELETE FROM database;

+------------+---------------+---------------------+-----------------+----------+
| TableName  | PartitionName | CreateTime          | DeleteCondition | State    |
+------------+---------------+---------------------+-----------------+----------+
| mail_merge | *             | 2023-03-14 10:39:03 | name EQ "Peter" | FINISHED |
+------------+---------------+---------------------+-----------------+----------+