---
displayed_sidebar: "Chinese"
---

# key_column_usage

`key_column_usage`标识受某些唯一、主键或外键约束限制的所有列。

在`key_column_usage`中提供以下字段：

| **字段**                       | **描述**                                                   |
| ---------------------------- | ------------------------------------------------------------ |
| CONSTRAINT_CATALOG            | 约束所属目录的名称。此值始终为“def” 。                  |
| CONSTRAINT_SCHEMA             | 约束所属数据库的名称。                                       |
| CONSTRAINT_NAME               | 约束的名称。                                                  |
| TABLE_CATALOG                 | 表所属目录的名称。此值始终为“def” 。                      |
| TABLE_SCHEMA                  | 表所属数据库的名称。                                          |
| TABLE_NAME                    | 具有该约束的表的名称。                                       |
| COLUMN_NAME                   | 具有约束的列的名称。如果约束是外键，则这是外键的列，而不是外键引用的列。 |
| ORDINAL_POSITION              | 约束内列的位置，而不是表内列的位置。列位置从1开始编号。    |
| POSITION_IN_UNIQUE_CONSTRAINT | 唯一约束和主键约束为`NULL`。对于外键约束，此列是被引用表中键的顺序位置。 |
| REFERENCED_TABLE_SCHEMA       | 约束引用的模式的名称。                                       |
| REFERENCED_TABLE_NAME         | 约束引用的表的名称。                                        |
| REFERENCED_COLUMN_NAME        | 约束引用的列的名称。                                        |