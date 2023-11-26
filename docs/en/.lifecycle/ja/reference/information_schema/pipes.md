---
displayed_sidebar: "Japanese"
---

# pipes

`pipes`は、現在のまたは指定されたデータベースに保存されているすべてのパイプに関する情報を提供します。このビューは、StarRocks v3.2以降でサポートされています。

`pipes`には、次のフィールドが提供されます：

| **フィールド**   | **説明**                                                     |
| --------------- | ------------------------------------------------------------ |
| DATABASE_NAME   | パイプが保存されているデータベースの名前です。                 |
| PIPE_ID         | パイプの固有のIDです。                                        |
| PIPE_NAME       | パイプの名前です。                                            |
| TABLE_NAME      | 宛先テーブルの名前です。フォーマット：`<database_name>.<table_name>`。 |
| STATE           | パイプの状態です。有効な値は、`RUNNING`、`FINISHED`、`SUSPENDED`、`ERROR`です。 |
| LOAD_STATUS     | パイプを介してロードされるデータファイルの全体的なステータスです。次のサブフィールドを含みます：<ul><li>`loadedFiles`：ロードされたデータファイルの数。</li><li>`loadedBytes`：バイト単位で測定されたロードされたデータのボリューム。</li><li>`loadingFiles`：ロード中のデータファイルの数。</li></ul> |
| LAST_ERROR      | パイプの実行中に発生した最後のエラーに関する詳細です。デフォルト値：`NULL`。 |
| CREATED_TIME    | パイプが作成された日時です。フォーマット：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
