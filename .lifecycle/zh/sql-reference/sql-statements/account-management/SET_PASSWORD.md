---
displayed_sidebar: English
---

# 设置密码

## 说明

### 语法

```SQL
SET PASSWORD [FOR user_identity] =
[PASSWORD('plain password')]|['hashed password']
```

SET PASSWORD 命令用于更改用户的登录密码。如果没有指定 [FOR user_identity] 字段，那么将修改当前用户的密码。

```plain
Please note that the user_identity must match exactly the user_identity specified when creating a user by using CREATE USER. Otherwise, the user will be reported as non-existent. If user_identity is not specified, the current user is 'username'@'ip, which may not match any user_identity. The current user can be viewed through SHOW GRANTS. 
```

PASSWORD() 函数接受明文密码作为输入，而直接使用字符串时，则需提供加密过的密码。

修改其他用户的密码需要管理员权限。

## 示例

1. 修改当前用户的密码

   ```SQL
   SET PASSWORD = PASSWORD('123456')
   SET PASSWORD = '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9'
   ```

2. 为指定用户修改密码

   ```SQL
   SET PASSWORD FOR 'jack'@'192.%' = PASSWORD('123456')
   SET PASSWORD FOR 'jack'@['domain'] = '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9'
   ```
