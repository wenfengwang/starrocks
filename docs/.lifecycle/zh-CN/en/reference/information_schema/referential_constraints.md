---
displayed_sidebar: "Chinese"
---

# 引用约束

`referential_constraints` 包含所有引用（外键）约束。

在`referential_constraints`中提供了以下字段：

| **字段**                   | **描述**                                                     |
| ------------------------- | ------------------------------------------------------------ |
| CONSTRAINT_CATALOG        | 约束所属的目录名称。该值始终为 `def`。                        |
| CONSTRAINT_SCHEMA         | 约束所属的数据库名称。                                         |
| CONSTRAINT_NAME           | 约束的名称。                                                   |
| UNIQUE_CONSTRAINT_CATALOG | 约束引用的唯一约束所在的目录名称。该值始终为 `def`。          |
| UNIQUE_CONSTRAINT_SCHEMA  | 约束引用的唯一约束所在的模式名称。                            |
| UNIQUE_CONSTRAINT_NAME    | 约束引用的唯一约束的名称。                                     |
| MATCH_OPTION              | 约束 `MATCH` 属性的值。目前唯一有效的值为 `NONE`。             |
| UPDATE_RULE               | 约束 `ON UPDATE` 属性的值。有效值包括：`CASCADE`, `SET NULL`, `SET DEFAULT`, `RESTRICT`, 和 `NO ACTION`。 |
| DELETE_RULE               | 约束 `ON DELETE` 属性的值。有效值包括：`CASCADE`, `SET NULL`, `SET DEFAULT`, `RESTRICT`, 和 `NO ACTION`。 |
| TABLE_NAME                | 表的名称。该值与 `TABLE_CONSTRAINTS` 表中的相同。              |
| REFERENCED_TABLE_NAME     | 约束引用的表的名称。                                           |