---
displayed_sidebar: "Japanese"
---

# columns

`columns`には、すべてのテーブルの列（またはビューの列）に関する情報が含まれています。

`columns`には、以下のフィールドが提供されます：

| **フィールド**                | **説明**                                              |
| ------------------------ | ------------------------------------------------------------ |
| TABLE_CATALOG            | 列が所属するテーブルのカタログの名前です。この値は常に`NULL`です。 |
| TABLE_SCHEMA             | 列が所属するデータベースの名前です。 |
| TABLE_NAME               | 列が所属するテーブルの名前です。                 |
| COLUMN_NAME              | 列の名前です。                                      |
| ORDINAL_POSITION         | テーブル内の列の序数位置です。         |
| COLUMN_DEFAULT           | 列のデフォルト値です。列が明示的なデフォルト値`NULL`を持つ場合、または列定義に`DEFAULT`句が含まれていない場合、この値は`NULL`です。 |
| IS_NULLABLE              | 列のNULL許容性です。列に`NULL`値を格納できる場合は`YES`、そうでない場合は`NO`です。 |
| DATA_TYPE                | 列のデータ型です。`DATA_TYPE`の値は、型名のみで他の情報は含まれません。`COLUMN_TYPE`の値には、型名と精度や長さなどの他の情報が含まれます。 |
| CHARACTER_MAXIMUM_LENGTH | 文字列列の場合、最大の文字数です。        |
| CHARACTER_OCTET_LENGTH   | 文字列列の場合、最大のバイト数です。             |
| NUMERIC_PRECISION        | 数値列の場合、数値の精度です。                  |
| NUMERIC_SCALE            | 数値列の場合、数値のスケールです。                      |
| DATETIME_PRECISION       | 時系列列の場合、小数秒の精度です。      |
| CHARACTER_SET_NAME       | 文字列列の場合、文字セットの名前です。        |
| COLLATION_NAME           | 文字列列の場合、照合順序の名前です。            |
| COLUMN_TYPE              | 列のデータ型。<br />`DATA_TYPE`の値は、型名のみで他の情報は含まれません。`COLUMN_TYPE`の値には、型名と精度や長さなどの他の情報が含まれます。 |
| COLUMN_KEY               | 列がインデックス化されているかどうか：<ul><li>`COLUMN_KEY`が空の場合、列はインデックス化されていないか、複数列の非一意インデックスの2番目以降の列としてのみインデックス化されています。</li><li>`COLUMN_KEY`が`PRI`の場合、列は`PRIMARY KEY`であるか、複数列の`PRIMARY KEY`の1つです。</li><li>`COLUMN_KEY`が`UNI`の場合、列は`UNIQUE`インデックスの最初の列です。（`UNIQUE`インデックスは複数の`NULL`値を許可しますが、`Null`列をチェックすることで列が`NULL`を許可するかどうかを確認できます。）</li><li>`COLUMN_KEY`が`DUP`の場合、列は非一意インデックスの最初の列であり、列内で特定の値の複数の出現が許可されています。</li></ul>テーブルの特定の列に複数の`COLUMN_KEY`の値が適用される場合、`COLUMN_KEY`は優先度の高い順に`PRI`、`UNI`、`DUP`の順で表示されます。<br />`UNIQUE`インデックスは、`NULL`値を含めることができず、テーブルに`PRIMARY KEY`がない場合、`PRI`として表示される場合があります。複数の列が複合`UNIQUE`インデックスを形成する場合、`UNIQUE`インデックスは`MUL`として表示される場合があります。組み合わせは一意ですが、各列は特定の値の複数の出現を保持できます。 |
| EXTRA                    | 特定の列に関する追加情報です。 |
| PRIVILEGES               | 列に対する権限です。                      |
| COLUMN_COMMENT           | 列定義に含まれるコメントです。               |
| COLUMN_SIZE              |                                                              |
| DECIMAL_DIGITS           |                                                              |
| GENERATION_EXPRESSION    | 生成列の場合、列の値を計算するために使用される式が表示されます。非生成列の場合は空です。 |
| SRS_ID                   | 空間列に適用される値です。列に格納されている値の空間参照システムを示す列`SRID`の値を含みます。 |
