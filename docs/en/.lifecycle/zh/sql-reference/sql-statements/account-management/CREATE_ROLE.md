---
displayed_sidebar: English
---

# 创建角色

## 描述

创建一个角色。创建角色后，您可以向该角色授予权限，然后将此角色分配给用户或其他角色。这样一来，与该角色关联的权限就会传递给用户或角色。

只有具有`user_admin`角色或`GRANT`权限的用户才能创建角色。

## 语法

```sql
CREATE ROLE <role_name>
```

## 参数

`role_name`：角色的名称。命名约定：

- 只能包含数字（0-9）、字母或下划线（_），并且必须以字母开头。
- 长度不能超过64个字符。

请注意，创建的角色名称不能与[系统定义的角色](../../../administration/privilege_overview.md#system-defined-roles)相同。

## 例子

创建一个角色。

```sql
CREATE ROLE role1;
```

## 引用

- [授予](GRANT.md)
- [显示角色](SHOW_ROLES.md)
- [删除角色](DROP_ROLE.md)
