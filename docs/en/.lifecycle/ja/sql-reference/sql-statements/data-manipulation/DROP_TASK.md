---
displayed_sidebar: "Japanese"
---

# DROP TASK（タスクの削除）

## 説明

[SUBMIT TASK（タスクの送信）](./SUBMIT_TASK.md)を使用して送信された非同期ETLタスクを削除します。この機能はStarRocks v2.5.7以降でサポートされています。

> **注意**
>
> DROP TASKを使用してタスクを削除すると、対応するTaskRunも同時にキャンセルされます。

## 構文

```SQL
DROP TASK '<task_name>'
```

## パラメータ

| **パラメータ** | **説明**                     |
| ------------- | ----------------------------- |
| task_name     | 削除するタスクの名前。        |

## 使用上の注意

非同期タスクの情報は、Information Schemaのメタデータビューである`tasks`と`task_runs`をクエリして確認できます。

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
1 行が返されました (1.19 秒)

MySQL > DROP TASK 'ctas';
クエリは正常に終了しましたが、結果は返されませんでした (0.35 秒)
```
