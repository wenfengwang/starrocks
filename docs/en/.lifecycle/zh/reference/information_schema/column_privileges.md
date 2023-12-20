---
displayed_sidebar: English
---

# column_privileges

`column_privileges` 标识当前启用角色授予或由当前启用角色授予的所有列权限。

`column_privileges` 中提供以下字段：

|**字段**|**描述**|
|---|---|
|GRANTEE|权限被授予的用户名称。|
|TABLE_CATALOG|包含该列的表所属目录的名称。此值始终为 `def`。|
|TABLE_SCHEMA|包含该列的表所属数据库的名称。|
|TABLE_NAME|包含该列的表的名称。|
|COLUMN_NAME|列的名称。|
|PRIVILEGE_TYPE|被授予的权限。该值可以是任何可以在列级别授予的权限。每行列出一个权限，因此每个被授予者对每个列权限有一行记录。|
|IS_GRANTABLE|如果用户有 `GRANT OPTION` 权限则为 `YES`，否则为 `NO`。输出不会以 `PRIVILEGE_TYPE='GRANT OPTION'` 作为单独的行来列出 `GRANT OPTION`。|