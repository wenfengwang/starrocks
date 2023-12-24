---
displayed_sidebar: English
---

# 设置密码

## 描述

### 语法

```SQL
SET PASSWORD [FOR user_identity] =
[PASSWORD('plain password')]|['hashed password']
```

SET PASSWORD 命令可用于更改用户的登录密码。如果字段 [对于user_identity] 不存在，则应修改当前用户的密码。

```plain text
请注意，user_identity 必须与使用 CREATE USER 创建用户时指定的 user_identity 完全匹配。否则，用户将被报告为不存在。如果未指定 user_identity，则当前用户是 'username'@'ip，这可能不匹配任何 user_identity。可以通过 SHOW GRANTS 查看当前用户。
```

PASSWORD() 输入明文密码，而直接使用字符串则需要传输加密密码。

修改其他用户的密码需要管理员权限。

## 例子

1. 修改当前用户的密码

    ```SQL
    SET PASSWORD = PASSWORD('123456')
    SET PASSWORD = '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9'
    ```

2. 修改指定用户的密码

    ```SQL
    SET PASSWORD FOR 'jack'@'192.%' = PASSWORD('123456')
    SET PASSWORD FOR 'jack'@['domain'] = '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9'
    ```
