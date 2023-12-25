---
displayed_sidebar: Chinese
---

# pipe_files

`pipe_files` は指定された Pipe のデータファイルのインポート状態を提供します。このビューは StarRocks v3.2 からサポートされています。

`pipe_files` は以下のフィールドを提供します：

| **フィールド**   | **説明**                                                     |
| ---------------- | ------------------------------------------------------------ |
| DATABASE_NAME    | Pipe が属するデータベースの名前です。                        |
| PIPE_ID          | Pipe の一意な ID です。                                      |
| PIPE_NAME        | Pipe の名前です。                                            |
| FILE_NAME        | データファイルの名前です。                                   |
| FILE_VERSION     | データファイルの内容の要約値です。                           |
| FILE_SIZE        | データファイルのサイズです。単位：バイト。                   |
| LAST_MODIFIED    | データファイルの最終変更時間です。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| LOAD_STATE       | データファイルのインポート状態で、`UNLOADED`、`LOADING`、`FINISHED`、`ERROR` を含みます。 |
| STAGED_TIME      | データファイルが Pipe に初めて記録された時間です。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| START_LOAD_TIME  | データファイルのインポートが開始された時間です。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| FINISH_LOAD_TIME | データファイルのインポートが終了した時間です。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| ERROR_MSG        | データファイルのインポートエラーメッセージです。           |
