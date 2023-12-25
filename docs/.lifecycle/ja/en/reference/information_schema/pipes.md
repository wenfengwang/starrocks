---
displayed_sidebar: English
---

# pipes

`pipes` は、現在のデータベースまたは指定されたデータベースに保存されているすべてのパイプに関する情報を提供します。このビューはStarRocks v3.2以降でサポートされています。

`pipes` には以下のフィールドが含まれています：

| **フィールド** | **説明**                                                      |
| --------------- | ------------------------------------------------------------- |
| DATABASE_NAME   | パイプが保存されているデータベースの名前。                    |
| PIPE_ID         | パイプのユニークなID。                                        |
| PIPE_NAME       | パイプの名前。                                                |
| TABLE_NAME      | 宛先テーブルの名前。フォーマット: `<database_name>.<table_name>`。 |
| STATE           | パイプの状態。有効な値: `RUNNING`, `FINISHED`, `SUSPENDED`, `ERROR`。 |
| LOAD_STATUS     | パイプを介してロードされるデータファイルの全体的な状態。以下のサブフィールドが含まれます：<ul><li>`loadedFiles`：ロードされたデータファイルの数。</li><li>`loadedBytes`：ロードされたデータの量、バイト単位。</li><li>`loadingFiles`：ロード中のデータファイルの数。</li></ul> |
| LAST_ERROR      | パイプ実行中に発生した最後のエラーの詳細。デフォルト値: `NULL`。 |
| CREATED_TIME    | パイプが作成された日付と時刻。フォーマット: `yyyy-MM-dd HH:mm:ss`。例: `2023-07-24 14:58:58`。 |
