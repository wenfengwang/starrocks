---
displayed_sidebar: English
---

# 统计信息

`statistics` 提供有关表索引的信息。

`statistics` 中提供以下字段：

|**字段**|**描述**|
|---|---|
|TABLE_CATALOG|包含索引的表所属的目录名称。该值始终为 `def`。|
|TABLE_SCHEMA|包含索引的表所属的数据库名称。|
|TABLE_NAME|包含索引的表的名称。|
|NON_UNIQUE|如果索引不能包含重复项则为 0，如果可以则为 1。|
|INDEX_SCHEMA|索引所属的数据库名称。|
|INDEX_NAME|索引的名称。如果索引是主键，名称始终为 `PRIMARY`。|
|SEQ_IN_INDEX|索引中的列序号，从 1 开始。|
|COLUMN_NAME|列名称。另请参见 `EXPRESSION` 列的描述。|
|COLLATION|列在索引中的排序方式。可能的值有 `A`（升序）、`D`（降序）或 `NULL`（未排序）。|
|CARDINALITY|索引中唯一值的估计数量。|
|SUB_PART|索引前缀。即，如果列只是部分被索引，则为索引字符的数量；如果整个列被索引，则为 `NULL`。|
|PACKED|指示键值的打包方式。如果没有打包，则为 `NULL`。|
|NULLABLE|如果列可以包含 `NULL` 值，则为 `YES`；如果不可以，则为 `''`。|
|INDEX_TYPE|所使用的索引方法（`BTREE`、`FULLTEXT`、`HASH`、`RTREE`）。|
|COMMENT|索引的其他信息，未在其他列中描述，例如如果索引被禁用，则为 `disabled`。|
|INDEX_COMMENT|创建索引时，为带有 `COMMENT` 属性的索引提供的任何注释。|