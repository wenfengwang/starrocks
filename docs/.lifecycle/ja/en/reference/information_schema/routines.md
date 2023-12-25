---
displayed_sidebar: English
---

# ルーチン

`routines` には、すべてのストアドルーチン（ストアドプロシージャとストアドファンクション）が含まれています。

`routine` には、以下のフィールドが提供されています。

| **フィールド**            | **説明**                                              |
| -------------------- | ------------------------------------------------------------ |
| SPECIFIC_NAME        | ルーチンの名前です。                                     |
| ROUTINE_CATALOG      | ルーチンが属するカタログの名前です。この値は常に `def` です。 |
| ROUTINE_SCHEMA       | ルーチンが属するデータベースの名前です。       |
| ROUTINE_NAME         | ルーチンの名前です。                                     |
| ROUTINE_TYPE         | ストアドプロシージャの場合は `PROCEDURE`、ストアドファンクションの場合は `FUNCTION` です。 |
| DTD_IDENTIFIER       | ルーチンがストアドファンクションである場合、戻り値のデータ型です。ストアドプロシージャの場合、この値は空です。 |
| ROUTINE_BODY         | ルーチン定義に使用される言語です。この値は常に `SQL` です。 |
| ROUTINE_DEFINITION   | ルーチンによって実行されるSQLステートメントのテキストです。       |
| EXTERNAL_NAME        | この値は常に `NULL` です。                                 |
| EXTERNAL_LANGUAGE    | ストアドルーチンの言語です。                          |
| PARAMETER_STYLE      | この値は常に `SQL` です。                                  |
| IS_DETERMINISTIC     | ルーチンが `DETERMINISTIC` 特性で定義されているかどうかにより `YES` または `NO` です。 |
| SQL_DATA_ACCESS      | ルーチンのデータアクセス特性です。値は `CONTAINS SQL`、`NO SQL`、`READS SQL DATA`、または `MODIFIES SQL DATA` のいずれかです。 |
| SQL_PATH             | この値は常に `NULL` です。                                 |
| SECURITY_TYPE        | ルーチンの `SQL SECURITY` 特性です。値は `DEFINER` または `INVOKER` のいずれかです。 |
| CREATED              | ルーチンが作成された日時です。これは `DATETIME` 値です。 |
| LAST_ALTERED         | ルーチンが最後に変更された日時です。これは `DATETIME` 値です。ルーチンが作成後に変更されていない場合、この値は `CREATED` 値と同じです。 |
| SQL_MODE             | ルーチンが作成または変更されたときに有効だったSQLモードで、ルーチンが実行されます。 |
| ROUTINE_COMMENT      | コメントのテキストです（ルーチンにコメントがある場合）。ない場合、この値は空です。 |
| DEFINER              | `DEFINER` 句で指定されたユーザーです（多くの場合、ルーチンを作成したユーザー）。 |
| CHARACTER_SET_CLIENT |                                                              |
| COLLATION_CONNECTION |                                                              |
| DATABASE_COLLATION   | ルーチンが関連付けられているデータベースの照合順序です。 |