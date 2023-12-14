---
displayed_sidebar: "Chinese"
---

# column_privileges

`column_privileges`标识当前启用的角色或由当前启用的角色授予的列的所有权限。

`column_privileges`提供以下字段：

| **字段**        | **描述**                                                     |
| -------------- | ------------------------------------------------------------ |
| GRANTEE        | 被授予权限的用户的名称。                                      |
| TABLE_CATALOG  | 包含列的表所属的目录的名称。此值始终为`def`。               |
| TABLE_SCHEMA   | 包含列的表所属的数据库的名称。                               |
| TABLE_NAME     | 包含列的表的名称。                                            |
| COLUMN_NAME    | 列的名称。                                                    |
| PRIVILEGE_TYPE | 授予的权限类型。该值可以是列级别可授予的任何权限。每行列出单个权限，因此每个授予方拥有的列权限都有一行。 |
| IS_GRANTABLE   | 如果用户拥有`GRANT OPTION`权限则为`YES`，否则为`NO`。输出不会将`GRANT OPTION`列为`PRIVILEGE_TYPE='GRANT OPTION'`的独立行。 |