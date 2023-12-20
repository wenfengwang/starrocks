---
displayed_sidebar: English
---

# 模式权限

schema_privileges 提供了有关数据库权限的信息。

在 schema_privileges 中，提供了以下字段：

|字段|描述|
|---|---|
|GRANTEE|被授予权限的用户的名称。|
|TABLE_CATALOG|模式所属目录的名称。该值始终是默认值。|
|TABLE_SCHEMA|架构的名称。|
|PRIVILEGE_TYPE|授予的权限。每行列出一个权限，因此受让人持有的每个架构权限有一行。|
|IS_GRANTABLE|YES 如果用户具有 GRANT OPTION 权限，否则为 NO。输出不会将 GRANT OPTION 列为具有 PRIVILEGE_TYPE='GRANT OPTION' 的单独行。|
