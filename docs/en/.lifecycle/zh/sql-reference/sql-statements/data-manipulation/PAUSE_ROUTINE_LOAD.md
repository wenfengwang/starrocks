---
displayed_sidebar: English
---

# 暂停例程加载

从 '../../../assets/commonMarkdown/RoutineLoadPrivNote.md' 导入 RoutineLoadPrivNote

## 描述

暂停例程加载作业，但不终止该作业。您可以执行 [RESUME ROUTINE LOAD](./RESUME_ROUTINE_LOAD.md) 来恢复它。加载作业暂停后，您可以执行 [SHOW ROUTINE LOAD](./SHOW_ROUTINE_LOAD.md) 和 [ALTER ROUTINE LOAD](./ALTER_ROUTINE_LOAD.md) 来查看和修改有关暂停加载作业的信息。

<RoutineLoadPrivNote />

## 语法

```SQL
PAUSE ROUTINE LOAD FOR [db_name.]<job_name>;
```

## 参数

| 参数 | 必填 | 描述                                                  |
| --------- | -------- | ------------------------------------------------------------ |
| db_name   |          | 例程加载作业所属的数据库的名称。 |
| job_name  | ✅        | 例程加载作业的名称。一个表可能有多个例程加载作业，建议使用可识别信息（例如创建加载作业时的 Kafka 主题名称或时间）设置有意义的例程加载作业名称，以区分多个例程加载作业。例程加载作业的名称在同一数据库中必须是唯一的。 |

## 例子

暂停数据库中的例程加载作业 `example_db.example_tbl1_ordertest1`。

```sql
PAUSE ROUTINE LOAD FOR example_db.example_tbl1_ordertest1;