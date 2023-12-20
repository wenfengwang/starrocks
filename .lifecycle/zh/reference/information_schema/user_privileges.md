---
displayed_sidebar: English
---

# 用户权限

user_privileges 提供了有关用户权限的信息。

在 user_privileges 中提供了以下字段：

|字段|描述|
|---|---|
|GRANTEE|被授予权限的用户的名称。|
|TABLE_CATALOG|目录的名称。该值始终是默认值。|
|PRIVILEGE_TYPE|授予的权限。该值可以是可以在全局级别授予的任何权限。|
|IS_GRANTABLE|YES 如果用户具有 GRANT OPTION 权限，否则为 NO。输出不会将 GRANT OPTION 列为具有 PRIVILEGE_TYPE='GRANT OPTION' 的单独行。|
