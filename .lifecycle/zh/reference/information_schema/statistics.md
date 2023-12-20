---
displayed_sidebar: English
---

# 统计信息

统计信息提供了有关表索引的详细信息。

以下是统计信息中提供的字段：

|字段|描述|
|---|---|
|TABLE_CATALOG|包含索引的表所属的目录的名称。该值始终是默认值。|
|TABLE_SCHEMA|包含索引的表所属的数据库的名称。|
|TABLE_NAME|包含索引的表的名称。|
|NON_UNIQUE|如果索引不能包含重复项则为 0，如果可以则为 1。|
|INDEX_SCHEMA|索引所属数据库的名称。|
|INDEX_NAME|索引的名称。如果索引是主键，则名称始终为 PRIMARY。|
|SEQ_IN_INDEX|索引中的列序列号，从1开始。|
|COLUMN_NAME|列名称。另请参阅 EXPRESSION 列的说明。|
|COLLATION|列在索引中的排序方式。这可以有值 A（升序）、D（降序）或 NULL（未排序）。|
|基数|索引中唯一值数量的估计。|
|SUB_PART|索引前缀。也就是说，如果列仅部分索引，则索引字符数；如果整个列都索引，则为 NULL。|
|PACKED|指示密钥的打包方式。如果不是，则为 NULL。|
|NULLABLE|如果列可能包含 NULL 值，则包含 YES，否则包含 ''。|
|INDEX_TYPE|使用的索引方法（BTREE、FULLTEXT、HASH、RTREE）。|
|COMMENT|有关索引的信息未在其自己的列中描述，例如如果索引已禁用，则禁用。|
|INDEX_COMMENT|创建索引时为带有 COMMENT 属性的索引提供的任何注释。|
