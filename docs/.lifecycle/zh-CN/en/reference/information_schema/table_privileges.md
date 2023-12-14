---
displayed_sidebar: "Chinese"
---

# table_privileges

`table_privileges`提供有关表权限的信息。

`table_privileges`中提供了以下字段：

| **字段**        | **描述**                                                    |
| -------------- | ------------------------------------------------------------ |
| GRANTEE        | 被授予权限的用户的名称。                                       |
| TABLE_CATALOG  | 表所属目录的名称。该值始终为`def`。                           |
| TABLE_SCHEMA   | 表所属数据库的名称。                                           |
| TABLE_NAME     | 表的名称。                                                    |
| PRIVILEGE_TYPE | 授予的权限。该值可以是可在表级别授予的任何权限。               |
| IS_GRANTABLE   | 如果用户具有`GRANT OPTION`权限则为`YES`，否则为`NO`。输出不会列出`PRIVILEGE_TYPE='GRANT OPTION'`的`GRANT OPTION`作为单独的行。 |