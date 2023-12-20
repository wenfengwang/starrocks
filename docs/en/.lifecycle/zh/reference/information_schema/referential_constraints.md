---
displayed_sidebar: English
---

# 参照完整性约束

`referential_constraints` 包含所有参照（外键）约束。

`referential_constraints` 中提供了以下字段：

|**字段**|**描述**|
|---|---|
|CONSTRAINT_CATALOG|约束所属的目录名称。该值始终是 `def`。|
|CONSTRAINT_SCHEMA|约束所属的数据库名称。|
|CONSTRAINT_NAME|约束的名称。|
|UNIQUE_CONSTRAINT_CATALOG|包含约束所引用唯一约束的目录名称。该值始终是 `def`。|
|UNIQUE_CONSTRAINT_SCHEMA|包含约束所引用唯一约束的模式名称。|
|UNIQUE_CONSTRAINT_NAME|约束所引用的唯一约束的名称。|
|MATCH_OPTION|约束的 `MATCH` 属性值。目前唯一有效的值是 `NONE`。|
|UPDATE_RULE|约束的 `ON UPDATE` 属性值。有效值包括：`CASCADE`、`SET NULL`、`SET DEFAULT`、`RESTRICT` 和 `NO ACTION`。|
|DELETE_RULE|约束的 `ON DELETE` 属性值。有效值包括：`CASCADE`、`SET NULL`、`SET DEFAULT`、`RESTRICT` 和 `NO ACTION`。|
|TABLE_NAME|表的名称。该值与 `TABLE_CONSTRAINTS` 表中的值相同。|
|REFERENCED_TABLE_NAME|被约束引用的表的名称。|