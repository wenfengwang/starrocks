---
displayed_sidebar: English
---

# 列

columns 包含了所有表格列（或视图列）的相关信息。

在 columns 中提供了以下字段：

|字段|描述|
|---|---|
|TABLE_CATALOG|包含该列的表所属的目录的名称。该值始终为 NULL。|
|TABLE_SCHEMA|包含该列的表所属的数据库的名称。|
|TABLE_NAME|包含该列的表的名称。|
|COLUMN_NAME|列的名称。|
|ORDINAL_POSITION|表中列的顺序位置。|
|COLUMN_DEFAULT|列的默认值。如果列具有显式默认值 NULL，或者列定义不包含 DEFAULT 子句，则此值为 NULL。|
|IS_NULLABLE|列的可为空性。如果列中可以存储 NULL 值，则值为 YES，否则值为 NO。|
|DATA_TYPE|列数据类型。 DATA_TYPE 值只是类型名称，没有其他信息。 COLUMN_TYPE 值包含类型名称以及可能的其他信息，例如精度或长度。|
|CHARACTER_MAXIMUM_LENGTH|对于字符串列，最大长度（以字符为单位）。|
|CHARACTER_OCTET_LENGTH|对于字符串列，最大长度（以字节为单位）。|
|NUMERIC_PRECISION|对于数字列，数字精度。|
|NUMERIC_SCALE|对于数字列，数字比例。|
|DATETIME_PRECISION|对于时间列，小数秒精度。|
|CHARACTER_SET_NAME|对于字符串列，字符集名称。|
|COLLATION_NAME|对于字符串列，排序规则名称。|
|COLUMN_TYPE|列数据类型。DATA_TYPE 值仅是类型名称，没有其他信息。 COLUMN_TYPE 值包含类型名称以及可能的其他信息，例如精度或长度。|
|COLUMN_KEY|列是否被索引：如果 COLUMN_KEY 为空，则该列不会被索引，或者仅作为多列、非唯一索引中的辅助列被索引。如果 COLUMN_KEY 为 PRI，则该列是 PRIMARY KEY 或多列 PRIMARY KEY 中的其中一列。如果 COLUMN_KEY 为 UNI，则该列是 UNIQUE 索引的第一列。 （UNIQUE 索引允许多个 NULL 值，但您可以通过检查 Null 列来判断该列是否允许 NULL。）如果 COLUMN_KEY 为 DUP，则该列是非唯一索引的第一列，其中允许给定值多次出现如果多个 COLUMN_KEY 值适用于表的给定列，则 COLUMN_KEY 按 PRI、UNI、DUP 的顺序显示具有最高优先级的值。如果不能，UNIQUE 索引可以显示为 PRI包含 NULL 值并且表中没有 PRIMARY KEY。如果多个列形成复合 UNIQUE 索引，则 UNIQUE 索引可能会显示为 MUL；尽管列的组合是唯一的，但每列仍然可以保存给定值的多次出现。|
|EXTRA|有关给定列的任何可用附加信息。|
|权限|您对该列拥有的权限。|
|COLUMN_COMMENT|列定义中包含的任何注释。|
|COLUMN_SIZE|
|十进制数字|
|GENERATION_EXPRESSION|对于生成的列，显示用于计算列值的表达式。对于非生成的列为空。|
|SRS_ID|此值适用于空间列。它包含列 SRID 值，该值指示存储在列中的值的空间参考系统。|
