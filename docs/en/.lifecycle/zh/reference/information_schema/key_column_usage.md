---
displayed_sidebar: English
---

# key_column_usage

`key_column_usage` 标识所有受唯一约束、主键约束或外键约束限制的列。

`key_column_usage` 中提供了以下字段：

|**字段**|**描述**|
|---|---|
|CONSTRAINT_CATALOG|约束所属的目录名称。该值始终为 `def`。|
|CONSTRAINT_SCHEMA|约束所属的数据库名称。|
|CONSTRAINT_NAME|约束的名称。|
|TABLE_CATALOG|表所属的目录名称。该值始终为 `def`。|
|TABLE_SCHEMA|表所属的数据库名称。|
|TABLE_NAME|具有约束的表的名称。|
|COLUMN_NAME|具有约束的列的名称。如果约束是外键，则这是外键列，而非外键所引用的列。|
|ORDINAL_POSITION|列在约束中的位置，而非列在表中的位置。列位置编号从 1 开始。|
|POSITION_IN_UNIQUE_CONSTRAINT|对于唯一约束和主键约束，此列为 `NULL`。对于外键约束，此列是被引用表中的唯一键的顺序位置。|
|REFERENCED_TABLE_SCHEMA|被约束引用的模式名称。|
|REFERENCED_TABLE_NAME|被约束引用的表的名称。|
|REFERENCED_COLUMN_NAME|被约束引用的列的名称。|