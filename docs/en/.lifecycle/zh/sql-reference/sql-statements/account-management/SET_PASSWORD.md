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

SET PASSWORD 命令可以用来更改用户的登录密码。如果字段 [FOR user_identity] 不存在，则会修改当前用户的密码。

```plain
请注意，user_identity 必须与使用 CREATE USER 创建用户时指定的 user_identity 完全匹配。否则，系统会报告用户不存在。如果没有指定 user_identity，当前用户是 'username'@'ip'，这可能与任何 user_identity 都不匹配。可以通过 SHOW GRANTS 查看当前用户。 
```

PASSWORD() 输入的是明文密码，而直接使用字符串则需要传递加密密码。

修改其他用户的密码需要管理员权限。

## 示例

1. 修改当前用户的密码

   ```SQL
   SET PASSWORD = PASSWORD('123456')
   SET PASSWORD = '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9'
   ```

2. 修改指定用户的密码

   ```SQL
   SET PASSWORD FOR 'jack'@'192.%' = PASSWORD('123456')
   SET PASSWORD FOR 'jack'@'192.%' = '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9'
   ```