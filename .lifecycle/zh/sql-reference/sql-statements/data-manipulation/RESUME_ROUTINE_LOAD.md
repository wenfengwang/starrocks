---
displayed_sidebar: English
---

# 恢复常规负载作业

从 '../../../assets/commonMarkdown/RoutineLoadPrivNote.md' 导入 RoutineLoadPrivNote

## 描述

恢复一个常规负载作业。作业将暂时进入**NEED_SCHEDULE**状态，因为作业正在重新调度。经过一段时间，作业将恢复到**RUNNING**状态，继续消费来自数据源的消息并加载数据。您可以使用[SHOW ROUTINE LOAD](./SHOW_ROUTINE_LOAD.md)语句来查看作业的信息。

<RoutineLoadPrivNote />


## 语法

```SQL
RESUME ROUTINE LOAD FOR [db_name.]<job_name>
```

## 参数

|参数|必填|说明|
|---|---|---|
|db_name|例程加载作业所属的数据库的名称。|
|job_name|✅|例行加载作业的名称。|

## 示例

在 example_db 数据库中恢复常规负载作业 example_tbl1_ordertest1。

```SQL
RESUME ROUTINE LOAD FOR example_db.example_tbl1_ordertest1;
```
