---
displayed_sidebar: English
---

# 设置默认角色

## 说明

当用户连接到服务器时，设置将被默认激活的角色。

该命令从 v3.0 版本开始支持。

## 语法

```SQL
-- Set specified roles as default roles.
SET DEFAULT ROLE <role_name>[,<role_name>,..] TO <user_identity>;
-- Set all roles of the user, including roles that will be assigned to this user, as default roles. 
SET DEFAULT ROLE ALL TO <user_identity>;
-- No default role is set but the public role is still enabled after a user login. 
SET DEFAULT ROLE NONE TO <user_identity>; 
```

## 参数

role_name：角色名

user_identity：用户标识

## 使用须知

个别用户可以为自己设置默认角色。user_admin 可以为其他用户设置默认角色。在执行此操作之前，请确保用户已经被授予了这些角色。

您可以使用 [SHOW GRANTS](SHOW_GRANTS.md) 命令查询用户的角色。

## 示例

查询当前用户的角色。

```SQL
SHOW GRANTS FOR test;
+--------------+---------+----------------------------------------------+
| UserIdentity | Catalog | Grants                                       |
+--------------+---------+----------------------------------------------+
| 'test'@'%'   | NULL    | GRANT 'db_admin', 'user_admin' TO 'test'@'%' |
+--------------+---------+----------------------------------------------+
```

示例 1：为用户 test 设置 db_admin 和 user_admin 为默认角色。

```SQL
SET DEFAULT ROLE db_admin TO test;
```

示例 2：为用户 test 设置包括未来将被授予的角色在内的所有角色作为默认角色。

```SQL
SET DEFAULT ROLE ALL TO test;
```

示例 3：清除用户 test 的所有默认角色。

```SQL
SET DEFAULT ROLE NONE TO test;
```
