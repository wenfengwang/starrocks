---
displayed_sidebar: English
---

# 创建用户

## 描述

创建 StarRocks 用户。在 StarRocks 中，“user_identity”唯一标识一个用户。

### 语法

```SQL
CREATE USER <user_identity> [auth_option] [DEFAULT ROLE <role_name>[, <role_name>, ...]]
```

## 参数

- `user_identity` 由“user_name”和“host”两部分组成，格式为 `username@'userhost'`。 对于“host”部分，您可以使用 `%` 进行模糊匹配。如果未指定 “host”，则默认使用 “%”，表示用户可以从任何主机连接到 StarRocks。

- `auth_option` 指定认证方法。目前支持三种认证方法：StarRocks 原生密码、mysql_native_password 和 authentication_ldap_simple。StarRocks 原生密码在逻辑上与 mysql_native_password 相同，但语法略有不同。一个用户标识只能使用一种认证方法。

    ```SQL
    auth_option: {
        IDENTIFIED BY 'auth_string'
        IDENTIFIED WITH mysql_native_password BY 'auth_string'
        IDENTIFIED WITH mysql_native_password AS 'auth_string'
        IDENTIFIED WITH authentication_ldap_simple AS 'auth_string'
        
    }
    ```

    | **认证方法**    | **用于创建用户的密码** | **登录密码** |
    | ---------------------------- | ------------------------------ | ---------------------- |
    | 本机密码              | 明文或密文        | 明文              |
    | `mysql_native_password BY`   | 明文                      | 明文              |
    | `mysql_native_password WITH` | 密文                     | 明文              |
    | `authentication_ldap_simple` | 明文                      | 明文              |

> 注意：StarRocks 在存储用户密码之前会对其进行加密。

- `DEFAULT ROLE <role_name>[, <role_name>, ...]`：如果指定了该参数，角色将自动分配给用户，并在用户登录时默认激活。如果未指定，此用户将没有任何权限。请确保指定的所有角色已存在。

## 例子

示例 1：使用纯文本密码创建用户，未指定主机，相当于 `jack@'%'`。

```SQL
CREATE USER 'jack' IDENTIFIED BY '123456';
```

示例 2：使用纯文本密码创建用户，并允许用户从  `'172.10.1.10'` 登录。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH mysql_native_password BY '123456';
```

示例 3：使用密文密码创建用户，并允许用户从  `'172.10.1.10'` 登录。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED BY PASSWORD '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9';
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH mysql_native_password AS '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9';
```

> 注意：您可以使用 password() 函数获取加密密码。

示例 4：创建允许从域名“example_domain”登录的用户。

```SQL
CREATE USER 'jack'@['example_domain'] IDENTIFIED BY '123456';
```

示例 5：创建使用 LDAP 认证的用户。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH authentication_ldap_simple;
```

示例 6：创建使用 LDAP 认证的用户，并在 LDAP 中指定用户的可分辨名称（DN）。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH authentication_ldap_simple AS 'uid=jack,ou=company,dc=example,dc=com';
```

示例 7：创建允许从 '192.168' 子网登录的用户，并将 `db_admin` 和 `user_admin` 设置为该用户的默认角色。

```SQL
CREATE USER 'jack'@'192.168.%' DEFAULT ROLE db_admin, user_admin;
```
