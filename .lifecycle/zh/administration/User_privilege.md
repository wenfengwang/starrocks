---
displayed_sidebar: English
---

# 管理用户权限

从'../assets/commonMarkdown/userPrivilegeCase.md'导入UserPrivilegeCase

本主题介绍如何在StarRocks中管理用户、角色和权限。

StarRocks采用角色基访问控制（RBAC）和基于身份的访问控制（IBAC）来管理集群内的权限，使得集群管理员可以轻松地在不同的粒度级别上限制集群内的权限。

在StarRocks集群内，可以将权限授予用户或角色。角色是一组可以根据需要分配给集群中的用户或其他角色的权限。用户可以被授予一个或多个角色，这些角色决定了他们对不同对象的权限。

## 查看用户和角色信息

拥有系统定义角色user_admin的用户可以查看StarRocks集群内的所有用户和角色信息。

### 查看权限信息

您可以使用[SHOW GRANTS](../sql-reference/sql-statements/account-management/SHOW_GRANTS.md)命令查看授予给用户或角色的权限。

- 查看当前用户的权限。

  ```SQL
  SHOW GRANTS;
  ```

    > **注意**
    > 任何用户都可以在**无需任何权限**的情况下查看自己的权限。

- 查看特定用户的权限。

  以下示例展示了用户jack的权限：

  ```SQL
  SHOW GRANTS FOR jack@'172.10.1.10';
  ```

- 查看特定角色的权限。

  以下示例展示了角色example_role的权限：

  ```SQL
  SHOW GRANTS FOR ROLE example_role;
  ```

### 查看用户属性

您可以使用[SHOW PROPERTY](../sql-reference/sql-statements/account-management/SHOW_PROPERTY.md)命令查看用户的属性。

以下示例展示了用户jack的属性：

```SQL
SHOW PROPERTY FOR jack@'172.10.1.10';
```

### 查看角色

您可以使用[SHOW ROLES](../sql-reference/sql-statements/account-management/SHOW_ROLES.md)命令查看StarRocks集群中的所有角色。

```SQL
SHOW ROLES;
```

### 查看用户

您可以使用SHOW USERS命令查看StarRocks集群中的所有用户。

```SQL
SHOW USERS;
```

## 管理用户

拥有系统定义角色user_admin的用户可以在StarRocks中创建、修改和删除用户。

### 创建用户

您可以通过指定用户身份、认证方法和默认角色来创建用户。

StarRocks支持使用登录凭证或LDAP认证进行用户认证。有关StarRocks认证的更多信息，请参阅[认证](../administration/Authentication.md)文档。有关创建用户的更多信息和高级指南，请参阅[CREATE USER](../sql-reference/sql-statements/account-management/CREATE_USER.md)文档。

