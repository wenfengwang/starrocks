---
displayed_sidebar: "Chinese"
---

# Manage User Permissions

import UserPrivilegeCase from '../assets/commonMarkdown/userPrivilegeCase.md'

This article describes how to manage users, roles, and permissions in StarRocks.

StarRocks uses role-based access control (RBAC) and identity-based access control (IBAC) to manage permissions within the cluster, allowing cluster administrators to easily restrict permissions at different granularity levels.

In StarRocks clusters, you can grant permissions to users or roles. Roles are a set of permissions that can be granted to users or roles within the cluster as needed. A user or role can be granted one or more roles, which determine their permissions for different objects.

## View User and Role Information

Users with the system-built role `user_admin` can view user and role information in the StarRocks cluster.

### View Permission Information

You can use [SHOW GRANTS](../sql-reference/sql-statements/account-management/SHOW_GRANTS.md) to view the permissions granted to users or roles.

- View the permissions of the current user.

  ```SQL
  SHOW GRANTS;
  ```

  > **Note**
  >
  > Any user can view their own permissions without any permission.

- View the permissions of a specific user.

  The following example shows the permissions of the user `jack`.

  ```SQL
  SHOW GRANTS FOR jack@'172.10.1.10';
  ```

- View the permissions of a specific role.

  The following example shows the permissions of the role `example_role`.

  ```SQL
  SHOW GRANTS FOR ROLE example_role;
  ```

### View User Properties

You can use [SHOW PROPERTY](../sql-reference/sql-statements/account-management/SHOW_PROPERTY.md) to view user properties.

The following example shows the properties of the user `jack`:

```SQL
SHOW PROPERTY FOR jack@'172.10.1.10';
```

### View Roles

You can use [SHOW ROLES](../sql-reference/sql-statements/account-management/SHOW_ROLES.md) to view all roles in the StarRocks cluster.

```SQL
SHOW ROLES;
```

### View Users

You can use SHOW USERS to view all users in the StarRocks cluster.

```SQL
SHOW USERS;
```

## Manage Users

Users with the system-built role `user_admin` can create, modify, and delete users in StarRocks.

### Create User

You can create a user by specifying user identity, authentication method, and default role.

StarRocks supports user password login or LDAP authentication as the user authentication method. For more information about StarRocks authentication methods, see [User Authentication](../administration/Authentication.md). For more operational instructions on creating users, see [CREATE USER](../sql-reference/sql-statements/account-management/CREATE_USER.md).

