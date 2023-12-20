---
displayed_sidebar: English
---

# 显示删除历史

## 描述

此语句用于展示当前数据库中重复键表上成功执行的历史删除（DELETE）任务。更多关于数据删除的信息，请参见[DELETE](DELETE.md)指令。

## 语法

```sql
SHOW DELETE [FROM <db_name>]
```

db_name：数据库名，为可选项。若未指定此参数，默认使用当前数据库。

返回字段：

- TableName：已删除数据的表名。
- PartitionName：已删除数据的分区名。若表为非分区类型，则显示*。
- CreateTime：删除（DELETE）任务创建的时间。
- DeleteCondition：特定的删除（DELETE）条件。
- State：删除（DELETE）任务的状态。

## 示例

展示数据库的所有历史删除（DELETE）任务。

```sql
SHOW DELETE FROM database;

+------------+---------------+---------------------+-----------------+----------+
| TableName  | PartitionName | CreateTime          | DeleteCondition | State    |
+------------+---------------+---------------------+-----------------+----------+
| mail_merge | *             | 2023-03-14 10:39:03 | name EQ "Peter" | FINISHED |
+------------+---------------+---------------------+-----------------+----------+
```
