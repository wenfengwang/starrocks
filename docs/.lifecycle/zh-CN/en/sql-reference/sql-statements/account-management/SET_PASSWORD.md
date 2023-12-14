---
displayed_sidebar: "Chinese"
---

# 设置密码

## 描述

### 语法

```SQL
SET PASSWORD [FOR 用户身份] =
[PASSWORD('纯文本密码')]|['哈希密码']
```

SET PASSWORD 命令可用于更改用户登录密码。如果字段 [FOR 用户身份] 不存在，将修改当前用户的密码。

```纯文本
请注意，用户身份必须与使用 CREATE USER 创建用户时指定的用户身份完全匹配。否则，用户将被报告为不存在。如果未指定用户身份，则当前用户是'用户名'@'ip'，这可能不匹配任何用户身份。当前用户可以通过 SHOW GRANTS 查看。          
```

PASSWORD() 输入明文密码，而直接使用字符串需要传输加密密码。

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
    SET PASSWORD FOR 'jack'@['domain'] = '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9'
    ```