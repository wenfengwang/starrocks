---
displayed_sidebar: English
---

# columns

`columns` 包含所有表列（或视图列）的信息。

`columns` 中提供以下字段：

|**字段**|**描述**|
|---|---|
|TABLE_CATALOG|包含该列的表所属目录的名称。此值始终为 `NULL`。|
|TABLE_SCHEMA|包含该列的表所属数据库的名称。|
|TABLE_NAME|包含该列的表的名称。|
|COLUMN_NAME|列的名称。|
|ORDINAL_POSITION|列在表中的顺序位置。|
|COLUMN_DEFAULT|列的默认值。如果列有显式的默认值 `NULL`，或者列定义没有包含 `DEFAULT` 子句，则此值为 `NULL`。|
|IS_NULLABLE|列的可空性。如果列可以存储 `NULL` 值，则值为 `YES`；如果不可以，则值为 `NO`。|
|DATA_TYPE|列的数据类型。`DATA_TYPE` 值仅包含类型名称，不包含其他信息。`COLUMN_TYPE` 值包含类型名称及可能的其他信息，如精度或长度。|
|CHARACTER_MAXIMUM_LENGTH|对于字符串列，字符的最大长度。|
|CHARACTER_OCTET_LENGTH|对于字符串列，字节的最大长度。|
|NUMERIC_PRECISION|对于数值列，数值的精度。|
|NUMERIC_SCALE|对于数值列，数值的刻度。|
|DATETIME_PRECISION|对于时间列，小数秒的精度。|
|CHARACTER_SET_NAME|对于字符字符串列，字符集名称。|
|COLLATION_NAME|对于字符字符串列，排序规则名称。|
|COLUMN_TYPE|列的数据类型。<br />`DATA_TYPE` 值仅为类型名称，不包含其他信息。`COLUMN_TYPE` 值包含类型名称及可能的其他信息，如精度或长度。|
|COLUMN_KEY|列是否被索引：<ul><li>如果 `COLUMN_KEY` 为空，表示该列未被索引或仅作为多列非唯一索引的次要列。</li><li>如果 `COLUMN_KEY` 为 `PRI`，表示该列是 `PRIMARY KEY` 或多列 `PRIMARY KEY` 中的一部分。</li><li>如果 `COLUMN_KEY` 为 `UNI`，表示该列是 `UNIQUE` 索引的首列。（`UNIQUE` 索引允许多个 `NULL` 值，但可以通过检查 `IS_NULLABLE` 字段来确定该列是否允许 `NULL`。）</li><li>如果 `COLUMN_KEY` 为 `DUP`，表示该列是非唯一索引的首列，允许该列中给定值的多次出现。</li></ul>如果多个 `COLUMN_KEY` 值适用于表的某列，`COLUMN_KEY` 将显示优先级最高的一个，优先级顺序为 `PRI`、`UNI`、`DUP`。<br />如果 `UNIQUE` 索引不允许 `NULL` 值且表中无 `PRIMARY KEY`，则可能显示为 `PRI`。如果多列组成复合 `UNIQUE` 索引，则 `UNIQUE` 索引可能显示为 `MUL`；尽管列的组合是唯一的，每列仍可包含给定值的多次出现。|
|EXTRA|关于给定列的任何额外信息。|
|PRIVILEGES|您对该列的权限。|
|COLUMN_COMMENT|列定义中包含的注释。|
|COLUMN_SIZE|
|DECIMAL_DIGITS|
|GENERATION_EXPRESSION|对于生成列，显示用于计算列值的表达式。对于非生成列为空。|
|SRS_ID|此值适用于空间列。它包含列 `SRID` 值，指示存储在列中的值的空间参考系统。|