---
displayed_sidebar: "Chinese"
---

# 删除任务

## 描述

删除使用 [SUBMIT TASK](./SUBMIT_TASK.md) 提交的异步 ETL 任务。该功能自 StarRocks v2.5.7 起开始支持。

> **注意**
>
> 使用 DROP TASK 删除任务将同时取消相应的 TaskRun。

## 语法

```SQL
DROP TASK <task_name>
```

## 参数

| **参数**   | **描述**                 |
| ----------- | -------------------------- |
| task_name   | 要删除的任务的名称。    |

## 使用注意事项

您可以通过在信息模式中查询元数据视图 `tasks` 和 `task_runs` 来查看异步任务的信息。

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
| TaskName | Status    |
+----------+-----------+
| ctas     | SUBMITTED |
+----------+-----------+
1 row in set (1.19 sec)

MySQL > DROP TASK ctas;
Query OK, 0 rows affected (0.35 sec)
```