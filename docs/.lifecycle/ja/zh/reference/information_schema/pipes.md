---
displayed_sidebar: Chinese
---

# pipes

`pipes` は現在のデータベースまたは指定されたデータベースにあるすべての Pipe の詳細情報を提供します。このビューは StarRocks v3.2 以降でサポートされています。

`pipes` は以下のフィールドを提供します：

| **フィールド** | **説明**                                                       |
| -------------- | -------------------------------------------------------------- |
| DATABASE_NAME  | Pipe が属するデータベースの名前です。                          |
| PIPE_ID        | Pipe の一意な ID です。                                        |
| PIPE_NAME      | Pipe の名前です。                                              |
| TABLE_NAME     | StarRocks のターゲットテーブルの名前です。形式：`<database_name>.<table_name>`。 |
| STATE          | Pipe の状態で、`RUNNING`、`FINISHED`、`SUSPENDED`、`ERROR` が含まれます。 |
| LOAD_STATUS    | Pipe における待機中のデータファイルの全体的な状態で、以下のフィールドが含まれます：<br />`loadedFiles`：読み込まれたデータファイルの総数。<br />`loadedBytes`：読み込まれたデータの総量で、単位はバイトです。<br />`loadingFiles`：読み込み中のデータファイルの総数。 |
| LAST_ERROR     | Pipe 実行中に発生した最新のエラーの詳細情報です。デフォルト値は `NULL` です。 |
| CREATED_TIME   | Pipe の作成時間です。形式：`yyyy-MM-dd HH:mm:ss`。例えば、`2023-07-24 14:58:58`。 |
