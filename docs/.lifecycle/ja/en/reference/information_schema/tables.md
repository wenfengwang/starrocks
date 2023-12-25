---
displayed_sidebar: English
---

# テーブル

`tables` はテーブルに関する情報を提供します。

`tables` には以下のフィールドが提供されています：

| **フィールド**       | **説明**                                              |
| --------------- | ------------------------------------------------------------ |
| TABLE_CATALOG   | テーブルを格納しているカタログの名前。                   |
| TABLE_SCHEMA    | テーブルを格納しているデータベースの名前。                  |
| TABLE_NAME      | テーブルの名前。                                           |
| TABLE_TYPE      | テーブルのタイプ。有効な値は `BASE TABLE` または `VIEW`。     |
| ENGINE          | テーブルのエンジンタイプ。有効な値は `StarRocks`、`MySQL`、`MEMORY`、または空文字列。 |
| VERSION         | StarRocksでは利用できない機能に適用されます。             |
| ROW_FORMAT      | StarRocksでは利用できない機能に適用されます。             |
| TABLE_ROWS      | テーブルの行数。                                      |
| AVG_ROW_LENGTH  | テーブルの平均行長（サイズ）。`DATA_LENGTH` ÷ `TABLE_ROWS` に相当します。単位はバイト。|
| DATA_LENGTH     | テーブルのデータ長（サイズ）。単位はバイト。                 |
| MAX_DATA_LENGTH | StarRocksでは利用できない機能に適用されます。             |
| INDEX_LENGTH    | StarRocksでは利用できない機能に適用されます。             |
| DATA_FREE       | StarRocksでは利用できない機能に適用されます。             |
| AUTO_INCREMENT  | StarRocksでは利用できない機能に適用されます。             |
| CREATE_TIME     | テーブルが作成された時刻。                          |
| UPDATE_TIME     | テーブルが最後に更新された時刻。                     |
| CHECK_TIME      | テーブルの整合性チェックが最後に実行された時刻。 |
| TABLE_COLLATION | テーブルのデフォルト照合順序。                          |
| CHECKSUM        | StarRocksでは利用できない機能に適用されます。             |
| CREATE_OPTIONS  | StarRocksでは利用できない機能に適用されます。             |
| TABLE_COMMENT   | テーブルに関するコメント。                                        |
