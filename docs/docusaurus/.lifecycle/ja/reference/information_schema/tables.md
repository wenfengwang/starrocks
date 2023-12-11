---
displayed_sidebar: "Japanese"
---

# テーブル

`tables`はテーブルに関する情報を提供します。

`tables`には次のフィールドが提供されます:

| **フィールド**    | **説明**                                                     |
| --------------- | ------------------------------------------------------------ |
| TABLE_CATALOG   | テーブルを格納するカタログの名前。                              |
| TABLE_SCHEMA    | テーブルを格納するデータベースの名前。                          |
| TABLE_NAME      | テーブルの名前。                                               |
| TABLE_TYPE      | テーブルのタイプ。有効な値: `BASE TABLE`または`VIEW`。           |
| ENGINE          | テーブルのエンジンタイプ。有効な値: `StarRocks`、`MySQL`、`MEMORY`または空の文字列。 |
| VERSION         | StarRocksで利用できない機能に適用されます。                       |
| ROW_FORMAT      | StarRocksで利用できない機能に適用されます。                       |
| TABLE_ROWS      | テーブルの行数。                                                |
| AVG_ROW_LENGTH  | テーブルの平均行長(サイズ)。`DATA_LENGTH`/`TABLE_ROWS`と等価です。単位: バイト。 |
| DATA_LENGTH     | テーブルのデータ長(サイズ)。単位: バイト。                        |
| MAX_DATA_LENGTH | StarRocksで利用できない機能に適用されます。                       |
| INDEX_LENGTH    | StarRocksで利用できない機能に適用されます。                       |
| DATA_FREE       | StarRocksで利用できない機能に適用されます。                       |
| AUTO_INCREMENT  | StarRocksで利用できない機能に適用されます。                       |
| CREATE_TIME     | テーブルが作成された時刻。                                         |
| UPDATE_TIME     | テーブルが最後に更新された時刻。                                     |
| CHECK_TIME      | テーブルの整合性チェックが最後に実行された時刻。                          |
| TABLE_COLLATION | テーブルのデフォルト照合順序。                                      |
| CHECKSUM        | StarRocksで利用できない機能に適用されます。                       |
| CREATE_OPTIONS  | StarRocksで利用できない機能に適用されます。                       |
| TABLE_COMMENT   | テーブルに関するコメント。                                         |