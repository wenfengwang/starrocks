---
displayed_sidebar: "Japanese"
---

# コラム

`columns`には、すべてのテーブル列（またはビュー列）に関する情報が含まれています。

`columns`には、以下のフィールドが提供されます:

| **フィールド**             | **説明**                                                      |
| ------------------------ | ----------------------------------------------------------- |
| TABLE_CATALOG            | 列を含むテーブルが所属するカタログの名前。この値は常に`NULL`です。          |
| TABLE_SCHEMA             | 列を含むテーブルが所属するデータベースの名前。                              |
| TABLE_NAME               | 列を含むテーブルの名前。                                            |
| COLUMN_NAME              | 列の名前。                                                      |
| ORDINAL_POSITION         | テーブル内の列の序数位置。                                            |
| COLUMN_DEFAULT           | 列のデフォルト値。列に明示的な`NULL`のデフォルト値がある場合は、`NULL`です。または、列定義に`DEFAULT`句が含まれていない場合も`NULL`です。         |
| IS_NULLABLE              | 列のNULL可能性。列に`NULL`値を格納できる場合は`YES`、できない場合は`NO`です。        |
| DATA_TYPE                | 列のデータ型。`DATA_TYPE`の値は、その型名のみで他の情報は含まれません。`COLUMN_TYPE`の値には、その型名や精度、長さなど、他の可能な情報が含まれます。 |
| CHARACTER_MAXIMUM_LENGTH | 文字列列の場合、最大の文字数。                                           |
| CHARACTER_OCTET_LENGTH   | 文字列列の場合、最大のバイト数。                                           |
| NUMERIC_PRECISION        | 数値列の場合、数値の精度。                                             |
| NUMERIC_SCALE            | 数値列の場合、数値のスケール。                                            |
| DATETIME_PRECISION       | 日時列の場合、小数秒の精度。                                           |
| CHARACTER_SET_NAME       | 文字列列の場合、文字セットの名前。                                          |
| COLLATION_NAME           | 文字列列の場合、照合順序の名前。                                           |
| COLUMN_TYPE              | 列のデータ型。<br />`DATA_TYPE`の値は、その型名のみで他の情報は含まれません。`COLUMN_TYPE`の値には、その型名や精度、長さなど、他の可能な情報が含まれます。 |
| COLUMN_KEY               | 列がインデックス化されているか:<ul><li>`COLUMN_KEY`が空の場合、その列はインデックス化されていないか、複数列で構成されるユニークでないインデックスの2番目以降の列としてのみインデックス化されています。</li><li>`COLUMN_KEY`が`PRI`の場合、その列は`PRIMARY KEY`であるか、複数列の`PRIMARY KEY`の1つです。</li><li>`COLUMN_KEY`が`UNI`の場合、その列は`UNIQUE`インデックスの最初の列です(`UNIQUE`インデックスには複数の`NULL`値を含めることができますが、列が`NULL`を許可するかどうかは`Null`列をチェックすることで判断できます)。</li><li>`COLUMN_KEY`が`DUP`の場合、その列は非一意インデックスの最初の列であり、列内で特定の値の複数の出現が許可されています。</li></ul>表の特定の列に複数の`COLUMN_KEY`の値が適用される場合は、`COLUMN_KEY`は`PRI`、`UNI`、`DUP`の順で高い優先度のものを表示します。<br />`UNIQUE`インデックスは、`NULL`値を含めることができず、テーブルに`PRIMARY KEY`がない場合は、`PRI`として表示されます。複数の列が複合`UNIQUE`インデックスを形成する場合は、`MUL`として表示される場合があります。列の組み合わせは一意であるが、それぞれの列は特定の値の複数の出現を持つことができます。 |
| EXTRA                    | 特定の列に関する追加情報。                                              |
| PRIVILEGES               | 列に対する権限。                                                  |
| COLUMN_COMMENT           | 列定義に含まれるコメント。                                              |
| COLUMN_SIZE              |                                                              |
| DECIMAL_DIGITS           |                                                              |
| GENERATION_EXPRESSION    | 生成列の場合、列の値を計算するために使用される式を表示します。非生成列の場合は空です。          |
| SRS_ID                   | 空間列に適用されるこの値は、列に格納される値の空間参照システムを示す`SRID`値を含みます。          |