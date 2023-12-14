---
displayed_sidebar: "Chinese"
---

# 显示用户

## 描述

显示系统中的所有用户。这里提到的用户是用户标识，而不是用户名。有关用户标识的更多信息，请参阅[创建用户](CREATE_USER.md)。此命令支持从v3.0开始。

您可以使用 `SHOW GRANTS FOR <user_identity>;` 查看特定用户的权限。有关更多信息，请参阅[SHOW GRANTS](SHOW_GRANTS.md)。

> 注意：只有 `user_admin` 角色才能执行此语句。

## 语法

```SQL
SHOW USERS
```

返回字段：

| **字段** | **描述**         |
| -------- | ----------------- |
| User     | 用户标识。        |

## 示例

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

## 参考

[创建用户](CREATE_USER.md), [修改用户](ALTER_USER.md), [删除用户](DROP_USER.md)