---
displayed_sidebar: "Chinese"
---

# 修改用户

## 描述

修改用户信息，如密码、认证方法或默认角色。

>个人用户可使用此命令修改自己的信息。`user_admin` 可使用此命令修改其他用户的信息。

## 语法

```SQL
ALTER USER user_identity [auth_option] [default_role]
```

## 参数

- `user_identity` 由“user_name”和“host”两部分组成，格式为 `username@'userhost'`。对于“host”部分，您可以使用`％`进行模糊匹配。如果未指定“host”，则默认使用`％`，表示用户可以从任何主机连接到 StarRocks。

- `auth_option` 指定认证方法。目前支持三种认证方法：StarRocks 本机密码、mysql_native_password 和“authentication_ldap_simple”。StarRocks 本机密码在逻辑上与 mysql_native_password 相同，但在语法上略有不同。一个用户身份只能使用一种认证方法。您可以使用 ALTER USER 来修改用户的密码和认证方法。

    ```SQL
    auth_option: {
        IDENTIFIED BY 'auth_string'
        IDENTIFIED WITH mysql_native_password BY 'auth_string'
        IDENTIFIED WITH mysql_native_password AS 'auth_string'
        IDENTIFIED WITH authentication_ldap_simple AS 'auth_string'
        
    }
    ```

    | **认证方法**         | **用户创建时的密码**    | **登录密码**  |
    | --------------------- | ----------------------- | ------------- |
    | 本机密码             | 明文或密文             | 明文          |
    | `mysql_native_password BY` | 明文               | 明文          |
    | `mysql_native_password WITH` | 密文             | 明文          |
    | `authentication_ldap_simple` | 明文              | 明文          |

> 注意：StarRocks 在存储用户密码之前会对其进行加密。

- `DEFAULT ROLE`

   ```SQL
    -- 将指定角色设置为默认角色。
    DEFAULT ROLE <role_name>[, <role_name>, ...]
    -- 设置用户的所有角色，包括将分配给该用户的角色，作为默认角色。
    DEFAULT ROLE ALL
    -- 不设置默认角色，但是在用户登录后仍然启用公共角色。
    DEFAULT ROLE NONE
    ```

  在运行 ALTER USER 设置默认角色之前，请确保所有角色已分配给用户。角色在用户再次登录后会自动激活。

## 示例

示例 1：将用户密码更改为明文密码。

```SQL
ALTER USER 'jack' IDENTIFIED BY '123456';
ALTER USER jack@'172.10.1.10' IDENTIFIED WITH mysql_native_password BY '123456';
```

示例 2：将用户密码更改为密文密码。

```SQL
ALTER USER jack@'172.10.1.10' IDENTIFIED BY PASSWORD '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9';
ALTER USER jack@'172.10.1.10' IDENTIFIED WITH mysql_native_password AS '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9';
```

> 您可以使用 password() 函数获取加密密码。

示例 3：将认证方法更改为 LDAP。

```SQL
ALTER USER jack@'172.10.1.10' IDENTIFIED WITH authentication_ldap_simple;
```

示例 4：将认证方法更改为 LDAP 并指定用户在 LDAP 中的唯一名称（DN）。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH authentication_ldap_simple AS 'uid=jack,ou=company,dc=example,dc=com';
```

示例 5：将用户的默认角色更改为 `db_admin` 和 `user_admin`。注意用户必须已分配这两个角色。

```SQL
ALTER USER 'jack'@'192.168.%' DEFAULT ROLE db_admin, user_admin;
```

示例 6：将用户的所有角色，包括将分配给该用户的角色，设置为默认角色。

```SQL
ALTER USER 'jack'@'192.168.%' DEFAULT ROLE ALL;
```

示例 7：清除用户的所有默认角色。

```SQL
ALTER USER 'jack'@'192.168.%' DEFAULT ROLE NONE;
```

> 注意：默认情况下，用户仍然激活`public`角色。

## 参考

- [CREATE USER](CREATE_USER.md)
- [GRANT](GRANT.md)
- [SHOW USERS](SHOW_USERS.md)
- [DROP USER](DROP_USER.md)