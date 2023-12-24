---
displayed_sidebar: English
---
# schema_privileges

`schema_privileges` 提供有关数据库特权的信息。

`schema_privileges` 中提供了以下字段：

| **字段**      | **描述**                                              |
| -------------- | ------------------------------------------------------------ |
| GRANTEE        | 被授予权限的用户的名称。      |
| TABLE_CATALOG  | 架构所属的目录的名称。此值始终为 `def`。 |
| TABLE_SCHEMA   | 架构的名称。                                      |
| PRIVILEGE_TYPE | 授予的特权。每行列出一个权限，因此被授权者持有的每个架构权限都有一行。 |
| IS_GRANTABLE   | 如果用户具有`GRANT OPTION`特权，则为 `YES`，否则为 `NO`。输出不会列出`GRANT OPTION`作为单独的行，而是将其合并到`PRIVILEGE_TYPE='GRANT OPTION'`的行中。