---
displayed_sidebar: "Chinese"
---

# 统计

`statistics`提供关于表索引的信息。

`statistics`包含以下字段：

| **字段**       | **描述**                                                   |
| ------------- | ------------------------------------------------------------ |
| TABLE_CATALOG | 包含索引的表所属的目录的名称。此值始终为“def”。               |
| TABLE_SCHEMA  | 包含索引的表所属的数据库的名称。                               |
| TABLE_NAME    | 包含索引的表的名称。                                        |
| NON_UNIQUE    | 如果索引不包含重复项则为0，如果包含则为1。                     |
| INDEX_SCHEMA  | 索引所属的数据库的名称。                                      |
| INDEX_NAME    | 索引的名称。如果索引是主键，则名称始终为“PRIMARY”。             |
| SEQ_IN_INDEX  | 索引中的列序号，从1开始。                                      |
| COLUMN_NAME   | 列名。还可以查看“EXPRESSION”列的描述。                           |
| COLLATION     | 索引中列的排序方式。可以是“A”（升序）、“D”（降序）或“NULL”（未排序）。 |
| CARDINALITY   | 索引中唯一值的估计数量。                                       |
| SUB_PART      | 索引前缀。即，如果列仅部分索引，则为索引字符的数量，如果整列被索引，则为“NULL”。 |
| PACKED        | 指示键的打包方式。如果不是，则为“NULL”。                          |
| NULLABLE      | 如果列可能包含“NULL”值，则为“YES”，否则为“''”。                 |
| INDEX_TYPE    | 使用的索引方法（“BTREE”、“FULLTEXT”、“HASH”、“RTREE”）。          |
| COMMENT       | 索引的其他信息，比如索引为禁用状态则显示“disabled”。                |
| INDEX_COMMENT | 创建索引时使用`COMMENT`属性提供的索引备注。                        |