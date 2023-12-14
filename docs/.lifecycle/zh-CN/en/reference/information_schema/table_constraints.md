---
displayed_sidebar: "英文"
---

# table_constraints（表约束）

`table_constraints` 用来描述哪些表含有约束。

在 `table_constraints` 中提供了以下字段：

| **字段**            | **描述**                                                      |
| -------------------- | ------------------------------------------------------------ |
| CONSTRAINT_CATALOG   | 约束所属的目录名称。这个值总是 `def`。                        |
| CONSTRAINT_SCHEMA    | 约束所属的数据库名称。                                        |
| CONSTRAINT_NAME      | 约束的名称。                                                  |
| TABLE_SCHEMA         | 表所属的数据库名称。                                          |
| TABLE_NAME           | 表的名称。                                                    |
| CONSTRAINT_TYPE      | 约束的类型。                                                  |