---
displayed_sidebar: English
---

# 停止常规加载任务

从 '../../../assets/commonMarkdown/RoutineLoadPrivNote.md' 导入 RoutineLoadPrivNote

## 说明

停止一个常规加载（Routine Load）任务。

<RoutineLoadPrivNote />


::: 警告

- 一旦停止，常规加载任务将无法恢复。因此，在执行此操作时请格外小心。
- 如果您只是想暂停常规加载任务，请使用 [PAUSE ROUTINE LOAD](./PAUSE_ROUTINE_LOAD.md) 命令。

:::

## 语法

```SQL
STOP ROUTINE LOAD FOR [db_name.]<job_name>
```

## 参数表

|参数|必填|说明|
|---|---|---|
|db_name|例程加载作业所属的数据库的名称。|
|job_name|✅|例行加载作业的名称。|

## 示例

在数据库 example_db 中停止常规加载任务 example_tbl1_ordertest1。

```SQL
STOP ROUTINE LOAD FOR example_db.example_tbl1_ordertest1;
```