以下示例创建了用户jack，只允许其从IP地址172.10.1.10连接，为其设置密码12345，并将角色example_role分配为其默认角色：

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED BY '12345' DEFAULT ROLE 'example_role';
```

> **注意**
- StarRocks会在存储用户密码之前加密它们。您可以使用password()函数获取加密密码。
- 如果在创建用户时未指定默认角色，系统会为用户分配默认角色PUBLIC。

### 修改用户

您可以修改用户的密码、默认角色或属性。

用户连接到StarRocks时，默认角色会自动激活。有关如何在连接后为用户启用所有（默认和已授予的）角色的说明，请参阅[启用所有角色](#enable-all-roles)。

#### 修改用户的默认角色

您可以使用[SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md)或[ALTER USER](../sql-reference/sql-statements/account-management/ALTER_USER.md)命令设置用户的默认角色。

以下两个示例都将用户jack的默认角色设置为db1_admin。注意，db1_admin必须已经分配给了jack。

- 使用SET DEFAULT ROLE设置默认角色：

  ```SQL
  SET DEFAULT ROLE 'db1_admin' TO jack@'172.10.1.10';
  ```

- 使用ALTER USER设置默认角色：

  ```SQL
  ALTER USER jack@'172.10.1.10' DEFAULT ROLE 'db1_admin';
  ```

#### 修改用户的属性

您可以使用[SET PROPERTY](../sql-reference/sql-statements/account-management/SET_PROPERTY.md)命令设置用户的属性。

以下示例将用户jack的最大连接数设置为1000。具有相同用户名的用户身份共享同一属性。

因此，您只需要为jack设置属性，该设置将对所有用户名为jack的用户身份生效。

```SQL
SET PROPERTY FOR jack 'max_user_connections' = '1000';
```

#### 为用户重置密码

您可以使用[SET PASSWORD](../sql-reference/sql-statements/account-management/SET_PASSWORD.md)或[ALTER USER](../sql-reference/sql-statements/account-management/ALTER_USER.md)命令为用户重置密码。

> **注意**
- 任何用户都可以在无需任何权限的情况下重置自己的密码。
- 只有`root`用户自己可以设置其密码。如果您丢失了密码并且无法连接到StarRocks，请参阅[重置丢失的root密码](#reset-lost-root-password)以获取更多指导。

以下两个示例都将用户jack的密码重置为54321：

- 使用SET PASSWORD重置密码：

  ```SQL
  SET PASSWORD FOR jack@'172.10.1.10' = PASSWORD('54321');
  ```

- 使用ALTER USER重置密码：

  ```SQL
  ALTER USER jack@'172.10.1.10' IDENTIFIED BY '54321';
  ```

#### 重置丢失的root密码

如果您丢失了root用户的密码并且无法连接到StarRocks，您可以按照以下步骤进行重置：

1. 在所有**FE节点**的配置文件**fe/conf/fe.conf**中添加以下配置项以禁用用户认证：

   ```YAML
   enable_auth_check = false
   ```

2. 重启**所有FE节点**，让配置生效。

   ```Bash
   ./fe/bin/stop_fe.sh
   ./fe/bin/start_fe.sh
   ```

3. 通过MySQL客户端以root用户身份连接到StarRocks。在用户认证被禁用时，您无需指定密码。

   ```Bash
   mysql -h <fe_ip_or_fqdn> -P<fe_query_port> -uroot
   ```

4. 为root用户重置密码。

   ```SQL
   SET PASSWORD for root = PASSWORD('xxxxxx');
   ```

5. 通过在**所有FE节点**的配置文件**fe/conf/fe.conf**中将配置项`enable_auth_check`设置为`true`来重新启用用户认证。

   ```YAML
   enable_auth_check = true
   ```

6. 重启**所有FE节点**，让配置生效。

   ```Bash
   ./fe/bin/stop_fe.sh
   ./fe/bin/start_fe.sh
   ```

7. 使用新密码和root用户身份通过MySQL客户端连接到StarRocks，以验证密码是否已成功重置。

   ```Bash
   mysql -h <fe_ip_or_fqdn> -P<fe_query_port> -uroot -p<xxxxxx>
   ```

### 删除用户

您可以使用[DROP USER](../sql-reference/sql-statements/account-management/DROP_USER.md)命令删除用户。

以下示例删除了用户jack：

```SQL
DROP USER jack@'172.10.1.10';
```

## 管理角色

拥有系统定义角色user_admin的用户可以在StarRocks中创建、授权、撤销或删除角色。

### 创建角色

您可以使用[CREATE ROLE](../sql-reference/sql-statements/account-management/CREATE_ROLE.md)命令创建角色。

以下示例创建了角色example_role：

```SQL
CREATE ROLE example_role;
```

### 授权角色

您可以使用[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)命令将角色授权给用户或另一个角色。

- 将角色授权给用户。

  以下示例将角色example_role授予了用户jack：

  ```SQL
  GRANT example_role TO USER jack@'172.10.1.10';
  ```

- 将角色授权给另一个角色。

  以下示例将角色example_role授予了角色test_role：

  ```SQL
  GRANT example_role TO ROLE test_role;
  ```

### 撤销角色

您可以使用[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md)命令从用户或另一个角色中撤销角色。

> **注意**
> 您不能从用户中撤销系统定义的默认角色 `PUBLIC`。

- 从用户中撤销角色。

  以下示例从用户jack中撤销了角色example_role：

  ```SQL
  REVOKE example_role FROM USER jack@'172.10.1.10';
  ```

- 从另一个角色中撤销角色。

  以下示例从角色test_role中撤销了角色example_role：

  ```SQL
  REVOKE example_role FROM ROLE test_role;
  ```

### 删除角色

您可以使用[DROP ROLE](../sql-reference/sql-statements/account-management/DROP_ROLE.md)命令删除角色。

以下示例删除了角色example_role：

```SQL
DROP ROLE example_role;
```

> **警告**
> 系统定义的角色不能被删除。

### 启用所有角色

用户的默认角色是每次连接到StarRocks集群时自动激活的角色。

如果您希望在所有StarRocks用户连接到集群时启用所有角色（默认角色和授予的角色），您可以执行以下操作。

此操作需要系统权限OPERATE。

```SQL
SET GLOBAL activate_all_roles_on_login = TRUE;
```

您也可以使用SET ROLE命令激活分配给您的角色。例如，用户jack@'172.10.1.10'拥有角色db_admin和user_admin，但这些不是用户的默认角色，不会在用户连接到StarRocks时自动激活。如果jack@'172.10.1.10'需要激活db_admin和user_admin，他可以执行SET ROLE db_admin, user_admin;。请注意，SET ROLE会覆盖原有角色。如果您想启用所有角色，请执行SET ROLE ALL。

## 管理权限

拥有系统定义角色user_admin的用户可以在StarRocks中授予或撤销权限。

### 授予权限

您可以使用[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)命令向用户或角色授予权限。

- 向用户授予权限。

  以下示例向用户jack授予了表sr_member的SELECT权限，并允许jack将该权限授予其他用户或角色（通过在SQL中指定WITH GRANT OPTION）：

  ```SQL
  GRANT SELECT ON TABLE sr_member TO USER jack@'172.10.1.10' WITH GRANT OPTION;
  ```

- 向角色授予权限。

  以下示例向角色example_role授予了表sr_member的SELECT权限：

  ```SQL
  GRANT SELECT ON TABLE sr_member TO ROLE example_role;
  ```

### 撤销权限

您可以使用[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md)命令撤销用户或角色的权限。

- 撤销用户的权限。

  以下示例撤销了用户jack对表sr_member的SELECT权限，并且禁止jack将该权限授予其他用户或角色：

  ```SQL
  REVOKE SELECT ON TABLE sr_member FROM USER jack@'172.10.1.10';
  ```

- 撤销角色的权限。

  以下示例从角色example_role撤销了对表sr_member的SELECT权限：

  ```SQL
  REVOKE SELECT ON TABLE sr_member FROM ROLE example_role;
  ```

## 最佳实践

### 多服务访问控制

通常，公司拥有的StarRocks集群由单一服务提供商管理，并维护多条业务线（LOB），每条业务线使用一个或多个数据库。

如下所示，StarRocks集群的用户包括服务提供商成员以及两个业务线LOB A和LOB B。每个LOB由分析师和高管两个角色运营。分析师负责生成和分析业务报表，而高管则负责查询这些报表。

![User Privileges](../assets/user_privilege_1.png)

LOB A独立管理数据库DB_A，LOB B独立管理数据库DB_B。LOB A和LOB B在DB_C中使用不同的表。DB_PUBLIC可被两个LOB的所有成员访问。

![User Privileges](../assets/user_privilege_2.png)

由于不同成员对不同数据库和表执行不同操作，我们建议您根据服务和职位创建角色，仅为每个角色分配必要的权限，并将这些角色分配给相应成员。如下所示：

![User Privileges](../assets/user_privilege_3.png)

1. 将系统定义角色db_admin、user_admin和cluster_admin分配给集群维护人员，将db_admin和user_admin设为他们的默认角色以便日常维护，并在需要操作集群节点时手动激活cluster_admin角色。

   例如：

   ```SQL
   GRANT db_admin, user_admin, cluster_admin TO USER user_platform;
   ALTER USER user_platform DEFAULT ROLE db_admin, user_admin;
   ```

2. 为LOB内的每位成员创建用户，并为每个用户设定复杂密码。
3. 为LOB内的每个职位创建角色，并赋予相应权限。
   为每个LOB的主管授予其LOB所需的最大权限集合，并赋予相应的GRANT权限（通过在声明中指定WITH GRANT OPTION），使他们能将这些权限分配给LOB成员。如果他们的日常工作需要，将该角色设为默认角色。
   例如：

   ```SQL
   GRANT SELECT, ALTER, INSERT, UPDATE, DELETE ON ALL TABLES IN DATABASE DB_A TO ROLE linea_admin WITH GRANT OPTION;
   GRANT SELECT, ALTER, INSERT, UPDATE, DELETE ON TABLE TABLE_C1, TABLE_C2, TABLE_C3 TO ROLE linea_admin WITH GRANT OPTION;
   GRANT linea_admin TO USER user_linea_admin;
   ALTER USER user_linea_admin DEFAULT ROLE linea_admin;
   ```
   为分析师和高管分配具有相应权限的角色。
   例如：

   ```SQL
   GRANT SELECT ON ALL TABLES IN DATABASE DB_A TO ROLE linea_query;
   GRANT SELECT ON TABLE TABLE_C1, TABLE_C2, TABLE_C3 TO ROLE linea_query;
   GRANT linea_query TO USER user_linea_salesa;
   GRANT linea_query TO USER user_linea_salesb;
   ALTER USER user_linea_salesa DEFAULT ROLE linea_query;
   ALTER USER user_linea_salesb DEFAULT ROLE linea_query;
   ```

4. 对于所有集群用户均可访问的数据库DB_PUBLIC，将DB_PUBLIC的SELECT权限授予系统定义的角色public。

   例如：

   ```SQL
   GRANT SELECT ON ALL TABLES IN DATABASE DB_PUBLIC TO ROLE public;
   ```

在复杂场景中，您可以通过角色分配实现角色继承。

例如，如果分析师需要对DB_PUBLIC中的表进行写入和查询操作，而高管仅能查询这些表，则您可以创建角色public_analysis和public_sales，为这些角色赋予相关权限，并将它们分配给分析师和高管的原始角色。

例如：

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

### 根据场景自定义角色。

<UserPrivilegeCase />

