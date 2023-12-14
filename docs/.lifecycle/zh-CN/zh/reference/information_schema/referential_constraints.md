---
displayed_sidebar: "中文"
---

# 参照性约束

`referential_constraints` 包含所有的参照（外键）约束信息。

`referential_constraints` 提供以下字段：

| 字段                      | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| CONSTRAINT_CATALOG        | 约束所属目录的名称。此值始终为 def。                         |
| CONSTRAINT_SCHEMA         | 约束所属数据库的名称。                                       |
| CONSTRAINT_NAME           | 约束的名称。                                                 |
| UNIQUE_CONSTRAINT_CATALOG | 包含约束引用的唯一约束所在目录的名称。此值始终为 def。       |
| UNIQUE_CONSTRAINT_SCHEMA  | 包含约束引用的唯一约束所在模式（schema）的名称。             |
| UNIQUE_CONSTRAINT_NAME    | 被引用的唯一约束的名称。                                     |
| MATCH_OPTION              | 约束的 MATCH 属性值。目前唯一有效的值是 NONE。               |
| UPDATE_RULE               | 约束的 ON UPDATE 属性值。有效值包括：CASCADE、SET NULL、SET DEFAULT、RESTRICT 和 NO ACTION。 |
| DELETE_RULE               | 约束的 ON DELETE 属性值。有效值包括：CASCADE、SET NULL、SET DEFAULT、RESTRICT 和 NO ACTION。 |
| TABLE_NAME                | 表的名称。此值与 TABLE_CONSTRAINTS 表中的名称相同。          |
| REFERENCED_TABLE_NAME     | 被参照的表的名称。                                           |