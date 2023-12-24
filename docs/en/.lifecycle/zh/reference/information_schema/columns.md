---
displayed_sidebar: English
---

# 列

`columns` 包含有关所有表列（或视图列）的信息。

`columns` 中提供了以下字段：

| **字段**                | **描述**                                              |
| ------------------------ | ------------------------------------------------------------ |
| TABLE_CATALOG            | 包含包含该列的表所属的目录的名称。此值始终为 `NULL`。 |
| TABLE_SCHEMA             | 包含包含该列的表所属的数据库的名称。 |
| TABLE_NAME               | 包含包含该列的表的名称。                 |
| COLUMN_NAME              | 列的名称。                                      |
| ORDINAL_POSITION         | 列在表中的序号位置。         |
| COLUMN_DEFAULT           | 列的默认值。如果列具有显式默认值为 `NULL`，或者列定义中不包含 `DEFAULT` 子句，则该值为 `NULL`。|
| IS_NULLABLE              | 列的可为 null 性。如果 `NULL` 值可以存储在列中，则该值为 `YES`，如果不能，则为 `NO`。 |
| DATA_TYPE                | 列的数据类型。`DATA_TYPE` 值只包含类型名称，没有其他信息。`COLUMN_TYPE` 值包含类型名称以及可能的其他信息，例如精度或长度。 |
| CHARACTER_MAXIMUM_LENGTH | 字符串列的最大长度（以字符为单位）。        |
| CHARACTER_OCTET_LENGTH   | 字符串列的最大长度（以字节为单位）。             |
| NUMERIC_PRECISION        | 数值列的数值精度。                  |
| NUMERIC_SCALE            | 数值列的数值刻度。                      |
| DATETIME_PRECISION       | 时态列的小数秒精度。      |
| CHARACTER_SET_NAME       | 字符串列的字符集名称。        |
| COLLATION_NAME           | 字符串列的排序规则名称。            |
| COLUMN_TYPE              | 列的数据类型。<br />`DATA_TYPE` 值只包含类型名称，没有其他信息。`COLUMN_TYPE` 值包含类型名称以及可能的其他信息，例如精度或长度。 |
| COLUMN_KEY               | 列是否已编制索引：<ul><li>如果 `COLUMN_KEY` 为空，则该列要么未编制索引，要么仅作为多列非唯一索引中的辅助列编制索引。</li><li>如果 `COLUMN_KEY` 为 `PRI`，则该列是 `PRIMARY KEY` 或是多列 `PRIMARY KEY` 中的一列。</li><li>如果 `COLUMN_KEY` 为 `UNI`，则该列是 `UNIQUE` 索引的第一列。（`UNIQUE` 索引允许多个 `NULL` 值，但您可以通过检查 `Null` 列来判断该列是否允许 `NULL`。）</li><li>如果 `COLUMN_KEY` 为 `DUP`，则该列是非唯一索引的第一列，在该索引中允许列中给定值的多次出现。</li></ul>如果表的给定列适用多个 `COLUMN_KEY` 值，则 `COLUMN_KEY` 会显示优先级最高的值，按照 `PRI`、`UNI`、`DUP` 的顺序。<br />如果 `UNIQUE` 索引不能包含 `NULL` 值，并且表中没有 `PRIMARY KEY`，则 `UNIQUE` 索引可能显示为 `PRI`。如果几列形成复合 `UNIQUE` 索引，则该索引可能显示为 `MUL`；尽管列的组合是唯一的，但每列仍然可以保存给定值的多个匹配项。 |
| EXTRA                    | 有关给定列的任何其他可用信息。 |
| PRIVILEGES               | 您对列拥有的权限。                      |
| COLUMN_COMMENT           | 列定义中包含的任何注释。               |
| COLUMN_SIZE              |                                                              |
| DECIMAL_DIGITS           |                                                              |
| GENERATION_EXPRESSION    | 对于生成的列，显示用于计算列值的表达式。对于未生成的列，该值为空。 |
| SRS_ID                   | 此值适用于空间列。它包含指示存储在列中的值的空间参考系统的 `SRID` 值。 |
