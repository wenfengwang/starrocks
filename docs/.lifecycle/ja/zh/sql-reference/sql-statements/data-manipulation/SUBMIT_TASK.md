---
displayed_sidebar: Chinese
---

# タスクの送信

## 機能

ETLステートメントのための非同期タスクを作成します。この機能はStarRocks 2.5からサポートされています。

StarRocks v3.0では、[CREATE TABLE AS SELECT](../data-definition/CREATE_TABLE_AS_SELECT.md)と[INSERT](./INSERT.md)のための非同期タスクをサポートしています。

非同期タスクは、[DROP TASK](./DROP_TASK.md)を使用して削除できます。

## 文法

```SQL
SUBMIT TASK [task_name] AS <etl_statement>
```

## パラメータ説明

| **パラメータ** | **説明**                                                     |
| --------------- | ------------------------------------------------------------ |
| task_name       | タスク名。                                                   |
| etl_statement   | 非同期タスクを作成するためのETLステートメント。StarRocksは現在、[CREATE TABLE AS SELECT](../data-definition/CREATE_TABLE_AS_SELECT.md)と[INSERT](./INSERT.md)のための非同期タスクをサポートしています。 |

## 使用説明

このステートメントはTaskを作成し、ETLステートメントの実行タスクのストレージテンプレートを表します。Information Schemaのメタデータビュー`tasks`を照会して、Taskの情報を確認できます：

```SQL
SELECT * FROM INFORMATION_SCHEMA.tasks;
SELECT * FROM information_schema.tasks WHERE task_name = '<task_name>';
```

Taskの実行後、TaskRunが生成され、ETLステートメントの実行タスクを表します。TaskRunには以下の状態があります：

- `PENDING`：タスクは実行待ちです。
- `RUNNING`：タスクが実行中です。
- `FAILED`：タスクの実行に失敗しました。
- `SUCCESS`：タスクが成功しました。

Information Schemaのメタデータビュー`task_runs`を照会して、TaskRunの状態を確認できます：

```SQL
SELECT * FROM INFORMATION_SCHEMA.task_runs;
SELECT * FROM information_schema.task_runs WHERE task_name = '<task_name>';
```

## 関連するFEパラメータ

以下のFEパラメータを調整して、非同期ETLタスクを設定できます：

| **パラメータ**               | **デフォルト値** | **説明**                                                     |
| ---------------------------- | ---------------- | ------------------------------------------------------------ |
| task_ttl_second              | 259200           | Taskの有効期限（秒）。有効期限を超えたTaskは自動的に削除されます。 |
| task_check_interval_second   | 14400            | 期限切れのTaskを削除する間隔（秒）。                         |
| task_runs_ttl_second         | 259200           | TaskRunの有効期限（秒）。有効期限を超えたTaskRunは自動的に削除されます。さらに、成功および失敗状態のTaskRunも自動的に削除されます。 |
| task_runs_concurrency        | 20               | 同時に実行できるTaskRunの最大数。                            |
| task_runs_queue_length       | 500              | 実行待ちできるTaskRunの最大数。このパラメータのデフォルト値を超えると、新しいTaskを実行できなくなります。 |

## 例

例1：`CREATE TABLE tbl1 AS SELECT * FROM src_tbl`のための非同期タスクを作成し、`etl0`と名付けます：

```SQL
SUBMIT TASK etl0 AS CREATE TABLE tbl1 AS SELECT * FROM src_tbl;
```

例2：`INSERT INTO tbl2 SELECT * FROM src_tbl`のための非同期タスクを作成し、`etl1`と名付けます：

```SQL
SUBMIT TASK etl1 AS INSERT INTO tbl2 SELECT * FROM src_tbl;
```

例3：`INSERT OVERWRITE tbl3 SELECT * FROM src_tbl`のための非同期タスクを作成します：

```SQL
SUBMIT TASK AS INSERT OVERWRITE tbl3 SELECT * FROM src_tbl;
```

例4：`INSERT OVERWRITE insert_wiki_edit SELECT * FROM source_wiki_edit`のための非同期タスクを作成し、Hintを使用してクエリタイムアウトを`100000`秒に設定します：

```SQL
SUBMIT /*+set_var(query_timeout=100000)*/ TASK AS
INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```
