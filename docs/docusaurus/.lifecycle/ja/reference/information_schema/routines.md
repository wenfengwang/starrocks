---
displayed_sidebar: "Japanese"
---

# ルーチン

`routines`には、すべての格納されたルーチン（格納された手続きと格納された関数）が含まれています。

`routine`には、次のフィールドが提供されています：

| **フィールド**         | **説明**                                                   |
| -------------------- | ------------------------------------------------------------ |
| SPECIFIC_NAME        | ルーチンの名前。                                       |
| ROUTINE_CATALOG      | ルーチンが属するカタログの名前。この値は常に `def` です。 |
| ROUTINE_SCHEMA       | ルーチンが属するデータベースの名前。                     |
| ROUTINE_NAME         | ルーチンの名前。                                           |
| ROUTINE_TYPE         | 格納された手続きの場合は `PROCEDURE`、格納された関数の場合は `FUNCTION`。 |
| DTD_IDENTIFIER       | ルーチンが格納された関数の場合、返り値のデータ型。ルーチンが格納された手続きの場合、この値は空です。 |
| ROUTINE_BODY         | ルーチン定義に使用される言語。この値は常に `SQL` です。   |
| ROUTINE_DEFINITION   | ルーチンによって実行される SQL ステートメントのテキスト。    |
| EXTERNAL_NAME        | この値は常に `NULL` です。                                |
| EXTERNAL_LANGUAGE    | 格納されたルーチンの言語。                              |
| PARAMETER_STYLE      | この値は常に `SQL` です。                                 |
| IS_DETERMINISTIC     | ルーチンが `DETERMINISTIC` 特性で定義されているかどうかによって `YES` または `NO` があります。 |
| SQL_DATA_ACCESS      | ルーチンのデータアクセス特性。値は `CONTAINS SQL`、`NO SQL`、`READS SQL DATA`、または `MODIFIES SQL DATA` のいずれかです。 |
| SQL_PATH             | この値は常に `NULL` です。                                |
| SECURITY_TYPE        | ルーチンの `SQL SECURITY` 特性。値は `DEFINER` または `INVOKER` のいずれかです。 |
| CREATED              | ルーチンが作成された日時。これは `DATETIME` 値です。       |
| LAST_ALTERED         | ルーチンが最後に変更された日時。これは `DATETIME` 値です。ルーチンが作成以来変更されていない場合、この値は `CREATED` の値と同じです。 |
| SQL_MODE             | ルーチンが作成または変更された際の有効な SQL モード。また、ルーチンが実行される SQL モード。 |
| ROUTINE_COMMENT      | ルーチンにコメントがある場合はそのテキスト。ない場合は空の値です。 |
| DEFINER              | `DEFINER` 句で指定されたユーザー（ほとんどはルーチンを作成したユーザー） |
| CHARACTER_SET_CLIENT |                                                              |
| COLLATION_CONNECTION |                                                              |
| DATABASE_COLLATION   | ルーチンが関連付けられているデータベースの照合順序。       |
