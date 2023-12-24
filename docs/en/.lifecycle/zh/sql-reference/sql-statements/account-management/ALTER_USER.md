---
displayed_sidebar: English
---

# 修改用户

## 描述

修改用户信息，例如密码、身份验证方法或默认角色。

> 单个用户可以使用此命令自行修改信息。`user_admin` 可以使用此命令修改其他用户的信息。

## 语法

```SQL
ALTER USER user_identity [auth_option] [default_role]
```

## 参数

- `user_identity` 由“user_name”和“host”两部分组成，格式为 `username@'userhost'`。 对于“host”部分，您可以使用 `%` 进行模糊匹配。如果未指定 “host”，则默认使用 “%”，表示用户可以从任何主机连接到 StarRocks。

- `auth_option` 指定身份验证方法。目前支持三种身份验证方法：StarRocks原生密码、mysql_native_password 和 "authentication_ldap_simple"。StarRocks原生密码在逻辑上与mysql_native_password相同，但在语法上略有不同。一个用户标识只能使用一种身份验证方法。您可以使用ALTER USER来修改用户的密码和身份验证方法。

    ```SQL
    auth_option: {
        IDENTIFIED BY 'auth_string'
        IDENTIFIED WITH mysql_native_password BY 'auth_string'
        IDENTIFIED WITH mysql_native_password AS 'auth_string'
        IDENTIFIED WITH authentication_ldap_simple AS 'auth_string'
        
    }
    ```

    | **身份验证方法**    | **用于用户创建的密码** | **登录密码** |
    | ---------------------------- | ------------------------------ | ---------------------- |
    | 原生密码              | 明文或密文        | 明文              |
    | `mysql_native_password BY`   | 明文                      | 明文              |
    | `mysql_native_password WITH` | 密文                     | 明文              |
    | `authentication_ldap_simple` | 明文                      | 明文              |

> 注意：StarRocks在存储用户密码之前会对其进行加密。

- `DEFAULT ROLE`

   ```SQL
    -- 将指定角色设置为默认角色。
    DEFAULT ROLE <role_name>[, <role_name>, ...]
    -- 将用户的所有角色，包括将分配给此用户的角色，设置为默认角色。 
    DEFAULT ROLE ALL
    -- 用户登录后仍启用公共角色，但未设置默认角色。 
    DEFAULT ROLE NONE
    ```

  在运行ALTER USER设置默认角色之前，请确保已将所有角色分配给用户。用户再次登录后，这些角色将自动激活。

## 例子

示例 1：将用户的密码更改为明文密码。

```SQL
ALTER USER 'jack' IDENTIFIED BY '123456';
ALTER USER jack@'172.10.1.10' IDENTIFIED WITH mysql_native_password BY '123456';
```

示例 2：将用户的密码更改为密文密码。

```SQL
ALTER USER jack@'172.10.1.10' IDENTIFIED BY PASSWORD '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9';
ALTER USER jack@'172.10.1.10' IDENTIFIED WITH mysql_native_password AS '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9';
```

> 您可以使用password()函数获取加密密码。

示例 3：将身份验证方法更改为LDAP。

```SQL
ALTER USER jack@'172.10.1.10' IDENTIFIED WITH authentication_ldap_simple;
```

示例 4：将身份验证方法更改为LDAP，并指定LDAP中用户的可分辨名称（DN）。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH authentication_ldap_simple AS 'uid=jack,ou=company,dc=example,dc=com';
```

示例 5：将用户的默认角色更改为 `db_admin` 和 `user_admin`。请注意，用户必须已被分配这两个角色。

```SQL
ALTER USER 'jack'@'192.168.%' DEFAULT ROLE db_admin, user_admin;
```

示例 6：将用户的所有角色，包括将分配给此用户的角色，设置为默认角色。

```SQL
ALTER USER 'jack'@'192.168.%' DEFAULT ROLE ALL;
```

示例 7：清除用户的所有默认角色。

```SQL
ALTER USER 'jack'@'192.168.%' DEFAULT ROLE NONE;
```

> 注意：默认情况下，用户仍会激活`public`角色。

## 引用

- [创建用户](CREATE_USER.md)
- [授权](GRANT.md)
- [显示用户](SHOW_USERS.md)
- [删除用户](DROP_USER.md)
