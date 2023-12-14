---
displayed_sidebar: "Chinese"
---

# 设置角色

## 描述

激活角色，以及其所有关联的权限和嵌套角色，用于当前会话。激活角色后，用户可以使用此角色执行操作。

 运行此命令后，您可以运行 `select is_role_in_session("<role_name>");` 来验证该角色是否在当前会话中处于活动状态。

此命令从 v3.0 版本开始支持。

## 语法

```SQL
-- 激活特定角色并以该角色执行操作。
SET ROLE <role_name>[,<role_name>,..];
-- 激活除特定角色外的用户的所有角色。
SET ROLE ALL EXCEPT <role_name>[,<role_name>,..]; 
-- 激活用户的所有角色。
SET ROLE ALL;
```

## 参数

`role_name`: 角色名

## 用法注意事项

用户只能激活已分配给他们的角色。

您可以使用 [SHOW GRANTS](./SHOW_GRANTS.md) 查询用户的角色。

您可以使用 `SELECT CURRENT_ROLE()` 查询当前用户的活动角色。有关更多信息，请参阅 [current_role](../../sql-functions/utility-functions/current_role.md).

## 示例

查询当前用户的所有角色。

```SQL
SHOW GRANTS;
+--------------+---------+----------------------------------------------+
| UserIdentity | Catalog | Grants                                       |
+--------------+---------+----------------------------------------------+
| 'test'@'%'   | NULL    | GRANT 'db_admin', 'user_admin' TO 'test'@'%' |
+--------------+---------+----------------------------------------------+
```

激活 `db_admin` 角色。

```SQL
SET ROLE db_admin;
```

查询当前用户的活动角色。

```SQL
SELECT CURRENT_ROLE();
+--------------------+
| CURRENT_ROLE()     |
+--------------------+
| db_admin           |
+--------------------+
```

## 参考

- [CREATE ROLE](CREATE_ROLE.md): 创建角色。
- [GRANT](GRANT.md): 将角色分配给用户或其他角色。
- [ALTER USER](ALTER_USER.md): 修改角色。
- [SHOW ROLES](SHOW_ROLES.md): 显示系统中的所有角色。
- [current_role](../../sql-functions/utility-functions/current_role.md): 显示当前用户的角色。
- [is_role_in_session](../../sql-functions/utility-functions/is_role_in_session.md): 验证当前会话中角色（或嵌套角色）是否处于活动状态。
- [DROP ROLE](DROP_ROLE.md): 删除角色。