---
displayed_sidebar: English
---

# 分区

`partitions` 提供关于表分区的信息。

`partitions` 中提供了以下字段：

| **字段**                  | **描述**                                              |
| -------------------------- | ------------------------------------------------------------ |
| TABLE_CATALOG              | 表所属的目录名称。此值始终为 `def`。 |
| TABLE_SCHEMA               | 表所属的数据库名称。         |
| TABLE_NAME                 | 包含分区的表的名称。              |
| PARTITION_NAME             | 分区的名称。                                   |
| SUBPARTITION_NAME          | 如果 `PARTITIONS` 表行表示子分区，则为子分区的名称；否则为 `NULL`。对于 `NDB`：此值始终为 `NULL`。 |
| PARTITION_ORDINAL_POSITION | 所有分区按照定义顺序进行索引，`1` 表示分配给第一个分区的编号。随着分区的添加、删除和重组，索引可能会发生变化；此列显示的数字反映了当前顺序，考虑到了任何索引更改。 |
| PARTITION_METHOD           | 有效值：`RANGE`、`LIST`、`HASH`、`LINEAR HASH`、`KEY`或`LINEAR KEY`。 |
| SUBPARTITION_METHOD        | 有效值：`HASH`、`LINEAR HASH`、`KEY`或`LINEAR KEY`  |
| PARTITION_EXPRESSION       | 在创建表的 `CREATE TABLE` 或 `ALTER TABLE` 语句中使用的分区函数的表达式。|
| SUBPARTITION_EXPRESSION    | 与用于定义表分区的 `PARTITION_EXPRESSION` 类似，用于定义表子分区的子分区表达式。如果表没有子分区，则此列为 `NULL`。 |
| PARTITION_DESCRIPTION      | 此列用于 `RANGE` 和 `LIST` 分区。对于 `RANGE` 分区，它包含在分区的 `VALUES LESS THAN` 子句中设置的值，可以是整数或 `MAXVALUE`。对于 `LIST` 分区，此列包含在分区的 `VALUES IN` 子句中定义的值，即逗号分隔的整数值列表。对于 `PARTITION_METHOD` 为 `RANGE` 或 `LIST` 以外的分区，此列始终为 `NULL`。 |
| TABLE_ROWS                 | 分区中的表行数。                   |
| AVG_ROW_LENGTH             | 此分区或子分区中存储的行的平均长度，以字节为单位。这与 `DATA_LENGTH` 除以 `TABLE_ROWS` 的结果相同。 |
| DATA_LENGTH                | 此分区或子分区中存储的所有行的总长度，以字节为单位；即存储在分区或子分区中的总字节数。 |
| MAX_DATA_LENGTH            | 此分区或子分区中可以存储的最大字节数。 |
| INDEX_LENGTH               | 此分区或子分区的索引文件长度，以字节为单位。 |
| DATA_FREE                  | 分配给分区或子分区但未使用的字节数。 |
| CREATE_TIME                | 创建分区或子分区的时间。     |
| UPDATE_TIME                | 上次修改分区或子分区的时间。 |
| CHECK_TIME                 | 上次检查此分区或子分区所属的表的时间。 |
| CHECKSUM                   | 校验和值（如果有），否则为 `NULL`。               |
| PARTITION_COMMENT          | 分区的注释文本（如果有）。否则，此值为空。 |
| NODEGROUP                  | 分区所属的节点组。        |
| TABLESPACE_NAME            | 分区所属的表空间的名称。   |
