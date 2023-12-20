---
displayed_sidebar: English
---

# 删除任务

## 描述

删除一个使用 [SUBMIT TASK](./SUBMIT_TASK.md) 提交的异步 ETL 任务。该功能自 StarRocks v2.5.7 起支持。

> **注意**
> 使用 **DROP TASK** 删除任务会同时取消对应的 **TaskRun**。

## 语法

```SQL
DROP TASK <task_name>
```

## 参数

|**参数**|**说明**|
|---|---|
|task_name|要删除的任务名称。|

## 使用说明

您可以通过查询 Information Schema 中的 `tasks` 和 `task_runs` 元数据视图来检查异步任务的信息。

```SQL
SELECT * FROM INFORMATION_SCHEMA.tasks;
SELECT * FROM INFORMATION_SCHEMA.tasks WHERE task_name = '<task_name>';
SELECT * FROM INFORMATION_SCHEMA.task_runs;
SELECT * FROM INFORMATION_SCHEMA.task_runs WHERE task_name = '<task_name>';
```

## 示例

```Plain
MySQL > SUBMIT /*+set_var(query_timeout=100000)*/ TASK ctas AS
    -> CREATE TABLE insert_wiki_edit_new
    -> AS SELECT * FROM source_wiki_edit;
+----------+-----------+
| TaskName | Status    |
+----------+-----------+
| ctas     | SUBMITTED |
+----------+-----------+
1 行在集合中 (1.19 秒)

MySQL > DROP TASK ctas;
Query OK, 0 行受影响 (0.35 秒)
```