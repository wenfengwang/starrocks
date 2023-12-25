---
displayed_sidebar: Chinese
---

# tables_config

`tables_config` はテーブル設定に関する情報を提供します。

`tables_config` は以下のフィールドを提供します：

| **フィールド**   | **説明**                                                     |
| ---------------- | ------------------------------------------------------------ |
| TABLE_SCHEMA     | テーブルが属するデータベースの名前。                         |
| TABLE_NAME       | テーブル名。                                                 |
| TABLE_ENGINE     | テーブルのエンジンタイプ。                                   |
| TABLE_MODEL      | テーブルのデータモデル。有効な値：`DUP_KEYS`、`AGG_KEYS`、`UNQ_KEYS`、`PRI_KEYS`。|
| PRIMARY_KEY      | 主キーモデルまたは更新モデルテーブルの主キー。該当しない場合は空文字列を返します。 |
| PARTITION_KEY    | テーブルのパーティションキー。                               |
| DISTRIBUTE_KEY   | テーブルの分散キー。                                         |
| DISTRIBUTE_TYPE  | テーブルの分散タイプ。                                       |
| DISTRIBUTE_BUCKET| テーブルのバケット数。                                       |
| SORT_KEY         | テーブルのソートキー。                                       |
| PROPERTIES       | テーブルのプロパティ。                                       |
| TABLE_ID         | テーブルのID。                                               |
