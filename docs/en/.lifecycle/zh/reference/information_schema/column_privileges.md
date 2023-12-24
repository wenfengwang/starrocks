---
displayed_sidebar: English
---
# 列特权

`column_privileges` 标识当前启用的角色或由当前启用的角色授予的列的所有特权。

`column_privileges` 中提供了以下字段：

| **字段**      | **描述**                                              |
| -------------- | ------------------------------------------------------------ |
| 受权人        | 被授予该特权的用户的名称。      |
| TABLE_CATALOG  | 包含该列的表所属的目录的名称。该值始终为 `def`。 |
| TABLE_SCHEMA   | 包含该列的表所属的数据库的名称。 |
| TABLE_NAME     | 包含该列的表的名称。                 |
| COLUMN_NAME    | 列的名称。                                      |
| PRIVILEGE_TYPE | 授予的特权。该值可以是可以在列级别授予的任何权限。每行列出一个特权，因此被授予者拥有的每列特权都有一行。 |
| IS_GRANTABLE   | 如果用户具有 `GRANT OPTION` 特权，则为 `YES`，否则为 `NO`。输出不会将 `GRANT OPTION` 作为单独的行列出，而是以 `PRIVILEGE_TYPE='GRANT OPTION'` 的形式显示。