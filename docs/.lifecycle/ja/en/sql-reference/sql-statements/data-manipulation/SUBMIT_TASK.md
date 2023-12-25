---
displayed_sidebar: English
---

# タスクの送信

## 説明

ETLステートメントを非同期タスクとしてサブミットします。この機能はStarRocks v2.5以降でサポートされています。

StarRocks v3.0は、[CREATE TABLE AS SELECT](../data-definition/CREATE_TABLE_AS_SELECT.md)と[INSERT](./INSERT.md)の非同期タスクのサブミットをサポートしています。

非同期タスクは[DROP TASK](./DROP_TASK.md)を使用して削除できます。

## 構文

```SQL
SUBMIT TASK [task_name] AS <etl_statement>
```

## パラメーター

| **パラメーター** | **説明**                                              |
| ------------- | ------------------------------------------------------------ |
| task_name     | タスク名です。                                               |
| etl_statement | 非同期タスクとしてサブミットするETLステートメントです。StarRocksは現在、[CREATE TABLE AS SELECT](../data-definition/CREATE_TABLE_AS_SELECT.md)と[INSERT](./INSERT.md)の非同期タスクのサブミットをサポートしています。 |

## 使用上の注意

このステートメントは、ETLステートメントを実行するタスクを格納するテンプレートであるタスクを作成します。タスクの情報は、情報スキーマのメタデータビュー[`tasks`](../../../reference/information_schema/tasks.md)を照会することで確認できます。

```SQL
SELECT * FROM INFORMATION_SCHEMA.tasks;
SELECT * FROM information_schema.tasks WHERE task_name = '<task_name>';
```

タスクを実行すると、それに応じてTaskRunが生成されます。TaskRunは、ETLステートメントを実行するタスクを示します。TaskRunには以下の状態があります：

- `PENDING`: タスクは実行待ちです。
- `RUNNING`: タスクは実行中です。
- `FAILED`: タスクが失敗しました。
- `SUCCESS`: タスクが成功しました。

TaskRunの状態を確認するには、情報スキーマのメタデータビュー[`task_runs`](../../../reference/information_schema/task_runs.md)を照会します。

```SQL
SELECT * FROM INFORMATION_SCHEMA.task_runs;
SELECT * FROM information_schema.task_runs WHERE task_name = '<task_name>';
```

## FE設定項目による設定

以下のFE設定項目を使用して非同期ETLタスクを設定できます：

| **パラメーター**              | **デフォルト値** | **説明**                                              |
| -------------------------- | ----------------- | ------------------------------------------------------------ |
| task_ttl_second            | 259200            | タスクが有効な期間です。単位は秒です。有効期間を超えたタスクは削除されます。 |
| task_check_interval_second | 14400             | 無効なタスクを削除するための時間間隔です。単位は秒です。    |
| task_runs_ttl_second       | 259200            | TaskRunが有効な期間です。単位は秒です。有効期間を超えたTaskRunは自動的に削除されます。また、`FAILED`と`SUCCESS`状態のTaskRunも自動的に削除されます。 |
| task_runs_concurrency      | 20                | 並行して実行できるTaskRunの最大数です。  |
| task_runs_queue_length     | 500               | 実行待ちのTaskRunの最大数です。この数がデフォルト値を超えると、新規タスクは保留されます。 |

## 例

例1：`CREATE TABLE tbl1 AS SELECT * FROM src_tbl`の非同期タスクをサブミットし、タスク名を`etl0`として指定します：

```SQL
SUBMIT TASK etl0 AS CREATE TABLE tbl1 AS SELECT * FROM src_tbl;
```

例2：`INSERT INTO tbl2 SELECT * FROM src_tbl`の非同期タスクをサブミットし、タスク名を`etl1`として指定します：

```SQL
SUBMIT TASK etl1 AS INSERT INTO tbl2 SELECT * FROM src_tbl;
```

例3：`INSERT OVERWRITE tbl3 SELECT * FROM src_tbl`の非同期タスクをサブミットします：

```SQL
SUBMIT TASK AS INSERT OVERWRITE tbl3 SELECT * FROM src_tbl;
```

例4：タスク名を指定せずに`INSERT OVERWRITE insert_wiki_edit SELECT * FROM source_wiki_edit`の非同期タスクをサブミットし、ヒントを使用してクエリタイムアウトを`100000`秒に延長します：

```SQL
SUBMIT /*+set_var(query_timeout=100000)*/ TASK AS
INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```
