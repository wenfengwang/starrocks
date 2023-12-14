---
displayed_sidebar: "Chinese"
---

# 管理用户权限

从 UserPrivilegeCase 中导入 UserPrivilegeCase 的文档

本主题描述了如何在 StarRocks 中管理用户、角色和权限。

StarRocks 使用基于角色的访问控制（RBAC）和基于标识的访问控制（IBAC）来管理 StarRocks 集群内的权限，允许集群管理员在不同的粒度级别上轻松限制集群内的权限。

在 StarRocks 集群中，权限可以授予用户或角色。角色是一组权限，可以根据需要分配给集群中的用户或其他角色。用户可以被授予一个或多个角色，这些角色决定了他们在不同对象上的权限。

## 查看用户和角色信息

具有系统定义角色 `user_admin` 的用户可以查看 StarRocks 集群内的所有用户和角色信息。

### 查看权限信息

您可以使用 [SHOW GRANTS](../sql-reference/sql-statements/account-management/SHOW_GRANTS.md) 查看授予用户或角色的权限。

- 查看当前用户的权限。

  ```SQL
  SHOW GRANTS;
  ```

  > **注意**
  >
  > 任何用户可以查看自己的权限而无需任何权限。

- 查看特定用户的权限。

  以下示例显示了用户 `jack` 的权限：

  ```SQL
  SHOW GRANTS FOR jack@'172.10.1.10';
  ```

- 查看特定角色的权限。

  以下示例显示了角色 `example_role` 的权限：

  ```SQL
  SHOW GRANTS FOR ROLE example_role;
  ```

### 查看用户属性

您可以使用 [SHOW PROPERTY](../sql-reference/sql-statements/account-management/SHOW_PROPERTY.md) 查看用户的属性。

以下示例显示了用户 `jack` 的属性：

```SQL
SHOW PROPERTY FOR jack@'172.10.1.10';
```

### 查看角色

您可以使用 [SHOW ROLES](../sql-reference/sql-statements/account-management/SHOW_ROLES.md) 查看 StarRocks 集群内的所有角色。

```SQL
SHOW ROLES;
```

### 查看用户

您可以使用 SHOW USERS 查看 StarRocks 集群内的所有用户。

```SQL
SHOW USERS;
```

## 管理用户

具有系统定义角色 `user_admin` 的用户可以在 StarRocks 中创建用户、修改用户和移除用户。

### 创建用户

您可以通过指定用户标识、身份验证方法和默认角色来创建用户。

StarRocks 支持使用登录凭据或 LDAP 认证进行用户认证。有关 StarRocks 的身份验证的更多信息，请参阅 [Authentication](../administration/Authentication.md)。有关创建用户的更多信息和高级说明，请参阅 [CREATE USER](../sql-reference/sql-statements/account-management/CREATE_USER.md)。

