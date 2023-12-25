---
displayed_sidebar: English
---

# pipe_files

`pipe_files` は、指定されたパイプを介してロードされるデータファイルのステータスを提供します。このビューは StarRocks v3.2 以降でサポートされています。

`pipe_files` には以下のフィールドが提供されています：

| **フィールド**        | **説明**                                              |
| ---------------- | ------------------------------------------------------------ |
| DATABASE_NAME    | パイプが格納されているデータベースの名前です。        |
| PIPE_ID          | パイプの一意のIDです。                                   |
| PIPE_NAME        | パイプの名前です。                                        |
| FILE_NAME        | データファイルの名前です。                                   |
| FILE_VERSION     | データファイルのダイジェストです。                                 |
| FILE_SIZE        | データファイルのサイズ。単位はバイトです。                      |
| LAST_MODIFIED    | データファイルが最後に変更された時刻。形式: `yyyy-MM-dd HH:mm:ss`。例: `2023-07-24 14:58:58`。 |
| LOAD_STATE       | データファイルのロード状態。有効な値は `UNLOADED`、`LOADING`、`FINISHED`、`ERROR` です。 |
| STAGED_TIME      | データファイルがパイプによって最初に記録された日時。形式: `yyyy-MM-dd HH:mm:ss`。例: `2023-07-24 14:58:58`。 |
| START_LOAD_TIME  | データファイルのロードが開始された日時。形式: `yyyy-MM-dd HH:mm:ss`。例: `2023-07-24 14:58:58`。 |
| FINISH_LOAD_TIME | データファイルのロードが完了した日時。形式: `yyyy-MM-dd HH:mm:ss`。例: `2023-07-24 14:58:58`。 |
| ERROR_MSG        | データファイルのロードエラーに関する詳細です。          |
