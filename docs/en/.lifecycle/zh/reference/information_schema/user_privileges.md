---
displayed_sidebar: English
---

# 用户权限

`user_privileges` 提供有关用户权限的信息。

`user_privileges` 中提供了以下字段：

|**字段**|**描述**|
|---|---|
|GRANTEE|被授予权限的用户名称。|
|TABLE_CATALOG|目录的名称。此值始终为 `def`。|
|PRIVILEGE_TYPE|被授予的权限。该值可以是任何可以在全局级别授予的权限。|
|IS_GRANTABLE|如果用户具有 `GRANT OPTION` 权限，则为 `YES`，否则为 `NO`。输出不会以 `PRIVILEGE_TYPE='GRANT OPTION'` 的形式单独列出 `GRANT OPTION`。|