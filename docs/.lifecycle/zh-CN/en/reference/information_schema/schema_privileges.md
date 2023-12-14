---
displayed_sidebar: "Chinese"
---

# schema_privileges

`schema_privileges`提供有关数据库权限的信息。

`schema_privileges`提供以下字段：

| **字段**         | **描述**                                                            |
| --------------- | --------------------------------------------------------------------- |
| GRANTEE         | 被授予权限的用户的名称。                                              |
| TABLE_CATALOG   | 模式所属目录的名称。该值始终为`def`。                                     |
| TABLE_SCHEMA    | 模式的名称。                                                           |
| PRIVILEGE_TYPE  | 授予的权限。每行列出单个权限，因此每个模式权限由被授权者持有一行。                         |
| IS_GRANTABLE    | 如果用户拥有“GRANT OPTION”权限则为`YES`，否则为`NO`。输出不会将`GRANT OPTION`列为一个单独的行，其`PRIVILEGE_TYPE='GRANT OPTION'`。  |