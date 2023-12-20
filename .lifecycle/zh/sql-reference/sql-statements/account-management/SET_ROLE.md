---
displayed_sidebar: English
---

# 设定角色

## 描述

此命令用于在当前会话中激活角色以及其关联的所有权限和嵌套角色。一旦角色被激活，用户便可以利用此角色执行操作。

执行该命令后，您可以运行 select is_role_in_session("<role_name>"); 来检查该角色在当前会话中是否已激活。

此命令自 v3.0 版本起提供支持。

## 语法

```SQL
-- Active specific roles and perform operations as this role.
SET ROLE <role_name>[,<role_name>,..];
-- Activate all roles of a user, except for specific roles.
SET ROLE ALL EXCEPT <role_name>[,<role_name>,..]; 
-- Activate all roles of a user.
SET ROLE ALL;
```

## 参数

role_name：角色名

## 使用须知

用户只能激活已经分配给他们的角色。

您可以通过[SHOW GRANTS](./SHOW_GRANTS.md)命令查询用户的角色。

您可以通过 `SELECT CURRENT_ROLE()` 命令查询当前用户的激活角色。欲了解更多信息，请参见[current_role](../../sql-functions/utility-functions/current_role.md)。

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

激活 db_admin 角色。

```SQL
SET ROLE db_admin;
```

查询当前用户的激活角色。

```SQL
SELECT CURRENT_ROLE();
+--------------------+
| CURRENT_ROLE()     |
+--------------------+
| db_admin           |
+--------------------+
```

## 参考资料

- [CREATE ROLE](CREATE_ROLE.md)：创建一个角色。
- [GRANT](GRANT.md)：用于将角色分配给用户或其他角色。
- [修改用户](ALTER_USER.md)：修改角色。
- [SHOW ROLES](SHOW_ROLES.md)：显示系统中的所有角色。
- [current_role](../../sql-functions/utility-functions/current_role.md)：显示当前用户的角色。
- [is_role_in_session](../../sql-functions/utility-functions/is_role_in_session.md)：用于验证角色（或嵌套角色）是否在当前会话中激活。
- [DROP ROLE](DROP_ROLE.md)：删除一个角色。
