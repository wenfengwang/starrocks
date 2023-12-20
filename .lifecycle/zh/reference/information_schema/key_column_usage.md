---
displayed_sidebar: English
---

# 键列使用情况

key_column_usage 标识所有受唯一性、主键或外键约束限制的列。

key_column_usage 提供以下字段：

|字段|描述|
|---|---|
|CONSTRAINT_CATALOG|约束所属目录的名称。该值始终是默认值。|
|CONSTRAINT_SCHEMA|约束所属的数据库的名称。|
|CONSTRAINT_NAME|约束的名称。|
|TABLE_CATALOG|表所属目录的名称。该值始终是默认值。|
|TABLE_SCHEMA|表所属的数据库的名称。|
|TABLE_NAME|具有约束的表的名称。|
|COLUMN_NAME|具有约束的列的名称。如果约束是外键，则这是外键的列，而不是外键引用的列。|
|ORDINAL_POSITION|列在约束中的位置，而不是列在表中的位置。列位置从 1 开始编号。|
|POSITION_IN_UNIQUE_CONSTRAINT|NULL 表示唯一约束和主键约束。对于外键约束，此列是正在引用的表的键中的顺序位置。|
|REFERENCED_TABLE_SCHEMA|约束引用的架构的名称。|
|REFERENCED_TABLE_NAME|约束引用的表的名称。|
|REFERENCED_COLUMN_NAME|约束引用的列的名称。|
