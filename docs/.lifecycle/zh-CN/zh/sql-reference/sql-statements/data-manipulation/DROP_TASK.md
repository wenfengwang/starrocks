---
displayed_sidebar: "Chinese"
---

# 删除任务

## 功能

删除通过 [SUBMIT TASK](./SUBMIT_TASK.md) 命令提交的异步 ETL 任务。从 StarRocks v2.5.7 版开始支持此功能。

> **说明**
>
> 使用 DROP TASK 命令删除任务将同时取消该任务对应的 TaskRun。

## 语法

```SQL
DROP TASK <task_name>
```

## 参数说明

| **参数**  | **说明**       |
| --------- | -------------- |
| task_name | 待删除的任务名称。 |

## 使用说明

您可以通过查询 Information Schema 中的元数据视图 `tasks` 和 `task_runs` 来查看异步任务的信息。

```SQL
SELECT * FROM INFORMATION_SCHEMA.tasks;
SELECT * FROM information_schema.tasks WHERE task_name = '<task_name>';
SELECT * FROM information_schema.task_runs;
SELECT * FROM information_schema.task_runs WHERE task_name = '<task_name>';
```

## 示例

```Plain
MySQL > SUBMIT /*+set_var(query_timeout=100000)*/ TASK ctas AS
    -> CREATE TABLE insert_wiki_edit_new
    -> AS SELECT * FROM source_wiki_edit;
+----------+-----------+
| 任务名称  | 状态       |
+----------+-----------+
| ctas     | SUBMITTED |
+----------+-----------+
1 行受影响 (1.19 秒)

MySQL > DROP TASK ctas;
Query OK, 0 行受影响 (0.35 秒)
```