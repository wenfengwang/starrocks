---
displayed_sidebar: "Chinese"
---

# columns

`columns` 包含有关所有表列（或视图列）的信息。

在`columns`中提供以下字段：

| **字段**                  | **描述**                                                     |
| ------------------------ | ------------------------------------------------------------ |
| TABLE_CATALOG            | 包含列的表所属目录的名称。该值始终为`NULL`。|
| TABLE_SCHEMA             | 包含列的表所属数据库的名称。 |
| TABLE_NAME               | 包含列的表的名称。|
| COLUMN_NAME              | 列的名称。|
| ORDINAL_POSITION         | 列在表中的顺序位置。|
| COLUMN_DEFAULT           | 列的默认值。如果列具有显式的`NULL`默认值，或者列定义不包括`DEFAULT`子句，则此值为`NULL`。|
| IS_NULLABLE              | 列的可空性。如果可以在列中存储`NULL`值，则值为`YES`，否则为`NO`。|
| DATA_TYPE                | 列的数据类型。`DATA_TYPE`值仅为类型名称，不包含其他信息。`COLUMN_TYPE`值包含类型名称和可能的其他信息，如精度或长度。|
| CHARACTER_MAXIMUM_LENGTH | 字符串列的最大字符长度。|
| CHARACTER_OCTET_LENGTH   | 字符串列的最大字节长度。|
| NUMERIC_PRECISION        | 数值列的数值精度。|
| NUMERIC_SCALE            | 数值列的数值刻度。|
| DATETIME_PRECISION       | 时间列的小数秒精度。|
| CHARACTER_SET_NAME       | 字符串列的字符集名称。|
| COLLATION_NAME           | 字符串列的校对名称。|
| COLUMN_TYPE              | 列的数据类型。<br />`DATA_TYPE`值仅为类型名称，不包含其他信息。`COLUMN_TYPE`值包含类型名称和可能的其他信息，如精度或长度。|
| COLUMN_KEY               | 列是否被索引:<ul><li>如果`COLUMN_KEY`为空，则该列既未被索引，也只能作为多列非唯一索引的次要列。</li><li>如果`COLUMN_KEY`为`PRI`，则该列是`PRIMARY KEY`，或者是多列`PRIMARY KEY`中的一列。</li><li>如果`COLUMN_KEY`为`UNI`，则该列是`UNIQUE`索引的第一列。（`UNIQUE`索引允许多个`NULL`值，但可以通过检查`Null`列来确定列是否允许`NULL`。）</li><li>如果`COLUMN_KEY`为`DUP`，则该列是允许在列内允许给定值的多个出现的非唯一索引的第一列。</li></ul>如果多个`COLUMN_KEY`值适用于表的给定列，则`COLUMN_KEY`显示优先级最高的一个，按照`PRI`，`UNI`，`DUP`的顺序显示。<br />如果`UNIQUE`索引不能包含`NULL`值，并且表中没有`PRIMARY KEY`，则该`UNIQUE`索引可能显示为`PRI`。如果几列形成复合的`UNIQUE`索引，则该`UNIQUE`索引可能显示为`MUL`；尽管列的组合是唯一的，但每个列仍然可以包含给定值的多个出现。|
| EXTRA                    | 有关给定列的任何其他信息。|
| PRIVILEGES               | 您对该列拥有的权限。|
| COLUMN_COMMENT           | 包含在列定义中的任何注释。|
| COLUMN_SIZE              |                                                              |
| DECIMAL_DIGITS           |                                                              |
| GENERATION_EXPRESSION    | 对于生成列，显示用于计算列值的表达式。对于非生成列为空。|
| SRS_ID                   | 此值适用于空间列。其中包含`SRID`值，指示存储在列中的值的空间参考系统。|