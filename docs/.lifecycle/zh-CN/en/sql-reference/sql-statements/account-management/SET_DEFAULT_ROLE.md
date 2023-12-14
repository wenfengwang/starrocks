---
displayed_sidebar: "Chinese"
---

# 设置默认角色

## 描述

设置用户连接到服务器时默认激活的角色。

此命令从v3.0版本开始支持。

## 语法

```SQL
-- 将指定角色设置为默认角色。
SET DEFAULT ROLE <role_name>[,<role_name>,..] TO <user_identity>;
-- 将用户的所有角色，包括将分配给该用户的角色，设置为默认角色。
SET DEFAULT ROLE ALL TO <user_identity>;
-- 在用户登录后仍然启用公共角色，但没有设置默认角色。
SET DEFAULT ROLE NONE TO <user_identity>;
```

## 参数

`role_name`: 角色名称

`user_identity`: 用户标识

## 使用说明

个人用户可以为自己设置默认角色。`user_admin` 可以为其他用户设置默认角色。在执行此操作之前，请确保用户已被分配这些角色。

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

示例 1：将角色 `db_admin` 和 `user_admin` 设置为用户 `test` 的默认角色。

```SQL
SET DEFAULT ROLE db_admin TO test;
```

示例 2：将用户 `test` 的所有角色，包括将分配给该用户的角色，设置为默认角色。

```SQL
SET DEFAULT ROLE ALL TO test;
```

示例 3：清除用户 `test` 的所有默认角色。

```SQL
SET DEFAULT ROLE NONE TO test;
```