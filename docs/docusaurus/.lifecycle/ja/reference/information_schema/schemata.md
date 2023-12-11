---
displayed_sidebar: "Japanese"
---

# schemata

`schemata`はデータベースに関する情報を提供します。

`schemata`には次のフィールドが提供されています:

| **フィールド**               | **説明**                                                |
| -------------------------- | ------------------------------------------------------------ |
| CATALOG_NAME               | スキーマが所属するカタログの名前。この値は常に`NULL`です。 |
| SCHEMA_NAME                | スキーマの名前。                                            |
| DEFAULT_CHARACTER_SET_NAME | スキーマのデフォルトの文字セット。                           |
| DEFAULT_COLLATION_NAME     | スキーマのデフォルトの照合順序。                            |
| SQL_PATH                   | この値は常に`NULL`です。                                     |