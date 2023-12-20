---
displayed_sidebar: English
---

# 列权限

column_privileges 表用于标识当前启用角色授予或被授予的所有列级别权限。

column_privileges 表中包含以下字段：

|字段|描述|
|---|---|
|GRANTEE|被授予权限的用户的名称。|
|TABLE_CATALOG|包含该列的表所属的目录的名称。该值始终是默认值。|
|TABLE_SCHEMA|包含该列的表所属的数据库的名称。|
|TABLE_NAME|包含该列的表的名称。|
|COLUMN_NAME|列的名称。|
|PRIVILEGE_TYPE|授予的权限。该值可以是可以在列级别授予的任何权限。每行列出一个权限，因此被授予者拥有每列一行的权限。|
|IS_GRANTABLE|YES 如果用户具有 GRANT OPTION 权限，否则为 NO。输出不会将 GRANT OPTION 列为具有 PRIVILEGE_TYPE='GRANT OPTION' 的单独行。|
