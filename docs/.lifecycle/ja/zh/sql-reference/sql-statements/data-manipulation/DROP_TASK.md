---
displayed_sidebar: Chinese
---

# タスクの削除

## 機能

[SUBMIT TASK](./SUBMIT_TASK.md) ステートメントで提出された非同期 ETL タスクを削除します。この機能は StarRocks v2.5.7 からサポートされています。

> **注意**
>
> DROP TASK でタスクを削除すると、そのタスクに対応する TaskRun も同時にキャンセルされます。

## 文法

```SQL
DROP TASK <task_name>
```

## パラメータ説明

| **パラメータ** | **説明**         |
| -------------- | ---------------- |
| task_name      | 削除するタスク名。 |

## 使用説明

Information Schema のメタデータビュー `tasks` および `task_runs` をクエリして、非同期タスクの情報を確認できます。

```SQL
SELECT * FROM INFORMATION_SCHEMA.tasks;
SELECT * FROM information_schema.tasks WHERE task_name = '<task_name>';
SELECT * FROM information_schema.task_runs;
SELECT * FROM information_schema.task_runs WHERE task_name = '<task_name>';
```

## 例

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
