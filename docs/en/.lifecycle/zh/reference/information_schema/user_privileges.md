---
displayed_sidebar: English
---

# 用户权限

`用户权限` 提供有关用户权限的信息。

`用户权限` 中提供了以下字段：

| **字段**      | **描述**                                              |
| -------------- | ------------------------------------------------------------ |
| GRANTEE        | 被授予权限的用户的名称。      |
| TABLE_CATALOG  | 目录的名称。此值始终为 `def`。         |
| PRIVILEGE_TYPE | 被授予的权限类型。该值可以是可以在全局级别授予的任何权限。 |
| IS_GRANTABLE   | 如果用户具有`GRANT OPTION`权限则为`YES`，否则为`NO`。输出中不会将`GRANT OPTION`列为单独的行，其`PRIVILEGE_TYPE`为`GRANT OPTION`。 |
