---
displayed_sidebar: English
---

# tables_config

`tables_config` はテーブルの設定情報を提供します。

`tables_config` には以下のフィールドが含まれます：

| **フィールド**   | **説明**                                                      |
| ---------------- | ------------------------------------------------------------ |
| TABLE_SCHEMA     | テーブルを格納しているデータベースの名前。                  |
| TABLE_NAME       | テーブルの名前。                                           |
| TABLE_ENGINE     | テーブルのエンジンタイプ。                                    |
| TABLE_MODEL      | テーブルのデータモデル。有効な値は `DUP_KEYS`、`AGG_KEYS`、`UNQ_KEYS`、または `PRI_KEYS`。 |
| PRIMARY_KEY      | プライマリキーテーブルまたはユニークキーテーブルのプライマリキー。テーブルがプライマリキーテーブルまたはユニークキーテーブルでない場合、空文字列が返されます。 |
| PARTITION_KEY    | テーブルのパーティションキー。                       |
| DISTRIBUTE_KEY   | テーブルのディストリビュートキー。                          |
| DISTRIBUTE_TYPE  | テーブルのデータ分散タイプ。                   |
| DISTRUBTE_BUCKET | テーブルのバケット数。                              |
| SORT_KEY         | テーブルのソートキー。                                      |
| PROPERTIES       | テーブルのプロパティ。                                     |
| TABLE_ID         | テーブルのID。                                             |
