---
displayed_sidebar: "Japanese"
---

# tables_config

`tables_config`は、テーブルの設定に関する情報を提供します。

`tables_config`には、以下のフィールドが提供されます：

| **フィールド**       | **説明**                                                     |
| ---------------- | ------------------------------------------------------------ |
| TABLE_SCHEMA     | テーブルを格納するデータベースの名前。                                      |
| TABLE_NAME       | テーブルの名前。                                                |
| TABLE_ENGINE     | テーブルのエンジンタイプ。                                          |
| TABLE_MODEL      | テーブルのデータモデル。有効な値：`DUP_KEYS`、`AGG_KEYS`、`UNQ_KEYS`、`PRI_KEYS`。 |
| PRIMARY_KEY      | プライマリキーのテーブルまたはユニークキーのテーブルのプライマリキー。テーブルがプライマリキーのテーブルまたはユニークキーのテーブルでない場合は、空の文字列が返されます。 |
| PARTITION_KEY    | テーブルのパーティショニング列。                                      |
| DISTRIBUTE_KEY   | テーブルのバケット化列。                                           |
| DISTRIBUTE_TYPE  | テーブルのデータ分散方法。                                           |
| DISTRUBTE_BUCKET | テーブルのバケット数。                                               |
| SORT_KEY         | テーブルのソートキー。                                               |
| PROPERTIES       | テーブルのプロパティ。                                               |
| TABLE_ID         | テーブルのID。                                                 |
