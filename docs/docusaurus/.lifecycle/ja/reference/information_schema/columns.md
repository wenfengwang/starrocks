---
displayed_sidebar: "Japanese"
---

# columns

`columns`には、すべてのテーブル列（またはビュー列）の情報が含まれています。

`columns`には、次のフィールドが提供されています:

| **Field**                | **Description**                                              |
| ------------------------ | ------------------------------------------------------------ |
| TABLE_CATALOG            | 列を含むテーブルが属するカタログの名前。この値は常に`NULL`です。 |
| TABLE_SCHEMA             | 列を含むテーブルが属するデータベースの名前。                |
| TABLE_NAME               | 列を含むテーブルの名前。                                    |
| COLUMN_NAME              | 列の名前。                                                  |
| ORDINAL_POSITION         | テーブル内の列の序数位置。                                  |
| COLUMN_DEFAULT           | 列のデフォルト値。列が明示的なデフォルト値`NULL`を持つか、列定義に`DEFAULT`句が含まれない場合は`NULL`です。 |
| IS_NULLABLE              | 列のNULL可能性。列に`NULL`値を格納できる場合は`YES`、できない場合は`NO`です。 |
| DATA_TYPE                | 列のデータ型。`DATA_TYPE`の値は型名のみで、他の情報は含まれません。`COLUMN_TYPE`の値には、型名や精度、長さなどの他の情報が含まれる場合があります。 |
| CHARACTER_MAXIMUM_LENGTH | 文字列列の場合、文字数の最大長。                           |
| CHARACTER_OCTET_LENGTH   | 文字列列の場合、バイト数の最大長。                         |
| NUMERIC_PRECISION        | 数値列の場合、数値の精度。                                 |
| NUMERIC_SCALE            | 数値列の場合、数値のスケール。                             |
| DATETIME_PRECISION       | 時刻列の場合、小数秒の精度。                               |
| CHARACTER_SET_NAME       | 文字列列の場合、文字セット名。                             |
| COLLATION_NAME           | 文字列列の場合、照合順序名。                               |
| COLUMN_TYPE              | 列のデータ型。<br />`DATA_TYPE`の値は型名のみで、他の情報は含まれません。`COLUMN_TYPE`の値には、型名や精度、長さなどの他の情報が含まれる場合があります。 |
| COLUMN_KEY               | 列がインデックス化されているかどうか:<ul><li>`COLUMN_KEY`が空の場合、列はインデックス化されていないか、複数列の非ユニークインデックスの2番目以降の列としてのみインデックス化されています。</li><li>`COLUMN_KEY`が`PRI`の場合、列は`主キー`であるか、複数列の`主キー`の1つであるかのいずれかです。</li><li>`COLUMN_KEY`が`UNI`の場合、列は`UNIQUE`インデックスの最初の列です（`UNIQUE`インデックスには複数の`NULL`値が許可されますが、列が`NULL`を許可するかどうかは`Null`列をチェックすることでわかります）。</li><li>`COLUMN_KEY`が`DUP`の場合、列は値の複数の出現を許可する非ユニークインデックスの最初の列です。</li></ul>表の列の1つに複数の`COLUMN_KEY`値が適用される場合は、優先順位の高いものが順に`PRI`、`UNI`、`DUP`の順で表示されます。<br />`UNIQUE`インデックスは、`NULL`値を含めることができず、テーブルに`主キー`がない場合は`PRI`として表示されます。複数の列が合成`UNIQUE`インデックスを形成する場合は、`UNIQUE`インデックスは`MUL`と表示されることがあります。列の組み合わせは一意ですが、各列は値の複数の出現を保持できます。 |
| EXTRA                    | 列に関するその他の利用可能な情報。                         |
| PRIVILEGES               | 列に対する権限。                                            |
| COLUMN_COMMENT           | 列定義に含まれるコメント。                                  |
| COLUMN_SIZE              |                                                              |
| DECIMAL_DIGITS           |                                                              |
| GENERATION_EXPRESSION    | 生成された列の場合、列の値を計算するために使用される式。非生成列の場合は空です。 |
| SRS_ID                   | 空間列に適用されるこの値。列に格納されている値の空間参照システムを示す`SRID`値が含まれています。