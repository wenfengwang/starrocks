---
displayed_sidebar: "中文"
---

# 创建用户

## 描述

创建StarRocks用户。在StarRocks中，“user_identity”唯一标识用户。

### 语法

```SQL
CREATE USER <user_identity> [auth_option] [DEFAULT ROLE <role_name>[, <role_name>, ...]]
```

## 参数

- `user_identity` 由两部分组成，“user_name”和“host”，格式为`username@'userhost'`。对于“host”部分，可以使用`%`进行模糊匹配。如果没有指定“host”，则默认使用“%”，表示用户可以从任何主机连接到StarRocks。

- `auth_option` 指定认证方法。目前支持三种认证方法：StarRocks本机密码，mysql_native_password和“authentication_ldap_simple”。StarRocks本机密码在逻辑上与mysql_native_password相同，但在语法上略有不同。一个用户标识只能使用一种认证方法。

    ```SQL
    auth_option: {
        IDENTIFIED BY 'auth_string'
        IDENTIFIED WITH mysql_native_password BY 'auth_string'
        IDENTIFIED WITH mysql_native_password AS 'auth_string'
        IDENTIFIED WITH authentication_ldap_simple AS 'auth_string'
        
    }
    ```

    | **认证方法**          | **用于创建用户的密码** | **用于登录的密码** |
    | -------------------- | ---------------------- | -------------------- |
    | 本机密码             | 明文或密文             | 明文                 |
    | `mysql_native_password BY`   | 明文             | 明文                  |
    | `mysql_native_password WITH` | 密文             | 明文                  |
    | `authentication_ldap_simple` | 明文             | 明文                  |

> 注意：StarRocks在存储用户密码之前对其进行加密。

- `DEFAULT ROLE <role_name>[, <role_name>, ...]`：如果指定了此参数，角色将自动分配给用户，并在用户登录时默认激活。如果未指定，则该用户没有任何特权。确保所有指定的角色已存在。

## 示例

示例1：使用明文密码创建用户，未指定host，等效于`jack@'%'`。

```SQL
CREATE USER 'jack' IDENTIFIED BY '123456';
```

示例2：使用明文密码创建用户，并允许用户从`'172.10.1.10'`登录。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH mysql_native_password BY '123456';
```

示例3：使用密文密码创建用户，并允许用户从`'172.10.1.10'`登录。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED BY PASSWORD '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9';
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH mysql_native_password AS '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9';
```

> 注意：您可以使用password()函数获取加密的密码。

示例4：允许从域名'example_domain'登录的用户。

```SQL
CREATE USER 'jack'@['example_domain'] IDENTIFIED BY '123456';
```

示例5：使用LDAP认证创建用户。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH authentication_ldap_simple;
```

示例6：使用LDAP认证创建用户，并指定LDAP中用户的Distinguished Name（DN）。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH authentication_ldap_simple AS 'uid=jack,ou=company,dc=example,dc=com';
```

示例7：允许从'192.168'子网登录的用户，并将`db_admin`和`user_admin`设置为用户的默认角色。

```SQL
CREATE USER 'jack'@'192.168.%' DEFAULT ROLE db_admin, user_admin;
```