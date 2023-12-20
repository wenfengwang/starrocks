---
displayed_sidebar: English
---

# 显示认证信息

## 描述

展示当前用户或当前集群中所有用户的认证信息。每个用户都有权限查看自己的认证信息。只有拥有全局GRANT权限和user_admin角色的用户，才能查看所有用户或指定用户的认证信息。

## 语法

```SQL
SHOW [ALL] AUTHENTICATION [FOR USERNAME]
```

## 参数

|参数|必填|说明|
|---|---|---|
|ALL|否|如果指定该关键字，则返回当前集群中所有用户的认证信息。如果不指定该关键字，则仅返回当前用户的认证信息。|
|USERNAME|否|如果指定该参数，则可以查看指定用户的认证信息。如果不指定该参数，则只能查看当前用户的认证信息。|

## 输出

```SQL
+---------------+----------+-------------+-------------------+
| UserIdentity  | Password | AuthPlugin  | UserForAuthPlugin |
+---------------+----------+-------------+-------------------+
```

|字段|描述|
|---|---|
|用户身份|用户身份。|
|密码|是否使用密码登录StarRocks集群。是：使用密码。否：不使用密码。|
|AuthPlugin|用于身份验证的接口。有效值：MYSQL_NATIVE_PASSWORD、AUTHENTICATION_LDAP_SIMPLE 或 AUTHENTICATION_KERBEROS。如果没有使用接口，则返回 NULL。|
|UserForAuthPlugin|使用 LDAP 或 Kerberos 身份验证的用户的名称。如果未使用身份验证，则返回 NULL。|

## 示例

示例1：展示当前用户的认证信息。

```Plain
SHOW AUTHENTICATION;
+--------------+----------+------------+-------------------+
| UserIdentity | Password | AuthPlugin | UserForAuthPlugin |
+--------------+----------+------------+-------------------+
| 'root'@'%'   | No       | NULL       | NULL              |
+--------------+----------+------------+-------------------+
```

示例2：展示当前集群中所有用户的认证信息。

```Plain
SHOW ALL AUTHENTICATION;
+---------------+----------+-------------------------+-------------------+
| UserIdentity  | Password | AuthPlugin              | UserForAuthPlugin |
+---------------+----------+-------------------------+-------------------+
| 'root'@'%'    | Yes      | NULL                    | NULL              |
| 'chelsea'@'%' | No       | AUTHENTICATION_KERBEROS | HADOOP.COM        |
+---------------+----------+-------------------------+-------------------+
```

示例3：展示指定用户的认证信息。

```Plain
SHOW AUTHENTICATION FOR root;
+--------------+----------+------------+-------------------+
| UserIdentity | Password | AuthPlugin | UserForAuthPlugin |
+--------------+----------+------------+-------------------+
| 'root'@'%'   | Yes      | NULL       | NULL              |
+--------------+----------+------------+-------------------+
```
