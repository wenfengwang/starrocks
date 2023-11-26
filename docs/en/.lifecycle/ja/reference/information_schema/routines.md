---
displayed_sidebar: "Japanese"
---

# ルーチン

`routines`には、すべてのストアドルーチン（ストアドプロシージャとストアド関数）が含まれています。

`routine`には、以下のフィールドが提供されます：

| **フィールド**           | **説明**                                                     |
| ----------------------- | ------------------------------------------------------------ |
| SPECIFIC_NAME           | ルーチンの名前。                                              |
| ROUTINE_CATALOG         | ルーチンが所属するカタログの名前。この値は常に `def` です。     |
| ROUTINE_SCHEMA          | ルーチンが所属するデータベースの名前。                          |
| ROUTINE_NAME            | ルーチンの名前。                                              |
| ROUTINE_TYPE            | ストアドプロシージャの場合は `PROCEDURE`、ストアド関数の場合は `FUNCTION`。 |
| DTD_IDENTIFIER          | ルーチンがストアド関数の場合、戻り値のデータ型。ルーチンがストアドプロシージャの場合、この値は空です。 |
| ROUTINE_BODY            | ルーチンの定義に使用される言語。この値は常に `SQL` です。       |
| ROUTINE_DEFINITION      | ルーチンによって実行されるSQLステートメントのテキスト。        |
| EXTERNAL_NAME           | この値は常に `NULL` です。                                    |
| EXTERNAL_LANGUAGE       | ストアドルーチンの言語。                                      |
| PARAMETER_STYLE         | この値は常に `SQL` です。                                     |
| IS_DETERMINISTIC        | ルーチンが `DETERMINISTIC` 特性で定義されているかどうかに応じて、`YES` または `NO` です。 |
| SQL_DATA_ACCESS         | ルーチンのデータアクセス特性。値は `CONTAINS SQL`、`NO SQL`、`READS SQL DATA`、または `MODIFIES SQL DATA` のいずれかです。 |
| SQL_PATH                | この値は常に `NULL` です。                                    |
| SECURITY_TYPE           | ルーチンの `SQL SECURITY` 特性。値は `DEFINER` または `INVOKER` のいずれかです。 |
| CREATED                 | ルーチンが作成された日時。これは `DATETIME` の値です。          |
| LAST_ALTERED            | ルーチンが最後に変更された日時。これは `DATETIME` の値です。ルーチンが作成後に変更されていない場合、この値は `CREATED` の値と同じです。 |
| SQL_MODE                | ルーチンが作成または変更されたときに有効なSQLモード。ルーチンが実行されるSQLモードです。 |
| ROUTINE_COMMENT         | ルーチンにコメントがある場合はコメントのテキスト。コメントがない場合、この値は空です。 |
| DEFINER                 | `DEFINER` 句で指定されたユーザー（通常はルーチンを作成したユーザー）です。 |
| CHARACTER_SET_CLIENT    |                                                              |
| COLLATION_CONNECTION    |                                                              |
| DATABASE_COLLATION      | ルーチンが関連付けられているデータベースの照合順序。             |
