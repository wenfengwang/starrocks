---
displayed_sidebar: English
---

# 创建角色

## 描述

创建一个角色。角色创建后，您可以授予该角色权限，然后将其分配给用户或其他角色。这样，与该角色相关联的权限便会传递给用户或角色。

只有拥有 `user_admin` 角色或 `GRANT` 权限的用户才能创建角色。

## 语法

```sql
CREATE ROLE <role_name>
```

## 参数

`role_name`：角色的名称。命名规则：

- 只能包含数字（0-9）、字母或下划线（_），并且必须以字母开头。
- 长度不能超过 64 个字符。

请注意，创建的角色名称不能与[系统定义的角色](../../../administration/privilege_overview.md#system-defined-roles)相同。

## 示例

创建一个角色。

```sql
CREATE ROLE role1;
```

## 参考资料

- [GRANT](GRANT.md)
- [SHOW ROLES](SHOW_ROLES.md)
- [DROP ROLE](DROP_ROLE.md)