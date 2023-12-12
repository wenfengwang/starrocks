---
displayed_sidebar: "Japanese"
---

# tables_config

`tables_config`はテーブルの構成に関する情報を提供します。

`tables_config`には、以下のフィールドが提供されます：

| **フィールド**    | **説明**                                                     |
| ----------------- | ------------------------------------------------------------ |
| TABLE_SCHEMA      | テーブルを格納しているデータベースの名前。                  |
| TABLE_NAME        | テーブルの名前。                                              |
| TABLE_ENGINE      | テーブルのエンジンタイプ。                                    |
| TABLE_MODEL       | テーブルのデータモデル。有効な値：`DUP_KEYS`、`AGG_KEYS`、`UNQ_KEYS`、`PRI_KEYS`。 |
| PRIMARY_KEY       | プライマリキーテーブルまたはユニークキーテーブルのプライマリキー。テーブルがプライマリキーテーブルまたはユニークキーテーブルでない場合は空の文字列が返されます。 |
| PARTITION_KEY     | テーブルのパーティション列。                                 |
| DISTRIBUTE_KEY    | テーブルのバケティング列。                                    |
| DISTRIBUTE_TYPE   | テーブルのデータ分散方法。                                    |
| DISTRUBTE_BUCKET  | テーブル内のバケット数。                                     |
| SORT_KEY          | テーブルのソートキー。                                        |
| PROPERTIES        | テーブルのプロパティ。                                         |
| TABLE_ID          | テーブルのID。                                                |