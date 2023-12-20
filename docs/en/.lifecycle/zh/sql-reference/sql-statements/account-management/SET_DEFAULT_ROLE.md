---
displayed_sidebar: English
---

# 设置默认角色

## 描述

设置用户连接到服务器时默认激活的角色。

该命令从 v3.0 版本开始支持。

## 语法

```SQL
-- 将指定角色设置为默认角色。
SET DEFAULT ROLE <role_name>[,<role_name>,..] TO <user_identity>;
-- 将用户的所有角色，包括将来分配给该用户的角色，设置为默认角色。
SET DEFAULT ROLE ALL TO <user_identity>;
-- 不设置默认角色，但用户登录后仍启用 public 角色。
SET DEFAULT ROLE NONE TO <user_identity>; 
```

## 参数

`role_name`：角色名称

`user_identity`：用户标识

## 使用说明

个人用户可以为自己设置默认角色。`user_admin`可以为其他用户设置默认角色。在执行此操作之前，请确保用户已经被分配了这些角色。

您可以使用 [SHOW GRANTS](SHOW_GRANTS.md) 查询用户的角色。

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

示例 1：为用户 `test` 设置 `db_admin` 和 `user_admin` 为默认角色。

```SQL
SET DEFAULT ROLE db_admin, user_admin TO test;
```

示例 2：为用户 `test` 设置所有角色，包括将来分配给该用户的角色，为默认角色。

```SQL
SET DEFAULT ROLE ALL TO test;
```

示例 3：清除用户 `test` 的所有默认角色。

```SQL
SET DEFAULT ROLE NONE TO test;
```