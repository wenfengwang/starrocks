---
displayed_sidebar: English
---

# SET ROLE

## 描述

激活当前会话中的角色及其所有关联的权限和嵌套角色。激活角色后，用户可以使用该角色执行操作。

执行此命令后，您可以运行 `select is_role_in_session("<role_name>");` 来验证该角色是否在当前会话中处于活跃状态。

该命令从 v3.0 版本开始支持。

## 语法

```SQL
-- 激活特定角色并以该角色身份执行操作。
SET ROLE <role_name>[,<role_name>,..];
-- 激活用户的所有角色，除了特定角色。
SET ROLE ALL EXCEPT <role_name>[,<role_name>,..]; 
-- 激活用户的所有角色。
SET ROLE ALL;
```

## 参数

`role_name`：角色名称

## 使用说明

用户只能激活已分配给他们的角色。

您可以使用 [SHOW GRANTS](./SHOW_GRANTS.md) 查询用户的角色。

您可以使用 `SELECT CURRENT_ROLE()` 查询当前用户的活跃角色。更多信息，请参见 [current_role](../../sql-functions/utility-functions/current_role.md)。

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

查询当前用户的活跃角色。

```SQL
SELECT CURRENT_ROLE();
+--------------------+
| CURRENT_ROLE()     |
+--------------------+
| db_admin           |
+--------------------+
```

## 参考资料

- [CREATE ROLE](CREATE_ROLE.md)：创建角色。
- [GRANT](GRANT.md)：将角色授权给用户或其他角色。
- [ALTER USER](ALTER_USER.md)：修改用户角色。
- [SHOW ROLES](SHOW_ROLES.md)：展示系统中的所有角色。
- [current_role](../../sql-functions/utility-functions/current_role.md)：展示当前用户的角色。
- [is_role_in_session](../../sql-functions/utility-functions/is_role_in_session.md)：验证角色（或嵌套角色）是否在当前会话中活跃。
- [DROP ROLE](DROP_ROLE.md)：删除角色。