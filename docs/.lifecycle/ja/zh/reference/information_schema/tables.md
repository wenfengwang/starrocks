---
displayed_sidebar: Chinese
---

# テーブル

`tables` はテーブルに関する情報を提供します。

`tables` は以下のフィールドを提供します：

| **フィールド**   | **説明**                                                     |
| --------------- | ------------------------------------------------------------ |
| TABLE_CATALOG   | テーブルが属するカタログ名。                                 |
| TABLE_SCHEMA    | テーブルが属するデータベース名。                             |
| TABLE_NAME      | テーブル名。                                                 |
| TABLE_TYPE      | テーブルのタイプ。有効な値：`BASE TABLE` と `VIEW`。        |
| ENGINE          | テーブルのエンジンタイプ。有効な値：`StarRocks`、`MySQL`、`MEMORY`、及び空文字列。 |
| VERSION         | このフィールドは現在使用できません。                         |
| ROW_FORMAT      | このフィールドは現在使用できません。                         |
| TABLE_ROWS      | テーブルの行数。                                             |
| AVG_ROW_LENGTH  | テーブルの平均行長（サイズ）、`DATA_LENGTH`/`TABLE_ROWS` に等しい。単位：Byte。 |
| DATA_LENGTH     | データの長さ（サイズ）。単位：Byte。                         |
| MAX_DATA_LENGTH | このフィールドは現在使用できません。                         |
| INDEX_LENGTH    | このフィールドは現在使用できません。                         |
| DATA_FREE       | このフィールドは現在使用できません。                         |
| AUTO_INCREMENT  | このフィールドは現在使用できません。                         |
| CREATE_TIME     | テーブルを作成した時間。                                     |
| UPDATE_TIME     | テーブルを最後に更新した時間。                               |
| CHECK_TIME      | テーブルの整合性を最後にチェックした時間。                   |
| TABLE_COLLATION | テーブルのデフォルトの照合順序。                             |
| CHECKSUM        | このフィールドは現在使用できません。                         |
| CREATE_OPTIONS  | このフィールドは現在使用できません。                         |
| TABLE_COMMENT   | テーブルのコメント。                                         |
