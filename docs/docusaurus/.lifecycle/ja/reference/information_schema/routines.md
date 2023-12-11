---
displayed_sidebar: "Japanese"
---

# ルーチン

`routines`には、すべてのストアドルーチン（ストアドプロシージャとストアドファンクション）が含まれています。

`routine`には、次のフィールドが提供されています:

| **フィールド**         | **説明**                                                     |
| --------------------- | ------------------------------------------------------------ |
| SPECIFIC_NAME         | ルーチンの名前。                                              |
| ROUTINE_CATALOG       | ルーチンが属するカタログの名前。この値は常に `def` です。  |
| ROUTINE_SCHEMA        | ルーチンが所属するデータベースの名前。                       |
| ROUTINE_NAME          | ルーチンの名前。                                              |
| ROUTINE_TYPE          | ストアドプロシージャの場合は `PROCEDURE`、ストアドファンクションの場合は `FUNCTION`。 |
| DTD_IDENTIFIER        | ルーチンがストアドファンクションの場合は戻り値のデータ型。ストアドプロシージャの場合は、この値は空です。 |
| ROUTINE_BODY          | ルーチン定義に使用される言語。この値は常に `SQL` です。     |
| ROUTINE_DEFINITION    | ルーチンによって実行されるSQLステートメントのテキスト。     |
| EXTERNAL_NAME         | この値は常に `NULL` です。                                  |
| EXTERNAL_LANGUAGE     | ストアドルーチンの言語。                                    |
| PARAMETER_STYLE       | この値は常に `SQL` です。                                  |
| IS_DETERMINISTIC      | ルーチンが `DETERMINISTIC` 特性で定義されているかどうかに応じて、`YES` または `NO`。 |
| SQL_DATA_ACCESS       | ルーチンのデータアクセス特性。値は、`CONTAINS SQL`、`NO SQL`、`READS SQL DATA`、または `MODIFIES SQL DATA` のいずれかです。 |
| SQL_PATH              | この値は常に `NULL` です。                                  |
| SECURITY_TYPE         | ルーチンの `SQL SECURITY` 特性。値は `DEFINER` または `INVOKER` のいずれかです。 |
| CREATED               | ルーティンが作成された日時。`DATETIME` の値です。             |
| LAST_ALTERED          | ルーチンが最後に変更された日時。 `DATETIME` の値です。ルーチンが作成されてから変更されていない場合、この値は `CREATED` の値と同じです。 |
| SQL_MODE              | ルーチンが作成または変更されたときに有効だったSQLモード、およびルーチンが実行される際のSQLモード。 |
| ROUTINE_COMMENT       | ルーチンにコメントがある場合はそのテキスト。ない場合は空の値です。 |
| DEFINER               | `DEFINER` 句に記載されたユーザー（通常はルーチンを作成したユーザー）。 |
| CHARACTER_SET_CLIENT  |                                                              |
| COLLATION_CONNECTION  |                                                              |
| DATABASE_COLLATION    | ルーチンが関連付けられているデータベースの照合順序。         |