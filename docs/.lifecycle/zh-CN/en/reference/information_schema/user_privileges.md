---
displayed_sidebar: "Chinese"
---

# 用户权限

`user_privileges`提供有关用户权限的信息。

以下字段在`user_privileges`中提供：

| **字段**         | **描述**                                                    |
| --------------- | ------------------------------------------------------------ |
| GRANTEE         | 被授予特权的用户的名称。                                      |
| TABLE_CATALOG   | 目录的名称。此值始终为`def`。                               |
| PRIVILEGE_TYPE  | 授予的特权。该值可以是在全局级别授予的任何特权。                |
| IS_GRANTABLE    | 如果用户具有`GRANT OPTION`特权，则为`是`，否则为`否`。输出不会将`GRANT OPTION`列为具有`PRIVILEGE_TYPE='GRANT OPTION'`的单独行。 |