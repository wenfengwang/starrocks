---
displayed_sidebar: English
---

# 表权限

`table_privileges` 提供有关表权限的信息。

`table_privileges` 中提供了以下字段：

| **字段**      | **描述**                                              |
| -------------- | ------------------------------------------------------------ |
| GRANTEE        | 被授予权限的用户的名称。      |
| TABLE_CATALOG  | 表所属的目录的名称。此值始终为 `def`。 |
| TABLE_SCHEMA   | 表所属的数据库的名称。         |
| TABLE_NAME     | 表的名称。                                       |
| PRIVILEGE_TYPE | 授予的权限类型。该值可以是可以在表级别授予的任何权限。 |
| IS_GRANTABLE   | 如果用户具有`GRANT OPTION`权限则为`YES`，否则为`NO`。输出中不会单独列出`PRIVILEGE_TYPE='GRANT OPTION'`的`GRANT OPTION`权限。 |
