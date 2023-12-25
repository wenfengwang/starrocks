---
displayed_sidebar: Chinese
---

# パーティション

`partitions` はテーブルのパーティションに関する情報を提供します。

`partitions` は以下のフィールドを提供します：

| フィールド                  | 説明                                                         |
| -------------------------- | ------------------------------------------------------------ |
| TABLE_CATALOG              | テーブルが属するカタログの名前。この値は常に def です。     |
| TABLE_SCHEMA               | テーブルが属するデータベースの名前。                         |
| TABLE_NAME                 | パーティションが含まれるテーブルの名前。                     |
| PARTITION_NAME             | パーティションの名前。                                       |
| SUBPARTITION_NAME          | PARTITIONS テーブルの行がサブパーティションを表す場合はサブパーティションの名前、そうでなければ NULL。NDB については、この値は常に NULL です。 |
| PARTITION_ORDINAL_POSITION | パーティションは定義された順序でインデックスされ、1 は最初のパーティションに割り当てられた数字です。パーティションの追加、削除、再編成によりインデックスは変更される可能性があります。この列に表示される数字は、任意のインデックス変更を考慮した現在の順序を反映します。 |
| PARTITION_METHOD           | 有効な値：RANGE、LIST、HASH、LINEAR HASH、KEY、LINEAR KEY。 |
| SUBPARTITION_METHOD        | 有効な値：HASH、LINEAR HASH、KEY、LINEAR KEY。               |
| PARTITION_EXPRESSION       | CREATE TABLE または ALTER TABLE ステートメントでテーブルの現在のパーティションスキームを作成するために使用されるパーティション関数の式。 |
| SUBPARTITION_EXPRESSION    | サブパーティション式は、テーブルのパーティションを定義するための PARTITION_EXPRESSION と同じ機能を持ちます。テーブルにサブパーティションがない場合、この列は NULL です。 |
| PARTITION_DESCRIPTION      | この列は RANGE および LIST パーティションに使用されます。RANGE パーティションの場合、それはパーティションの VALUES LESS THAN 句で設定された値を含み、整数または MAXVALUE が可能です。LIST パーティションの場合、この列はパーティションの VALUES IN 句で定義された値、つまりコンマで区切られた整数値のリストを含みます。PARTITION_METHOD が RANGE または LIST でないパーティションの場合、この列は常に NULL です。 |
| TABLE_ROWS                 | パーティション内のテーブル行数。                             |
| AVG_ROW_LENGTH             | このパーティションまたはサブパーティションに格納されている行の平均長さ（バイト単位）。これは DATA_LENGTH を TABLE_ROWS で割ったものと同じです。 |
| DATA_LENGTH                | このパーティションまたはサブパーティションに格納されているすべての行の合計長さ（バイト単位）、つまりパーティションまたはサブパーティションに格納されているバイトの総数。 |
| MAX_DATA_LENGTH            | このパーティションまたはサブパーティションに格納できる最大バイト数。 |
| INDEX_LENGTH               | このパーティションまたはサブパーティションのインデックスファイルの長さ（バイト単位）。 |
| DATA_FREE                  | パーティションまたはサブパーティションに割り当てられているが使用されていないバイト数。 |
| CREATE_TIME                | パーティションまたはサブパーティションが作成された時間。     |
| UPDATE_TIME                | パーティションまたはサブパーティションが最後に変更された時間。 |
| CHECK_TIME                 | このパーティションまたはサブパーティションに属するテーブルが最後にチェックされた時間。 |
| CHECKSUM                   | チェックサム値、存在する場合。そうでなければ NULL。          |
| PARTITION_COMMENT          | パーティションのコメントテキスト、存在する場合。そうでなければこの値は空です。 |
| NODEGROUP                  | パーティションが属するノードグループ。                       |
| TABLESPACE_NAME            | パーティションが属するテーブルスペースの名前。               |
