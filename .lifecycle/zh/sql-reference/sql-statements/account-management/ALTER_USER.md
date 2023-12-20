---
displayed_sidebar: English
---

# 修改用户信息

## 描述

用于修改用户信息，如密码、认证方式或默认角色。

> 个人用户可以使用此命令修改自己的信息。用户管理员（user_admin）可以使用此命令修改其他用户的信息。

## 语法

```SQL
ALTER USER user_identity [auth_option] [default_role]
```

## 参数

- user_identity 由“用户名（user_name）”和“主机（host）”两部分组成，格式为 username@'userhost'。对于“主机（host）”部分，可以使用 % 进行模糊匹配。如果未指定“主机”，则默认使用“%”，这意味着用户可以从任何主机连接到 StarRocks。

- auth_option 指定认证方式。目前支持三种认证方式：StarRocks 原生密码、mysql_native_password 和“authentication_ldap_simple”。StarRocks 原生密码在逻辑上与 mysql_native_password 相同，但语法略有区别。一个用户身份只能使用一种认证方式。您可以使用 ALTER USER 来修改用户的密码和认证方式。

  ```SQL
  auth_option: {
      IDENTIFIED BY 'auth_string'
      IDENTIFIED WITH mysql_native_password BY 'auth_string'
      IDENTIFIED WITH mysql_native_password AS 'auth_string'
      IDENTIFIED WITH authentication_ldap_simple AS 'auth_string'
  
  }
  ```

  |身份验证方法|用户创建密码|登录密码|
|---|---|---|
  |本机密码|明文或ciphertext |纯文本|
  |mysql_native_password BY|明文|明文|
  | mysql_native_password with | ciphertext | plaintext |
  |authentication_ldap_simple|明文|明文|

> 注意：StarRocks 会在存储密码之前对其进行加密。

- 默认角色

  ```SQL
   -- Set specified roles as default roles.
   DEFAULT ROLE <role_name>[, <role_name>, ...]
   -- Set all roles of the user, including roles that will be assigned to this user, as default roles. 
   DEFAULT ROLE ALL
   -- No default role is set but the public role is still enabled after a user login. 
   DEFAULT ROLE NONE
  ```

  在使用 ALTER USER 命令设置默认角色之前，请确保所有角色已分配给用户。用户重新登录后，这些角色会自动激活。

## 示例

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

> 您可以使用 password() 函数来获取加密后的密码。

示例 3：将认证方式更改为 LDAP。

```SQL
ALTER USER jack@'172.10.1.10' IDENTIFIED WITH authentication_ldap_simple;
```

示例 4：将认证方式更改为 LDAP，并指定 LDAP 中用户的独特名称（DN）。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH authentication_ldap_simple AS 'uid=jack,ou=company,dc=example,dc=com';
```

示例 5：将用户的默认角色更改为 db_admin 和 user_admin。注意，用户必须已被分配这两个角色。

```SQL
ALTER USER 'jack'@'192.168.%' DEFAULT ROLE db_admin, user_admin;
```

示例 6：设置用户的所有角色，包括将作为默认角色分配给该用户的角色。

```SQL
ALTER USER 'jack'@'192.168.%' DEFAULT ROLE ALL;
```

示例 7：清除用户的所有默认角色。

```SQL
ALTER USER 'jack'@'192.168.%' DEFAULT ROLE NONE;
```

> 注意：默认情况下，公共角色仍然会为用户激活。

## 参考资料

- [CREATE USER](CREATE_USER.md)
- [GRANT](GRANT.md)
- [SHOW USERS](SHOW_USERS.md)
- [DROP USER](DROP_USER.md)
