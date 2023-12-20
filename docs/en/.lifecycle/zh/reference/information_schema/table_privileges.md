---
displayed_sidebar: English
---

# 表权限

`table_privileges` 提供有关表权限的信息。

`table_privileges` 中提供了以下字段：

|**字段**|**描述**|
|---|---|
|GRANTEE|被授予权限的用户名称。|
|TABLE_CATALOG|表所属的目录名称。此值始终为 `def`。|
|TABLE_SCHEMA|表所属的数据库名称。|
|TABLE_NAME|表的名称。|
|PRIVILEGE_TYPE|被授予的权限类型。该值可以是任何可在表级别授予的权限。|
|IS_GRANTABLE|如果用户拥有 `GRANT OPTION` 权限则为 `YES`，否则为 `NO`。输出不会以 `PRIVILEGE_TYPE='GRANT OPTION'` 作为单独的行列出 `GRANT OPTION`。|