---
displayed_sidebar: Chinese
---

# スキーマ

`schemata` はデータベースに関する情報を提供します。

`schemata` は以下のフィールドを提供します：

| フィールド                  | 説明                                      |
| --------------------------- | ----------------------------------------- |
| CATALOG_NAME                | スキーマが属するカタログの名前。常に NULL。 |
| SCHEMA_NAME                 | スキーマの名前。                          |
| DEFAULT_CHARACTER_SET_NAME  | スキーマのデフォルト文字セット。          |
| DEFAULT_COLLATION_NAME      | スキーマのデフォルト照合順序。            |
| SQL_PATH                    | この値は常に NULL です。                  |
