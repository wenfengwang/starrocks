---
displayed_sidebar: English
---

# 分区

`partitions` 提供有关表分区的信息。

`partitions` 中提供了以下字段：

|**字段**|**描述**|
|---|---|
|TABLE_CATALOG|表所属目录的名称。该值始终是 `def`。|
|TABLE_SCHEMA|表所属的数据库名称。|
|TABLE_NAME|包含分区的表的名称。|
|PARTITION_NAME|分区的名称。|
|SUBPARTITION_NAME|如果 `partitions` 表行代表子分区，则为子分区的名称；否则为 `NULL`。对于 `NDB`：该值始终为 `NULL`。|
|PARTITION_ORDINAL_POSITION|所有分区都按照其定义的顺序进行索引，其中 `1` 是分配给第一个分区的编号。索引可能会随着分区的添加、删除和重组而改变；显示的数字反映当前顺序，考虑到任何索引更改。|
|PARTITION_METHOD|有效值：`RANGE`、`LIST`、`HASH`、`LINEAR HASH`、`KEY` 或 `LINEAR KEY`。|
|SUBPARTITION_METHOD|有效值：`HASH`、`LINEAR HASH`、`KEY` 或 `LINEAR KEY`。|
|PARTITION_EXPRESSION|在创建表当前分区方案的 `CREATE TABLE` 或 `ALTER TABLE` 语句中使用的分区函数表达式。|
|SUBPARTITION_EXPRESSION|这与定义表子分区的子分区表达式的工作方式相同，如 `PARTITION_EXPRESSION` 对于定义表分区的分区表达式。如果表没有子分区，则此列为 `NULL`。|
|PARTITION_DESCRIPTION|此列用于 `RANGE` 和 `LIST` 分区。对于 `RANGE` 分区，它包含分区的 `VALUES LESS THAN` 子句中设置的值，可以是整数或 `MAXVALUE`。对于 `LIST` 分区，此列包含分区的 `VALUES IN` 子句中定义的值，是逗号分隔的整数值列表。对于 `PARTITION_METHOD` 不是 `RANGE` 或 `LIST` 的分区，此列始终为 `NULL`。|
|TABLE_ROWS|分区中的表行数。|
|AVG_ROW_LENGTH|此分区或子分区中存储的行的平均长度，以字节为单位。这与 `DATA_LENGTH` 除以 `TABLE_ROWS` 相同。|
|DATA_LENGTH|此分区或子分区中存储的所有行的总长度，以字节为单位；即分区或子分区中存储的总字节数。|
|MAX_DATA_LENGTH|此分区或子分区中可以存储的最大字节数。|
|INDEX_LENGTH|此分区或子分区的索引文件长度，以字节为单位。|
|DATA_FREE|分配给分区或子分区但未使用的字节数。|
|CREATE_TIME|创建分区或子分区的时间。|
|UPDATE_TIME|上次修改分区或子分区的时间。|
|CHECK_TIME|最后一次检查该分区或子分区所属表的时间。|
|CHECKSUM|校验和值（如果有）；否则为 `NULL`。|
|PARTITION_COMMENT|注释文本（如果分区有的话）。如果没有，则该值为空。|
|NODEGROUP|分区所属的节点组。|
|TABLESPACE_NAME|分区所属的表空间名称。|