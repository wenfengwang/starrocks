---
displayed_sidebar: "Japanese"
---

# タスクの送信

## 説明

非同期タスクとしてETLステートメントを送信します。この機能はStarRocks v2.5からサポートされています。

StarRocks v3.0では、[CREATE TABLE AS SELECT](../data-definition/CREATE_TABLE_AS_SELECT.md)や[INSERT](./INSERT.md)の非同期タスクの送信がサポートされています。

[DROP TASK](./DROP_TASK.md)を使用して非同期タスクを削除することができます。

## 構文

```SQL
SUBMIT TASK [task_name] AS <etl_statement>
```

## パラメータ

| **パラメータ** | **説明**                                                     |
| ------------- | ------------------------------------------------------------ |
| task_name     | タスクの名前。                                               |
| etl_statement | 非同期タスクとして送信したいETLステートメント。現在、StarRocksは[CREATE TABLE AS SELECT](../data-definition/CREATE_TABLE_AS_SELECT.md)や[INSERT](./INSERT.md)の非同期タスクの送信をサポートしています。 |

## 使用上の注意

このステートメントはタスクを作成し、ETLステートメントを実行するタスクのテンプレートを作成します。タスクの情報はメタデータビューの[`情報スキーマ`内の`tasks`](../../../reference/information_schema/tasks.md)をクエリすることで確認できます。

```SQL
SELECT * FROM INFORMATION_SCHEMA.tasks;
SELECT * FROM information_schema.tasks WHERE task_name = '<task_name>';
```

タスクを実行すると、それに応じてTaskRunが生成されます。TaskRunはETLステートメントを実行するタスクを示します。TaskRunには以下の状態があります:

- `PENDING`: タスクが実行を待っています。
- `RUNNING`: タスクが実行中です。
- `FAILED`: タスクが失敗しました。
- `SUCCESS`: タスクが正常に実行されました。

タスクの状態はメタデータビューの[`情報スキーマ`内の`task_runs`](../../../reference/information_schema/task_runs.md)をクエリすることで確認できます。

```SQL
SELECT * FROM INFORMATION_SCHEMA.task_runs;
SELECT * FROM information_schema.task_runs WHERE task_name = '<task_name>';
```

## FE構成アイテムを介して構成する

次のFE構成アイテムを使用して非同期ETLタスクを構成できます:

| **パラメータ**              | **デフォルト値** | **説明**                                                     |
| -------------------------- | ----------------- | ------------------------------------------------------------ |
| task_ttl_second            | 259200            | タスクの有効期間。単位: 秒。有効期間を超えるタスクは削除されます。 |
| task_check_interval_second | 14400             | 無効なタスクを削除するための時間間隔。単位: 秒。         |
| task_runs_ttl_second       | 259200            | TaskRunの有効期間。単位: 秒。有効期間を超えるTaskRunは自動的に削除されます。また、`FAILED`および`SUCCESS`状態のTaskRunも自動的に削除されます。 |
| task_runs_concurrency      | 20                | 並行して実行できるTaskRunの最大数。                        |
| task_runs_queue_length     | 500               | 実行待ちのTaskRunの最大数。デフォルト値を超える場合、新たなタスクは保留されます。 |

## 例

例1: `CREATE TABLE tbl1 AS SELECT * FROM src_tbl`の非同期タスクを送信し、タスク名を`etl0`と指定する。

```SQL
SUBMIT TASK etl0 AS CREATE TABLE tbl1 AS SELECT * FROM src_tbl;
```

例2: `INSERT INTO tbl2 SELECT * FROM src_tbl`の非同期タスクを送信し、タスク名を`etl1`と指定する。

```SQL
SUBMIT TASK etl1 AS INSERT INTO tbl2 SELECT * FROM src_tbl;
```

例3: `INSERT OVERWRITE tbl3 SELECT * FROM src_tbl`の非同期タスクを送信する。

```SQL
SUBMIT TASK AS INSERT OVERWRITE tbl3 SELECT * FROM src_tbl;
```

例4: タスク名を指定せずに`INSERT OVERWRITE insert_wiki_edit SELECT * FROM source_wiki_edit`の非同期タスクを送信し、ヒントを使用してクエリタイムアウトを`100000`秒に延長する。

```SQL
SUBMIT /*+set_var(query_timeout=100000)*/ TASK AS
INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```