---
displayed_sidebar: Chinese
---

# ルーチン

`routines` には、すべてのストアドプロシージャ（Routine）が含まれており、プロシージャと関数が含まれます。

`routine` は以下のフィールドを提供します：

| フィールド             | 説明                                                         |
| ---------------------- | ------------------------------------------------------------ |
| SPECIFIC_NAME          | プロシージャの名前です。                                     |
| ROUTINE_CATALOG        | プロシージャが属するカタログの名前。この値は常に def です。 |
| ROUTINE_SCHEMA         | プロシージャが属するデータベースの名前。                     |
| ROUTINE_NAME           | プロシージャの名前です。                                     |
| ROUTINE_TYPE           | ストアドプロシージャのタイプは PROCEDURE、ストアドファンクションのタイプは FUNCTION です。 |
| DTD_IDENTIFIER         | プロシージャがストアドファンクションの場合は戻り値のデータタイプ、ストアドプロシージャの場合はこの値は空です。 |
| ROUTINE_BODY           | プロシージャの定義に使用される言語。この値は常に SQL です。 |
| ROUTINE_DEFINITION     | プロシージャが実行する SQL ステートメントのテキスト。        |
| EXTERNAL_NAME          | この値は常に NULL です。                                      |
| EXTERNAL_LANGUAGE      | ストアドプロシージャの言語。                                 |
| PARAMETER_STYLE        | この値は常に SQL です。                                       |
| IS_DETERMINISTIC       | プロシージャが DETERMINISTIC 属性を使用して定義されているかどうかに依存します。YES または NO のいずれかです。 |
| SQL_DATA_ACCESS        | プロシージャのデータアクセス特性。この値は CONTAINS SQL、NO SQL、READS SQL DATA、または MODIFIES SQL DATA のいずれかです。 |
| SQL_PATH               | この値は常に NULL です。                                      |
| SECURITY_TYPE          | プロシージャの SQL SECURITY 特性。この値は DEFINER または INVOKER のいずれかです。 |
| CREATED                | プロシージャが作成された日付と時刻。これは DATETIME 値です。 |
| LAST_ALTERED           | プロシージャが最後に変更された日付と時刻。これは DATETIME 値です。プロシージャが作成されてから変更されていない場合、この値は CREATED 値と同じです。 |
| SQL_MODE               | プロシージャが作成または変更されたときに有効だった SQL モード、およびプロシージャが実行される SQL モード。 |
| ROUTINE_COMMENT        | プロシージャのコメントテキスト。コメントがある場合のみ。ない場合はこの値は空です。 |
| DEFINER                | DEFINER 句で指定されたユーザー（通常はプロシージャを作成したユーザー）。 |
| CHARACTER_SET_CLIENT   |                                                              |
| COLLATION_CONNECTION   |                                                              |
| DATABASE_COLLATION     | プロシージャが関連するデータベースの照合順序。               |
