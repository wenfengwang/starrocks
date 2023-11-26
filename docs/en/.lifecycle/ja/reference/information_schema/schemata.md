---
displayed_sidebar: "Japanese"
---

# スキーマ

`schemata`はデータベースに関する情報を提供します。

`schemata`には以下のフィールドが提供されます：

| **フィールド**                  | **説明**                                              |
| -------------------------- | ------------------------------------------------------------ |
| CATALOG_NAME               | スキーマが所属するカタログの名前です。この値は常に`NULL`です。 |
| SCHEMA_NAME                | スキーマの名前です。                                      |
| DEFAULT_CHARACTER_SET_NAME | スキーマのデフォルトの文字セットです。                            |
| DEFAULT_COLLATION_NAME     | スキーマのデフォルトの照合順序です。                                |
| SQL_PATH                   | この値は常に`NULL`です。                                 |
