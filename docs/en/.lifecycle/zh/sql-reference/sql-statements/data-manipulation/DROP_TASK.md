---
displayed_sidebar: English
---

# 删除任务

## 描述

删除使用 [SUBMIT TASK](./SUBMIT_TASK.md) 提交的异步 ETL 任务。此功能自 StarRocks v2.5.7 版本开始支持。

> **注意**
>
> 使用 DROP TASK 同时会取消相应的 TaskRun。

## 语法

```SQL
DROP TASK <task_name>
```

## 参数

| **参数** | **描述**               |
| ------------- | ----------------------------- |
| task_name     | 要删除的任务名称。 |

## 使用说明

您可以通过查询信息模式中的元数据视图 `tasks` 和 `task_runs` 来查看异步任务的信息。

```SQL
SELECT * FROM INFORMATION_SCHEMA.tasks;
SELECT * FROM information_schema.tasks WHERE task_name = '<task_name>';
SELECT * FROM information_schema.task_runs;
SELECT * FROM information_schema.task_runs WHERE task_name = '<task_name>';
```

## 例子

```Plain
MySQL > SUBMIT /*+set_var(query_timeout=100000)*/ TASK ctas AS
    -> CREATE TABLE insert_wiki_edit_new
    -> AS SELECT * FROM source_wiki_edit;
+----------+-----------+
| TaskName | Status    |
+----------+-----------+
| ctas     | SUBMITTED |
+----------+-----------+
1 row in set (1.19 sec)

MySQL > DROP TASK ctas;
Query OK, 0 行受到影响 (0.35 秒)