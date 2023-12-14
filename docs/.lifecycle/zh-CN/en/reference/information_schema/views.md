---
displayed_sidebar: "Chinese"
---

# 视图

`views`提供有关所有用户定义视图的信息。

`views`提供以下字段：

| **字段**               | **描述**                        |
| --------------------- | ------------------------------ |
| TABLE_CATALOG        | 视图所属目录的名称。该值始终为`def`。 |
| TABLE_SCHEMA         | 视图所属数据库的名称。             |
| TABLE_NAME           | 视图名称。                       |
| VIEW_DEFINITION      | 提供视图定义的`SELECT`语句。        |
| CHECK_OPTION         | `CHECK_OPTION`属性的值。该值为`NONE`、`CASCADE`或`LOCAL`中的一个。 |
| IS_UPDATABLE         | 视图是否可更新。如果可对视图进行`UPDATE`和`DELETE`（以及类似的操作），则将标志设置为`YES`（true）。否则，将标志设置为`NO`（false）。如果视图不可更新，则诸如`UPDATE`、`DELETE`和`INSERT`之类的语句是非法的并将被拒绝。 |
| DEFINER              | 创建视图的用户。                  |
| SECURITY_TYPE        | 视图的`SQL SECURITY`特性。该值为`DEFINER`或`INVOKER`中的一个。 |
| CHARACTER_SET_CLIENT |                                  |
| COLLATION_CONNECTION |                                  |