以下示例创建了用户 `jack`，允许它只能从 IP 地址 `172.10.1.10` 进行连接，为其设置密码为 `12345`，并将角色 `example_role` 分配为其默认角色：

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED BY '12345' DEFAULT ROLE 'example_role';
```

> **注意**
>
> - StarRocks 在存储用户密码之前会对其进行加密。您可以使用 password() 函数获取加密后的密码。
> - 如果在用户创建期间未指定默认角色，则将系统定义的默认角色 `PUBLIC` 分配给用户。

### 修改用户

您可以修改用户的密码、默认角色或属性。

用户连接到 StarRocks 时，用户的默认角色会自动激活。有关如何在连接后为用户启用所有 (默认和授予) 角色的说明，请参阅 [启用所有角色](#enable-all-roles)。

#### 修改用户的默认角色

您可以使用 [SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md) 或 [ALTER USER](../sql-reference/sql-statements/account-management/ALTER_USER.md) 设置用户的默认角色。

以下两个示例都将 `jack` 的默认角色设置为 `db1_admin`。请注意，`db1_admin` 必须已分配给 `jack`。

- 使用 SET DEFAULT ROLE 设置默认角色：

  ```SQL
  SET DEFAULT ROLE 'db1_admin' TO jack@'172.10.1.10';
  ```

- 使用 ALTER USER 设置默认角色：

  ```SQL
  ALTER USER jack@'172.10.1.10' DEFAULT ROLE 'db1_admin';
  ```

#### 修改用户的属性

您可以使用 [SET PROPERTY](../sql-reference/sql-statements/account-management/SET_PROPERTY.md) 设置用户的属性。

以下示例将用户 `jack` 的最大连接数设置为 `1000`。具有相同用户名的用户标识共享相同的属性。

因此，您只需要为 `jack` 设置属性，这个设置将对所有用户标识的用户名为 `jack` 的用户生效。

```SQL
SET PROPERTY FOR jack 'max_user_connections' = '1000';
```

#### 重置用户的密码

您可以使用 [SET PASSWORD](../sql-reference/sql-statements/account-management/SET_PASSWORD.md) 或 [ALTER USER](../sql-reference/sql-statements/account-management/ALTER_USER.md) 重新设置用户的密码。

> **注意**
>
> - 任何用户都可以在无需任何权限的情况下重置自己的密码。
> - 只有 `root` 用户本身可以设置其密码。如果您忘记了其密码并且无法连接到 StarRocks，请参阅 [重置丢失的 root 密码](#reset-lost-root-password) 获取更多说明。

以下两个示例将 `jack` 的密码重置为 `54321`：

- 使用 SET PASSWORD 重置密码：

  ```SQL
  SET PASSWORD FOR jack@'172.10.1.10' = PASSWORD('54321');
  ```

- 使用 ALTER USER 重置密码：

  ```SQL
  ALTER USER jack@'172.10.1.10' IDENTIFIED BY '54321';
  ```

#### 重置丢失的 root 密码

如果您忘记了 `root` 用户的密码并且无法连接到 StarRocks，您可以按照以下步骤重置它：

1. 将以下配置项添加到**所有 FE 节点**的配置文件 **fe/conf/fe.conf** 中，以禁用用户身份验证：

   ```YAML
   enable_auth_check = false
   ```

2. 重新启动**所有 FE 节点**以使配置生效。

   ```Bash
   ./fe/bin/stop_fe.sh
   ./fe/bin/start_fe.sh
   ```

3. 通过 MySQL 客户端以 `root` 用户连接到 StarRocks。当用户身份验证被禁用时，您不需要在连接时指定密码。

   ```Bash
   mysql -h <fe_ip_or_fqdn> -P<fe_query_port> -uroot
   ```

4. 为 `root` 用户重置密码。

   ```SQL
   SET PASSWORD for root = PASSWORD('xxxxxx');
   ```

5. 通过在配置文件**所有 FE 节点**的配置文件**fe/conf/fe.conf**中设置配置项 `enable_auth_check` 为 `true` 来重新启用用户身份验证。

   ```YAML
   enable_auth_check = true
   ```

6. 重新启动**所有 FE 节点**以使配置生效。

   ```Bash
   ./fe/bin/stop_fe.sh
   ./fe/bin/start_fe.sh
   ```

7. 使用新密码通过 `root` 用户从 MySQL 客户端连接到 StarRocks，以验证密码是否成功重置。

   ```Bash
   mysql -h <fe_ip_or_fqdn> -P<fe_query_port> -uroot -p<xxxxxx>
   ```

### 移除用户

您可以使用 [DROP USER](../sql-reference/sql-statements/account-management/DROP_USER.md) 移除用户。

以下示例移除用户 `jack`：

```SQL
DROP USER jack@'172.10.1.10';
```

## 管理角色

具有系统定义角色 `user_admin` 的用户可以在 StarRocks 中创建、授予、撤销或移除角色。

### 创建角色

您可以使用 [CREATE ROLE](../sql-reference/sql-statements/account-management/CREATE_ROLE.md) 创建角色。

以下示例创建了角色 `example_role`：

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

您可以使用 [REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md) 从用户或其他角色撤销角色。

> **注意**
>
> 不能从用户中撤消系统定义的默认角色`PUBLIC`。

- 从用户中撤消角色。

  以下示例从用户`jack`中撤消角色`example_role`：

  ```SQL
  REVOKE example_role FROM USER jack@'172.10.1.10';
  ```

- 从另一个角色中撤消角色。

  以下示例从角色`test_role`中撤消角色`example_role`：

  ```SQL
  REVOKE example_role FROM ROLE test_role;
  ```

### 撤销角色

您可以使用[DROP ROLE](../sql-reference/sql-statements/account-management/DROP_ROLE.md)来撤销角色。

以下示例撤销了角色`example_role`：

```SQL
DROP ROLE example_role;
```

> **注意**
>
> 无法撤销系统定义的角色。

### 启用所有角色

用户的默认角色是每次用户连接到StarRocks集群时自动激活的角色。

如果要在StarRocks用户连接到StarRocks集群时为所有StarRocks用户启用所有角色（默认和授予的角色），可以执行以下操作。

此操作需要系统权限OPERATE。

```SQL
SET GLOBAL activate_all_roles_on_login = TRUE;
```

您还可以使用SET ROLE来激活分配给您的角色。例如，用户`jack@'172.10.1.10'`具有`db_admin`和`user_admin`角色，但它们不是用户的默认角色，并且当用户连接到StarRocks时不会自动激活这些角色。如果jack@'172.10.1.10'需要激活`db_admin`和`user_admin`，可以运行`SET ROLE db_admin, user_admin;`。请注意，SET ROLE会覆盖原始角色。如果要启用所有您的角色，请运行SET ROLE ALL。

## 管理权限

拥有系统定义角色`user_admin`的用户可以在StarRocks中授予或撤销权限。

### 授予权限

您可以使用[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)向用户或角色授予权限。

- 向用户授予权限。

  以下示例向用户`jack`授予对表`sr_member`的SELECT权限，并允许`jack`将此权限授予其他用户或角色（在SQL中指定WITH GRANT OPTION）：

  ```SQL
  GRANT SELECT ON TABLE sr_member TO USER jack@'172.10.1.10' WITH GRANT OPTION;
  ```

- 向角色授予权限。

  以下示例向角色`example_role`授予对表`sr_member`的SELECT权限：

  ```SQL
  GRANT SELECT ON TABLE sr_member TO ROLE example_role;
  ```

### 撤销权限

您可以使用[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md)从用户或角色中撤销权限。

- 从用户中撤消权限。

  以下示例从用户`jack`中撤消对表`sr_member`的SELECT权限，并禁止`jack`将此权限授予其他用户或角色：

  ```SQL
  REVOKE SELECT ON TABLE sr_member FROM USER jack@'172.10.1.10';
  ```

- 从角色中撤消权限。

  以下示例从角色`example_role`中撤消对表`sr_member`的SELECT权限：

  ```SQL
  REVOKE SELECT ON TABLE sr_member FROM ROLE example_role;
  ```

## 最佳实践

### 多服务访问控制

通常，由唯一服务提供商管理的公司拥有的StarRocks集群维护多条业务线（LOB），每条业务线使用一个或多个数据库。

如下所示，StarRocks集群的用户包括来自服务提供商和两条业务线（A和B）的成员。每条业务线由两个角色操作 - 分析员和执行员。 分析员生成和分析业务报表，执行员查询报表。

![](../assets/user_privilege_1.png)

LOB A独立管理数据库`DB_A`，LOB B管理数据库`DB_B`。LOB A和LOB B在`DB_C`中使用不同的表。`DB_PUBLIC`可以被来自两个LOB的所有成员访问。

![](../assets/user_privilege_2.png)

由于不同成员在不同数据库和表上执行不同操作，我们建议您根据其服务和职位创建角色，并仅向每个角色分配必要的权限，并将这些角色分配给相应的成员。 如下所示：

1. 将系统定义角色`db_admin`，`user_admin`和`cluster_admin`分配给集群维护者，将`db_admin`和`user_admin`设置为他们的默认角色以进行日常维护，并在他们需要操作集群节点时手动激活角色`cluster_admin`。

   示例：

   ```SQL

   GRANT db_admin, user_admin, cluster_admin TO USER user_platform;
   ALTER USER user_platform DEFAULT ROLE db_admin, user_admin;
   ```


2. 为LOB内的每个成员创建用户，并为每个用户设置复杂密码。
3. 为LOB内的每个职位创建角色，并向每个角色应用相应权限。

   对于每个LOB的主管，向他们的角色授予他们的LOB需要的最大权限集，以及相应的授权权限（在语句中指定WITH GRANT OPTION）。因此，他们可以将这些权限分配给LOB的成员。如有需要，将角色设置为他们的默认角色。

   示例：

   ```SQL
   GRANT SELECT, ALTER, INSERT, UPDATE, DELETE ON ALL TABLES IN DATABASE DB_A TO ROLE linea_admin WITH GRANT OPTION;
   GRANT SELECT, ALTER, INSERT, UPDATE, DELETE ON TABLE TABLE_C1, TABLE_C2, TABLE_C3 TO ROLE linea_admin WITH GRANT OPTION;
   GRANT linea_admin TO USER user_linea_admin;
   ALTER USER user_linea_admin DEFAULT ROLE linea_admin;
   ```

   对于分析员和执行员，将分配具有相应权限的角色给他们。

   示例：

   ```SQL
   GRANT SELECT ON ALL TABLES IN DATABASE DB_A TO ROLE linea_query;
   GRANT SELECT ON TABLE TABLE_C1, TABLE_C2, TABLE_C3 TO ROLE linea_query;
   GRANT linea_query TO USER user_linea_salesa;
   GRANT linea_query TO USER user_linea_salesb;
   ALTER USER user_linea_salesa DEFAULT ROLE linea_query;
   ALTER USER user_linea_salesb DEFAULT ROLE linea_query;
   ```

4. 对于`DB_PUBLIC`数据库，可以被所有集群用户访问，将`DB_PUBLIC`的SELECT权限授予系统定义的角色`public`。

   示例：

   ```SQL
   GRANT SELECT ON ALL TABLES IN DATABASE DB_PUBLIC TO ROLE public;
   ```

您可以将角色分配给其他成员，以实现在复杂场景中的角色继承。

例如，如果分析员需要对`DB_PUBLIC`中的表进行写入和查询操作，而执行员只能查询这些表，您可以创建`public_analysis`和`public_sales`角色，并向原始角色分析员和执行员分配相关权限。

示例：

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

### 根据场景自定义角色

<UserPrivilegeCase />