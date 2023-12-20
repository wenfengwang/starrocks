---
displayed_sidebar: English
---

# 显示删除操作

## 描述

此语句用于展示当前数据库中，对重复键表成功执行的历史 DELETE 任务。更多关于数据删除的信息，请参见 [DELETE](DELETE.md)。

## 语法

```sql
SHOW DELETE [FROM <db_name>]
```

`db_name`：数据库名称，可选。若未指定此参数，默认使用当前数据库。

返回字段：

- TableName：数据被删除的表名。
- PartitionName：数据被删除的分区。若表为非分区表，则显示 `*`。
- CreateTime：DELETE 任务创建的时间。
- DeleteCondition：指定的 DELETE 条件。
- State：DELETE 任务的状态。

## 示例

展示 `database` 的所有历史 DELETE 任务。

```sql
SHOW DELETE FROM database;

+------------+---------------+---------------------+-----------------+----------+
| TableName  | PartitionName | CreateTime          | DeleteCondition | State    |
+------------+---------------+---------------------+-----------------+----------+
| mail_merge | *             | 2023-03-14 10:39:03 | name EQ "Peter" | FINISHED |
+------------+---------------+---------------------+-----------------+----------+
```