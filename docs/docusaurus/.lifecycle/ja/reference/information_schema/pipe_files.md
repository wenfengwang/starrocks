---
displayed_sidebar: "Japanese"
---

# pipe_files

`pipe_files`は、指定されたパイプを介してロードされるデータファイルのステータスを提供します。このビューは、StarRocks v3.2以降でサポートされています。

次のフィールドが`pipe_files`で提供されます:

| **フィールド**    | **説明**                                                    |
| ---------------- | ------------------------------------------------------------ |
| DATABASE_NAME    | パイプが格納されているデータベースの名前。                 |
| PIPE_ID          | パイプの固有のID。                                           |
| PIPE_NAME        | パイプの名前。                                                |
| FILE_NAME        | データファイルの名前。                                       |
| FILE_VERSION     | データファイルのダイジェスト。                                |
| FILE_SIZE        | データファイルのサイズ。単位: バイト。                       |
| LAST_MODIFIED    | データファイルが最後に変更された時刻。形式: `yyyy-MM-dd HH:mm:ss`。例: `2023-07-24 14:58:58`。 |
| LOAD_STATE       | データファイルのロード状態。有効な値: `UNLOADED`, `LOADING`, `FINISHED`, `ERROR`。 |
| STAGED_TIME      | データファイルがパイプによって最初に記録された日時。形式: `yyyy-MM-dd HH:mm:ss`。例: `2023-07-24 14:58:58`。 |
| START_LOAD_TIME  | データファイルのロードが開始された日時。形式: `yyyy-MM-dd HH:mm:ss`。例: `2023-07-24 14:58:58`。 |
| FINISH_LOAD_TIME | データファイルのロードが終了した日時。形式: `yyyy-MM-dd HH:mm:ss`。例: `2023-07-24 14:58:58`。 |
| ERROR_MSG        | データファイルのロードエラーに関する詳細。                 |