---
displayed_sidebar: English
---

# 表权限

table_privileges 提供关于表权限的信息。

table_privileges 包含以下字段：

|字段|描述|
|---|---|
|GRANTEE|被授予权限的用户的名称。|
|TABLE_CATALOG|表所属目录的名称。该值始终是默认值。|
|TABLE_SCHEMA|表所属的数据库的名称。|
|TABLE_NAME|表的名称。|
|PRIVILEGE_TYPE|授予的权限。该值可以是可在表级别授予的任何权限。|
|IS_GRANTABLE|YES 如果用户具有 GRANT OPTION 权限，否则为 NO。输出不会将 GRANT OPTION 列为具有 PRIVILEGE_TYPE='GRANT OPTION' 的单独行。|
