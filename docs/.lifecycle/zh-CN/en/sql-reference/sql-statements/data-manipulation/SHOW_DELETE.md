---
displayed_sidebar: "Chinese"
---

# 显示删除

## 描述

此语句用于显示当前数据库中成功执行的重复键表上的历史删除任务。有关数据删除的更多信息，请参阅[DELETE](DELETE.md)。

## 语法

```sql
SHOW DELETE [FROM <db_name>]
```

`db_name`：数据库名称，可选。如果未指定此参数，默认使用当前数据库。

返回字段：

- TableName：删除数据的表。
- PartitionName：删除数据的分区。如果表是非分区表，则显示 `*`。
- CreateTime：创建删除任务的时间。
- DeleteCondition：指定的删除条件。
- State：删除任务的状态。

## 示例

显示 `database` 的所有历史删除任务。

```sql
SHOW DELETE FROM database;

+------------+---------------+---------------------+-----------------+----------+
| TableName  | PartitionName | CreateTime          | DeleteCondition | State    |
+------------+---------------+---------------------+-----------------+----------+
| mail_merge | *             | 2023-03-14 10:39:03 | name EQ "Peter" | FINISHED |
+------------+---------------+---------------------+-----------------+----------+
```