---
displayed_sidebar: English
---

# 暂停例行加载任务

从 '../../../assets/commonMarkdown/RoutineLoadPrivNote.md' 导入 RoutineLoadPrivNote

## 描述

该操作会暂停一个例行加载任务，但不会终止该任务。您可以执行 [RESUME ROUTINE LOAD](./RESUME_ROUTINE_LOAD.md) 命令来恢复任务。在加载任务暂停后，您可以通过执行 [SHOW ROUTINE LOAD](./SHOW_ROUTINE_LOAD.md) 和 [ALTER ROUTINE LOAD](./ALTER_ROUTINE_LOAD.md) 命令来查看和修改暂停的加载任务信息。

<RoutineLoadPrivNote />


## 语法

```SQL
PAUSE ROUTINE LOAD FOR [db_name.]<job_name>;
```

## 参数

|参数|必填|说明|
|---|---|---|
|db_name|例程加载作业所属的数据库的名称。|
|job_name|✅|例行加载作业的名称。一张表可能有多个例行加载作业，建议使用可识别信息（例如Kafka主题名称或创建加载作业的时间）设置一个有意义的例行加载作业名称，以区分多个例行加载作业。 Routine Load作业的名称在同一数据库内必须是唯一的|

## 示例

在数据库 example_db 中暂停名为 example_tbl1_ordertest1 的例行加载任务。

```sql
PAUSE ROUTINE LOAD FOR example_db.example_tbl1_ordertest1;
```
