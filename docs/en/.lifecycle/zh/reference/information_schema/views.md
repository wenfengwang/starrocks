---
displayed_sidebar: English
---

# 视图

`views` 提供有关所有用户定义视图的信息。

`views` 中提供了以下字段：

| **字段**            | **描述**                                              |
| -------------------- | ------------------------------------------------------------ |
| TABLE_CATALOG        | 视图所属的目录名称。此值始终为 `def`。 |
| TABLE_SCHEMA         | 视图所属的数据库名称。          |
| TABLE_NAME           | 视图的名称。                                        |
| VIEW_DEFINITION      | 提供视图定义的 `SELECT` 语句。 |
| CHECK_OPTION         | `CHECK_OPTION` 属性的值。该值是 `NONE`、 `CASCADE` 或 `LOCAL`之一。 |
| IS_UPDATABLE         | 视图是否可更新。如果对视图进行 `UPDATE`、`DELETE`（以及类似操作）是合法的，则该标志设置为 `YES`（true）。否则，该标志设置为 `NO`（false）。如果视图不可更新，则诸如 `UPDATE`、`DELETE` 和 `INSERT` 等语句是非法的，将被拒绝。 |
| DEFINER              | 创建视图的用户。                   |
| SECURITY_TYPE        | 视图的 `SQL SECURITY` 特性。该值是 `DEFINER` 或 `INVOKER`之一。 |
| CHARACTER_SET_CLIENT |                                                              |
| COLLATION_CONNECTION |                                                              |

