---
displayed_sidebar: English
---

# 显示用户

## 描述

显示系统中的所有用户。这里提到的用户是用户身份，而不是用户名。有关用户身份的详细信息，请参阅 [创建用户](CREATE_USER.md)。此命令从 v3.0 版本开始支持。

您可以使用 `SHOW GRANTS FOR <user_identity>;` 来查看特定用户的权限。有关详细信息，请参阅 [显示授权](SHOW_GRANTS.md)。

> 注意：只有 `user_admin` 角色才能执行此语句。

## 语法

```SQL
SHOW USERS
```

返回字段：

| **字段** | **描述**    |
| --------- | ------------------ |
| 用户      | 用户身份。 |

## 例子

显示系统中的所有用户。

```SQL
mysql> SHOW USERS;
+-----------------+
| User            |
+-----------------+
| 'wybing5'@'%'   |
| 'root'@'%'      |
| 'admin'@'%'     |
| 'star'@'%'      |
| 'wybing_30'@'%' |
| 'simo'@'%'      |
| 'wybing1'@'%'   |
| 'wybing2'@'%'   |
+-----------------+
```

## 引用

[创建用户](CREATE_USER.md)、[更改用户](ALTER_USER.md)、[删除用户](DROP_USER.md)