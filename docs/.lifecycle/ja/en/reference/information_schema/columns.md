---
displayed_sidebar: English
---

# columns

`columns` は、すべてのテーブル列（またはビュー列）に関する情報を含んでいます。

`columns` には、以下のフィールドが提供されています：

| **フィールド**                | **説明**                                              |
| ------------------------ | ------------------------------------------------------------ |
| TABLE_CATALOG            | 列が含まれるテーブルが属するカタログの名前。この値は常に `NULL` です。 |
| TABLE_SCHEMA             | 列が含まれるテーブルが属するデータベースの名前。 |
| TABLE_NAME               | 列が含まれるテーブルの名前。                 |
| COLUMN_NAME              | 列の名前。                                      |
| ORDINAL_POSITION         | テーブル内での列の順序位置。         |
| COLUMN_DEFAULT           | 列のデフォルト値。これは、列が `NULL` を明示的なデフォルトとして持つ場合、または列定義に `DEFAULT` 句が含まれていない場合に `NULL` です。 |
| IS_NULLABLE              | 列のNULL許容性。`NULL` 値が列に格納できる場合は `YES`、できない場合は `NO` です。|
| DATA_TYPE                | 列のデータ型。`DATA_TYPE` の値は型名のみで、他の情報は含まれません。`COLUMN_TYPE` の値には型名と、場合によっては精度や長さなどの他の情報が含まれます。 |
| CHARACTER_MAXIMUM_LENGTH | 文字列列の場合、文字の最大長。        |
| CHARACTER_OCTET_LENGTH   | 文字列列の場合、バイト単位の最大長。             |
| NUMERIC_PRECISION        | 数値列の場合、数値の精度。                  |
| NUMERIC_SCALE            | 数値列の場合、数値のスケール。                      |
| DATETIME_PRECISION       | 時間列の場合、小数秒の精度。      |
| CHARACTER_SET_NAME       | 文字列列の場合、文字セットの名前。        |
| COLLATION_NAME           | 文字列列の場合、照合順序の名前。            |
| COLUMN_TYPE              | 列のデータ型。<br />`DATA_TYPE` の値は型名のみで、他の情報は含まれません。`COLUMN_TYPE` の値には型名と、場合によっては精度や長さなどの他の情報が含まれます。 |
| COLUMN_KEY               | 列がインデックスされているかどうか：<ul><li>`COLUMN_KEY` が空の場合、列はインデックスされていないか、複数列の非ユニークインデックスのセカンダリ列としてのみインデックスされています。</li><li>`COLUMN_KEY` が `PRI` の場合、列は `PRIMARY KEY` であるか、複数列の `PRIMARY KEY` の一部です。</li><li>`COLUMN_KEY` が `UNI` の場合、列は `UNIQUE` インデックスの最初の列です。（`UNIQUE` インデックスは複数の `NULL` 値を許可しますが、列が `NULL` を許可するかどうかは `IS_NULLABLE` 列を確認することで判断できます。）</li><li>`COLUMN_KEY` が `DUP` の場合、列は非ユニークインデックスの最初の列で、列内で特定の値の複数の出現が許可されています。</li></ul>テーブルの特定の列に複数の `COLUMN_KEY` 値が当てはまる場合、`COLUMN_KEY` は優先順位が最も高いものを表示します。優先順位は `PRI`、`UNI`、`DUP` の順です。<br />`UNIQUE` インデックスは、`NULL` 値を含むことができない場合やテーブルに `PRIMARY KEY` がない場合に `PRI` として表示されることがあります。複数の列が `UNIQUE` インデックスを形成する場合、それぞれの列は特定の値を複数回含むことができるため、`MUL` として表示されることがあります。 |
| EXTRA                    | 列に関する利用可能な追加情報。 |
| PRIVILEGES               | 列に対する権限。                      |
| COLUMN_COMMENT           | 列定義に含まれるコメント。               |
| COLUMN_SIZE              |                                                              |
| DECIMAL_DIGITS           |                                                              |
| GENERATION_EXPRESSION    | 生成列の場合、列の値を計算するために使用される式が表示されます。非生成列の場合は空です。 |
| SRS_ID                   | この値は空間列に適用されます。列に格納された値の空間参照系を示す `SRID` 値が含まれます。 |