The following example creates the user `jack`, only allowing connections from the IP address `172.10.1.10`, setting the password to `12345`, and assigning the role `example_role` to it as its default role:

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED BY '12345' DEFAULT ROLE 'example_role';
```

> **Note**
>
> - StarRocks encrypts the user password before storing it. You can use the password() function to obtain the encrypted password.
> - If no default role is specified during user creation, StarRocks assigns the system-built role `PUBLIC` as the user's default role.

### Modify User

You can modify a user's password, default role, or properties.

When a user connects to StarRocks, their default role is automatically activated. For instructions on how to enable all roles (default and granted) for the user after connection, see [Enable All Roles](#enable-all-roles).

#### Modify User Default Role

You can use [SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md) or [ALTER USER](../sql-reference/sql-statements/account-management/ALTER_USER.md) to set the user's default role.

Both of the following examples set `jack`'s default role to `db1_admin`. Before setting it, ensure that the `db1_admin` role has been granted to `jack`.

- Set the default role using SET DEFAULT ROLE:

  ```SQL
  SET DEFAULT ROLE 'db1_admin' TO jack@'172.10.1.10';
  ```

- Set the default role using ALTER USER:

  ```SQL
  ALTER USER jack@'172.10.1.10' DEFAULT ROLE 'db1_admin';
  ```

#### Modify User Properties

You can use [SET PROPERTY](../sql-reference/sql-statements/account-management/SET_PROPERTY.md) to set user properties.

User identities with the same username share properties. In the following example, configuring the property for `jack` will take effect for all user identities containing the username `jack`.

Set the maximum number of connections for the user `jack` to `1000`:

```SQL
SET PROPERTY FOR jack 'max_user_connections' = '1000';
```

#### Reset User Password

You can use [SET PASSWORD](../sql-reference/sql-statements/account-management/SET_PASSWORD.md) or [ALTER USER](../sql-reference/sql-statements/account-management/ALTER_USER.md) to reset a user's password.

> **Note**
>
> - Any user can reset their own password without any permission.
> - Only the `root` user can reset the password for the `root` user. If you lose your password and cannot connect to StarRocks, see [Resetting the Lost Root Password](#resetting-the-lost-root-password).

Both of the following examples reset `jack`'s password to `54321`:

- Reset the password using SET PASSWORD:

  ```SQL
  SET PASSWORD FOR jack@'172.10.1.10' = PASSWORD('54321');
  ```

- Reset the password using ALTER USER:

  ```SQL
  ALTER USER jack@'172.10.1.10' IDENTIFIED BY '54321';
  ```

#### Resetting the Lost Root Password

If you lose the password of the root user and cannot connect to StarRocks, you can reset the password by following these steps:

1. Add the following configuration in the configuration file **fe/conf/fe.conf** on **all FE nodes** to disable user authentication:

   ```YAML
   enable_auth_check = false
   ```

2. Restart **all FE nodes** to apply the configuration changes.

   ```Bash
   ./fe/bin/stop_fe.sh
   ./fe/bin/start_fe.sh
   ```

3. Connect to StarRocks from the MySQL client using the `root` user. When user authentication is disabled, you can log in without a password.

   ```Bash
   mysql -h <fe_ip_or_fqdn> -P<fe_query_port> -uroot
   ```

4. Reset the password for the `root` user.

   ```SQL
   SET PASSWORD for root = PASSWORD('xxxxxx');
   ```

5. Set the configuration item `enable_auth_check` to `true` in the configuration file **fe/conf/fe.conf** on **all FE nodes** to re-enable user authentication.

   ```YAML
   enable_auth_check = true
   ```

6. Restart **all FE nodes** to apply the configuration changes.

   ```Bash
   ./fe/bin/stop_fe.sh
   ./fe/bin/start_fe.sh
   ```

7. Connect to StarRocks using the `root` user and the new password from the MySQL client to verify that the password has been successfully reset.

   ```Bash
   mysql -h <fe_ip_or_fqdn> -P<fe_query_port> -uroot -p<xxxxxx>
   ```

### Delete User

You can use [DROP USER](../sql-reference/sql-statements/account-management/DROP_USER.md) to delete a user.

The following example deletes the user `jack`:

```SQL
DROP USER jack@'172.10.1.10';
```

## Manage Roles

拥有系统预置角色 `user_admin` 的用户可以在 StarRocks 中创建、授予、撤销和删除角色。

### 创建角色

您可以使用 [CREATE ROLE](../sql-reference/sql-statements/account-management/CREATE_ROLE.md) 创建角色。

以下示例创建角色 `example_role`：

```SQL
CREATE ROLE example_role;
```

### 授予角色

您可以使用 [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) 将角色授予用户或其他角色。

- 将角色授予用户。

  以下示例将角色 `example_role` 授予用户 `jack`：

  ```SQL
  GRANT example_role TO USER jack@'172.10.1.10';
  ```

- 将角色授予其他角色。

  以下示例将角色 `example_role` 授予角色 `test_role`：

  ```SQL
  GRANT example_role TO ROLE test_role;
  ```

### 撤销角色

您可以使用 [REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md) 将角色从用户或其他角色撤销。

> **说明**
>
> 系统预置的默认角色 `PUBLIC` 无法撤销。

- 从用户撤销角色。

  以下示例从用户 `jack` 撤销角色 `example_role`：

  ```SQL
  REVOKE example_role FROM USER jack@'172.10.1.10';
  ```

- 从角色撤销其他角色。

  以下示例从角色 `test_role` 撤销角色 `example_role`：

  ```SQL
  REVOKE example_role FROM ROLE test_role;
  ```

### 删除角色

您可以使用 [DROP ROLE](../sql-reference/sql-statements/account-management/DROP_ROLE.md) 删除角色。

以下示例删除角色 `example_role`：

```SQL
DROP ROLE example_role;
```

> **注意**
>
> 系统预置角色无法删除。

### 启用所有角色

用户的默认角色是每次用户连接到 StarRocks 集群时自动激活的角色。授予给角色的权限仅在授予后生效。

如果您希望集群里所有的用户在登录时都默认激活所有角色（默认和授予的角色），可以执行如下操作。该操作需要 system 层的 OPERATE 权限。

执行以下语句为集群中用户启用所有角色：

```SQL
SET GLOBAL activate_all_roles_on_login = TRUE;
```

您还可以通过 SET ROLE 来手动激活拥有的角色。例如用户 jack@'172.10.1.10' 拥有 `db_admin` 和 `user_admin` 角色，但此角色不是他的默认角色，因此在登录时不会被默认激活。当 jack@'172.10.1.10' 需要激活 `db_admin`和 `user_admin` 时，可以手动执行 `SET ROLE db_admin, user_admin;`。 注意 SET ROLE 命令是覆盖的，如果您希望激活拥有的所有角色，可以执行 SET ROLE ALL。

## 管理权限

拥有系统预置角色 `user_admin` 的用户可以在 StarRocks 中授予和撤销权限。

### 授予权限

您可以使用 [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) 向用户或角色授予权限。

- 向用户授予权限。

  以下示例将表 `sr_member` 的 SELECT 权限授予用户 `jack`，并允许 `jack` 将此权限授予其他用户或角色（通过在 SQL 中指定 WITH GRANT OPTION）：

  ```SQL
  GRANT SELECT ON TABLE sr_member TO USER jack@'172.10.1.10' WITH GRANT OPTION;
  ```

- 向角色授予权限。

  以下示例将表 `sr_member` 的 SELECT 权限授予角色 `example_role`：

  ```SQL
  GRANT SELECT ON TABLE sr_member TO ROLE example_role;
  ```

### 撤销权限

您可以使用 [REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md) 撤销用户或角色的权限。

- 撤销用户的权限。

  以下示例撤销用户 `jack` 对表 `sr_member` 的 SELECT 权限，并禁止 `jack` 将此权限授予其他用户或角色：

  ```SQL
  REVOKE SELECT ON TABLE sr_member FROM USER jack@'172.10.1.10';
  ```

- 撤销角色的权限。

  以下示例撤销角色 `example_role` 对表 `sr_member` 的 SELECT 权限：

  ```SQL
  REVOKE SELECT ON TABLE sr_member FROM ROLE example_role;
  ```

## 最佳实践

### 多业务线权限管理

通常，在企业内部，StarRocks 集群会由平台方统一运维管理，向各类业务方提供服务。其中，一个 StarRocks 集群内可能包含多个业务线，每个业务线可能涉及到一个或多个数据库。

举例来说，在人员架构上包含平台方和业务方。业务方涉及业务线 A 和业务线 B，业务线内包含不同角色的岗位，例如分析师和业务员。分析师日常需要产出报表、分析报告，业务员日常需要查询分析师产出的报表。

![User Privileges](../assets/user_privilege_1.png)

在数据结构上，业务 A 和业务 B 均有自己的数据库 `DB_A` 和 `DB_B`。在数据库 `DB_C` 中，业务 A 和业务 B 均需要用到部分表。并且公司中所有人都可以访问公共数据库 `DB_PUBLIC`。

![User Privileges](../assets/user_privilege_2.png)

由于不同业务、不同岗位的日常操作与涉及库表不同，StarRocks 建议您按照业务、岗位来创建角色，将所需权限赋予给对应角色后再分配给用户。具体来说：

![User Privileges](../assets/user_privilege_3.png)

1. 将系统预置角色 `db_admin`、`user_admin` 以及 `cluster_admin` 赋予给平台运维角色。同时将 `db_admin` 和 `user_admin` 作为默认角色，用于日常的基础运维。当确认需要进行节点操作时，再手动激活 `cluster_admin` 角色。

   例如：

   ```SQL
   GRANT db_admin, user_admin, cluster_admin TO USER user_platform;
   ALTER USER user_platform DEFAULT ROLE db_admin, user_admin;
   ```

2. 由平台运维人员创建系统内的所有用户，每人对应一个用户，并设置复杂密码。
3. 为每个业务方按照职能设置角色，例如本例中的业务管理员、业务分析师、业务员。并为他们赋予对应权限。

   对于业务负责人，可以赋予该业务所需权限的最大集合，并赋予他们赋权权限（即，在授权时加上 WITH GRANT OPTION 关键字）。从而，他们可以在后续工作中自行为下属分配所需权限。如果此角色为他们的日常角色，则可以设置为Default Role。

   例如：

   ```SQL
   GRANT SELECT, ALTER, INSERT, UPDATE, DELETE ON ALL TABLES IN DATABASE DB_A TO ROLE linea_admin WITH GRANT OPTION;
   GRANT SELECT, ALTER, INSERT, UPDATE, DELETE ON TABLE TABLE_C1, TABLE_C2, TABLE_C3 TO ROLE linea_admin WITH GRANT OPTION;
   GRANT linea_admin TO USER user_linea_admin;
   ALTER USER user_linea_admin DEFAULT ROLE linea_admin;
   ```

对于分析师、业务员等角色，赋予他们对应操作权限即可。

例如：

```SQL
GRANT SELECT ON ALL TABLES IN DATABASE DB_A TO ROLE linea_query;
GRANT SELECT ON TABLE TABLE_C1, TABLE_C2, TABLE_C3 TO ROLE linea_query;
GRANT linea_query TO USER user_linea_salesa;
GRANT linea_query TO USER user_linea_salesb;
ALTER USER user_linea_salesa DEFAULT ROLE linea_query;
ALTER USER user_linea_salesb DEFAULT ROLE linea_query;
```

4. 对于任何人都可以访问的公共库，可将该库下所有表的查询权限赋予给预置角色 `public`。

例如：

```SQL
GRANT SELECT ON ALL TABLES IN DATABASE DB_PUBLIC TO ROLE public;
```

在其他复杂情况下，您也可以通过将角色赋予给其他的角色来达到权限继承的目的。

例如，所有分析师可以对 DB_PUBLIC 的数据进行导入与修改，所有业务员可以对 DB_PUBLIC 的数据进行查询，您可以创建 public_analysis 和 public_sales，授予对应权限后，再将角色赋予给所有业务线的分析师和业务员角色。

```SQL
CREATE ROLE public_analysis;
CREATE ROLE public_sales;
GRANT SELECT, ALTER, INSERT, UPDATE, DELETE ON ALL TABLES IN DATABASE DB_PUBLIC TO ROLE public_analysis;
GRANT SELECT ON ALL TABLES IN DATABASE DB_PUBLIC TO ROLE public_sales;
GRANT public_analysis TO ROLE linea_analysis;
GRANT public_analysis TO ROLE lineb_analysis;
GRANT public_sales TO ROLE linea_query;
GRANT public_sales TO ROLE lineb_query;
```

### 基于使用场景创建自定义角色

<UserPrivilegeCase />