---
displayed_sidebar: English
---

# 创建角色

## 描述

创建一个角色。在角色被创建之后，你可以授予该角色权限，然后将其分配给用户或其他角色。这样，与该角色相关联的权限便会传递给这些用户或角色。

只有拥有user_admin角色或GRANT权限的用户才有权限创建角色。

## 语法

```sql
CREATE ROLE <role_name>
```

## 参数

role_name：角色的名称。命名规则：

- 它只能包含数字（0-9）、字母或下划线（_），并且必须以字母开头。
- 长度不能超过64个字符。

请注意，创建的角色名称不能与[系统预定义的角色名称](../../../administration/privilege_overview.md#system-defined-roles)相同。

## 示例

创建一个角色。

```sql
CREATE ROLE role1;
```

## 参考资料

- [GRANT](GRANT.md)（授权）
- [SHOW ROLES](SHOW_ROLES.md)（显示角色）
- [DROP ROLE](DROP_ROLE.md)（删除角色）
