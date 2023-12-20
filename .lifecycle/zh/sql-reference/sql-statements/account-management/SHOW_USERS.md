---
displayed_sidebar: English
---

# 显示用户

## 说明

展示系统中所有的用户。这里提到的用户指的是用户身份，而非用户名。关于用户身份的详细信息，请参考[CREATE USER](CREATE_USER.md)部分。该命令从 v3.0 版本开始支持。

您可以使用 `SHOW GRANTS FOR \\u003cuser_identity\\u003e;` 来查看特定用户的权限。更多信息，请查看 [SHOW GRANTS](SHOW_GRANTS.md)。

> 注意：只有 user_admin 角色才能执行此语句。

## 语法

```SQL
SHOW USERS
```

返回字段：

|字段|描述|
|---|---|
|用户|用户身份。|

## 示例

展示系统中的所有用户。

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

## 参考资料

[创建用户](CREATE_USER.md)，[修改用户](ALTER_USER.md)，[删除用户](DROP_USER.md)
