---
displayed_sidebar: "中文"
---

# schema_privileges

`schema_privileges` 提供关于数据库权限的信息。

`schema_privileges` 提供以下字段：

| 字段           | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| GRANTEE        | 被授权用户的名称。                                           |
| TABLE_CATALOG  | 模式所属的目录名称。该值始终为 def。                         |
| TABLE_SCHEMA   | 模式的名称。                                                 |
| PRIVILEGE_TYPE | 授予的权限类型。每一行列出一种权限，因此每个由被授权用户拥有的模式权限都对应一行记录。 |
| IS_GRANTABLE   | 如果用户具有 GRANT OPTION 权限，则为 YES；否则为 NO。输出不会将 GRANT OPTION 作为独立行列出，而是表现为 PRIVILEGE_TYPE='GRANT OPTION'。 |