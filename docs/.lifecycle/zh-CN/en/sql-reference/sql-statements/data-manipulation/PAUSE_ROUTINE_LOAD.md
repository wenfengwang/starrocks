---
displayed_sidebar: "Chinese"
---

# 暂停例行加载

## 描述

此语句暂停例行加载作业，但不终止此作业。您可以执行[恢复例行加载](./RESUME_ROUTINE_LOAD.md)来恢复它。在加载作业暂停后，您可以执行[显示例行加载](./SHOW_ROUTINE_LOAD.md)和[修改例行加载](./ALTER_ROUTINE_LOAD.md)来查看和修改有关已暂停加载作业的信息。

> **注意**
>
> 只有对加载数据的表具有LOAD_PRIV特权的用户才有权限暂停该表上的例行加载作业。

## 语法

```SQL
PAUSE ROUTINE LOAD FOR <db_name>.<job_name>;
```

## 参数

| 参数      | 是否必须 | 描述                                                         |
| --------- | -------- | ------------------------------------------------------------ |
| db_name   |          | 要暂停例行加载作业的数据库名称。                                  |
| job_name  | ✅        | 例行加载作业的名称。一个表可能有多个例行加载作业，建议使用可识别的信息来设置有意义的例行加载作业名称，例如Kafka主题名称或创建加载作业的时间，以区分多个例行加载作业。例行加载作业的名称在同一数据库内必须是唯一的。 |

## 示例

暂停数据库`example_db`中的例行加载作业`example_tbl1_ordertest1`。

```sql
PAUSE ROUTINE LOAD FOR example_db.example_tbl1_ordertest1;
```