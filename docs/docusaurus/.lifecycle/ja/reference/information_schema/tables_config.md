---
displayed_sidebar: "Japanese"
---

# tables_config

`tables_config`はテーブルの構成に関する情報を提供します。

`tables_config`には以下のフィールドが提供されています：

| **フィールド**      | **説明**                                                    |
| ---------------- | ------------------------------------------------------------ |
| TABLE_SCHEMA     | テーブルを格納するデータベースの名前。                             |
| TABLE_NAME       | テーブルの名前。                                           |
| TABLE_ENGINE     | テーブルのエンジン種別。                                    |
| TABLE_MODEL      | テーブルのデータモデル。有効な値：`DUP_KEYS`、`AGG_KEYS`、`UNQ_KEYS`、`PRI_KEYS`。 |
| PRIMARY_KEY      | プライマリキーまたはユニークキーテーブルのプライマリキー。テーブルがプライマリキーまたはユニークキーテーブルでない場合は空の文字列が返されます。 |
| PARTITION_KEY    | テーブルのパーティショニング列。                       |
| DISTRIBUTE_KEY   | テーブルのバケット列。                          |
| DISTRIBUTE_TYPE  | テーブルのデータ分散方法。                   |
| DISTRUBTE_BUCKET | テーブル内のバケットの数。                              |
| SORT_KEY         | テーブルのソートキー。                                      |
| PROPERTIES       | テーブルのプロパティ。                                     |
| TABLE_ID         | テーブルのID。                                             |