---
displayed_sidebar: English
---

# 显示认证信息

## 描述

显示当前用户或当前集群中所有用户的认证信息。每个用户都有权限查看自己的认证信息。只有拥有全局 `GRANT` 权限和 `user_admin` 角色的用户才能查看所有用户或指定用户的认证信息。

## 语法

```SQL
SHOW [ALL] AUTHENTICATION [FOR USERNAME]
```

## 参数

|**参数**|**必填**|**描述**|
|---|---|---|
|ALL|否|如果指定了此关键字，则返回当前集群中所有用户的认证信息。如果未指定此关键字，则只返回当前用户的认证信息。|
|USERNAME|否|如果指定了此参数，则可以查看指定用户的认证信息。如果未指定此参数，则只能查看当前用户的认证信息。|

## 输出

```SQL
+---------------+----------+-------------+-------------------+
| UserIdentity  | Password | AuthPlugin  | UserForAuthPlugin |
+---------------+----------+-------------+-------------------+
```

|**字段**|**描述**|
|---|---|
|UserIdentity|用户标识。|
|Password|是否使用密码登录到StarRocks集群。<ul><li>`Yes`：使用密码。</li><li>`No`：不使用密码。</li></ul>|
|AuthPlugin|用于认证的插件。有效值：`MYSQL_NATIVE_PASSWORD`、`AUTHENTICATION_LDAP_SIMPLE` 或 `AUTHENTICATION_KERBEROS`。如果没有使用插件，则返回 `NULL`。|
|UserForAuthPlugin|使用LDAP或Kerberos认证的用户名称。如果未使用认证，则返回 `NULL`。|

## 示例

示例1：显示当前用户的认证信息。

```Plain
SHOW AUTHENTICATION;
+--------------+----------+------------+-------------------+
| UserIdentity | Password | AuthPlugin | UserForAuthPlugin |
+--------------+----------+------------+-------------------+
| 'root'@'%'   | No       | NULL       | NULL              |
+--------------+----------+------------+-------------------+
```

示例2：显示当前集群中所有用户的认证信息。

```Plain
SHOW ALL AUTHENTICATION;
+---------------+----------+-------------------------+-------------------+
| UserIdentity  | Password | AuthPlugin              | UserForAuthPlugin |
+---------------+----------+-------------------------+-------------------+
| 'root'@'%'    | Yes      | NULL                    | NULL              |
| 'chelsea'@'%' | No       | AUTHENTICATION_KERBEROS | HADOOP.COM        |
+---------------+----------+-------------------------+-------------------+
```

示例3：显示指定用户的认证信息。

```Plain
SHOW AUTHENTICATION FOR root;
+--------------+----------+------------+-------------------+
| UserIdentity | Password | AuthPlugin | UserForAuthPlugin |
+--------------+----------+------------+-------------------+
| 'root'@'%'   | Yes      | NULL       | NULL              |
+--------------+----------+------------+-------------------+
```