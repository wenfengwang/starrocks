---
displayed_sidebar: English
---

# schema_privileges

`schema_privileges` 提供有关数据库权限的信息。

`schema_privileges` 中提供了以下字段：

|**字段**|**描述**|
|---|---|
|GRANTEE|被授予权限的用户名称。|
|TABLE_CATALOG|架构所属的目录名称。该值始终为 `def`。|
|TABLE_SCHEMA|架构的名称。|
|PRIVILEGE_TYPE|被授予的权限。每行列出一个权限，因此每个受让人持有的架构权限对应一行。|
|IS_GRANTABLE|如果用户具有 `GRANT OPTION` 权限则为 `YES`，否则为 `NO`。输出不会以 `PRIVILEGE_TYPE='GRANT OPTION'` 作为单独的行来列出 `GRANT OPTION`。|