---
displayed_sidebar: English
---

```markdown
# 管理用户权限

import UserPrivilegeCase from '../assets/commonMarkdown/userPrivilegeCase.md'

本主题描述了如何在 StarRocks 中管理用户、角色和权限。

StarRocks 采用基于角色的访问控制（RBAC）和基于身份的访问控制（IBAC）来管理 StarRocks 集群内的权限，使集群管理员能够轻松限制集群内不同粒度级别的权限。

在 StarRocks 集群内，可以向用户或角色授予权限。角色是一组可以根据需要分配给集群中的用户或其他角色的权限。用户可以被授予一个或多个角色，这决定了他们对不同对象的权限。

## 查看用户和角色信息

具有系统定义角色 `user_admin` 的用户可以查看 StarRocks 集群内的所有用户和角色信息。

### 查看权限信息

您可以使用 [SHOW GRANTS](../sql-reference/sql-statements/account-management/SHOW_GRANTS.md) 查看授予给用户或角色的权限。

- 查看当前用户的权限。

  ```SQL
  SHOW GRANTS;
  ```

    > **注意**
    > 任何用户都可以查看自己的权限，无需任何特殊权限。

- 查看特定用户的权限。

  下面的示例显示了用户 `jack` 的权限：

  ```SQL
  SHOW GRANTS FOR jack@'172.10.1.10';
  ```

- 查看特定角色的权限。

  下面的示例显示了角色 `example_role` 的权限：

  ```SQL
  SHOW GRANTS FOR ROLE example_role;
  ```

### 查看用户属性

您可以使用 [SHOW PROPERTY](../sql-reference/sql-statements/account-management/SHOW_PROPERTY.md) 查看用户的属性。

下面的示例显示了用户 `jack` 的属性：

```SQL
SHOW PROPERTY FOR jack@'172.10.1.10';
```

### 查看角色

您可以使用 [SHOW ROLES](../sql-reference/sql-statements/account-management/SHOW_ROLES.md) 查看 StarRocks 集群中的所有角色。

```SQL
SHOW ROLES;
```

### 查看用户

您可以使用 SHOW USERS 查看 StarRocks 集群中的所有用户。

```SQL
SHOW USERS;
```

## 管理用户

具有系统定义角色 `user_admin` 的用户可以在 StarRocks 中创建、修改和删除用户。

### 创建用户

您可以通过指定用户标识、认证方式和默认角色来创建用户。

StarRocks 支持使用登录凭证或 LDAP 认证进行用户认证。有关 StarRocks 认证的更多信息，请参阅[认证](../administration/Authentication.md)。有关创建用户的更多信息和高级指南，请参阅 [CREATE USER](../sql-reference/sql-statements/account-management/CREATE_USER.md)。

下面的示例创建了用户 `jack`，允许其仅从 IP 地址 `172.10.1.10` 连接，为其设置密码 `12345`，并将角色 `example_role` 分配给它作为其默认角色：

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED BY '12345' DEFAULT ROLE 'example_role';
```

> **注意**
- StarRocks 在存储用户密码之前会对其进行加密。您可以使用 password() 函数获取加密后的密码。
- 如果在创建用户时未指定默认角色，则会为用户分配系统定义的默认角色 `PUBLIC`。

### 修改用户

您可以修改用户的密码、默认角色或属性。

用户的默认角色在其连接到 StarRocks 时会自动激活。有关如何在连接后为用户启用所有（默认和授予的）角色的指南，请参阅[启用所有角色](#enable-all-roles)。

#### 修改用户的默认角色

您可以使用 [SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md) 或 [ALTER USER](../sql-reference/sql-statements/account-management/ALTER_USER.md) 来设置用户的默认角色。

以下两个示例都将 `jack` 的默认角色设置为 `db1_admin`。请注意，`db1_admin` 必须已经分配给 `jack`。

- 使用 SET DEFAULT ROLE 设置默认角色：

  ```SQL
  SET DEFAULT ROLE 'db1_admin' TO jack@'172.10.1.10';
  ```

- 使用 ALTER USER 设置默认角色：

  ```SQL
  ALTER USER jack@'172.10.1.10' DEFAULT ROLE 'db1_admin';
  ```

#### 修改用户的属性

您可以使用 [SET PROPERTY](../sql-reference/sql-statements/account-management/SET_PROPERTY.md) 来设置用户的属性。

下面的示例将用户 `jack` 的最大连接数设置为 `1000`。具有相同用户名的用户身份共享相同的属性。

因此，您只需为 `jack` 设置属性，该设置将对所有用户名为 `jack` 的用户身份生效。

```SQL
SET PROPERTY FOR jack 'max_user_connections' = '1000';
```

#### 为用户重置密码

您可以使用 [SET PASSWORD](../sql-reference/sql-statements/account-management/SET_PASSWORD.md) 或 [ALTER USER](../sql-reference/sql-statements/account-management/ALTER_USER.md) 来重置用户的密码。

> **注意**
- 任何用户都可以重置自己的密码，无需任何特殊权限。
- 只有 `root` 用户本身可以设置其密码。如果您丢失了密码并且无法连接到 StarRocks，请参阅[重置丢失的 root 密码](#reset-lost-root-password)以获取更多指南。

以下两个示例都将 `jack` 的密码重置为 `54321`：

- 使用 SET PASSWORD 重置密码：

  ```SQL
  SET PASSWORD FOR jack@'172.10.1.10' = PASSWORD('54321');
  ```

- 使用 ALTER USER 重置密码：

  ```SQL
  ALTER USER jack@'172.10.1.10' IDENTIFIED BY '54321';
  ```

#### 重置丢失的 root 密码

如果您丢失了 `root` 用户的密码并且无法连接到 StarRocks，您可以按照以下步骤来重置它：

1. 在所有 FE 节点的配置文件 **fe/conf/fe.conf** 中添加以下配置项，以禁用用户认证：

   ```YAML
   enable_auth_check = false
   ```

2. 重启所有 FE 节点以使配置生效。

   ```Bash
   ./fe/bin/stop_fe.sh
   ./fe/bin/start_fe.sh
   ```

3. 使用 `root` 用户从 MySQL 客户端连接到 StarRocks。当用户认证被禁用时，无需指定密码。

   ```Bash
   mysql -h <fe_ip_or_fqdn> -P<fe_query_port> -uroot
   ```

4. 重置 `root` 用户的密码。

   ```SQL
   SET PASSWORD for root = PASSWORD('xxxxxx');
   ```

5. 通过在所有 FE 节点的配置文件 **fe/conf/fe.conf** 中将配置项 `enable_auth_check` 设置为 `true`，重新启用用户认证。

   ```YAML
   enable_auth_check = true
   ```

6. 重启所有 FE 节点以使配置生效。

   ```Bash
   ./fe/bin/stop_fe.sh
   ./fe/bin/start_fe.sh
   ```

7. 使用 `root` 用户和新密码从 MySQL 客户端连接到 StarRocks，以验证密码是否成功重置。

   ```Bash
   mysql -h <fe_ip_or_fqdn> -P<fe_query_port> -uroot -p<xxxxxx>
   ```

### 删除用户

您可以使用 [DROP USER](../sql-reference/sql-statements/account-management/DROP_USER.md) 来删除用户。

下面的示例删除了用户 `jack`：

```SQL
DROP USER jack@'172.10.1.10';
```

## 管理角色

具有系统定义角色 `user_admin` 的用户可以在 StarRocks 中创建、授权、撤销或删除角色。

### 创建角色

您可以使用 [CREATE ROLE](../sql-reference/sql-statements/account-management/CREATE_ROLE.md) 来创建角色。

下面的示例创建了角色 `example_role`：

```SQL
CREATE ROLE example_role;
```
```
```SQL
CREATE ROLE example_role;
```

### 授予角色

您可以使用 [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) 将角色授予用户或其他角色。

- 向用户授予角色。

  下面的示例将 `example_role` 角色授予用户 `jack`：

  ```SQL
  GRANT example_role TO USER jack@'172.10.1.10';
  ```

- 将角色授予另一个角色。

  下面的示例将 `example_role` 角色授予 `test_role` 角色：

  ```SQL
  GRANT example_role TO ROLE test_role;
  ```

### 撤销角色

您可以使用 [REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md) 撤销用户或其他角色的角色。

> **注意**
> 您无法撤销用户的系统定义默认角色 `PUBLIC`。

- 撤销用户的角色。

  下面的示例从用户 `jack` 撤销 `example_role` 角色：

  ```SQL
  REVOKE example_role FROM USER jack@'172.10.1.10';
  ```

- 从另一个角色撤销角色。

  下面的示例从 `test_role` 角色撤销 `example_role` 角色：

  ```SQL
  REVOKE example_role FROM ROLE test_role;
  ```

### 删除角色

您可以使用 [DROP ROLE](../sql-reference/sql-statements/account-management/DROP_ROLE.md) 删除角色。

下面的示例删除 `example_role` 角色：

```SQL
DROP ROLE example_role;
```

> **警告**
> 系统定义的角色不能被删除。

### 启用所有角色

用户的默认角色是每次连接到 StarRocks 集群时自动激活的角色。

如果您希望在所有 StarRocks 用户连接到 StarRocks 集群时启用所有角色（默认角色和授予的角色），可以执行以下操作。

此操作需要系统权限 `OPERATE`。

```SQL
SET GLOBAL activate_all_roles_on_login = TRUE;
```

您还可以使用 `SET ROLE` 激活分配给您的角色。例如，用户 `jack@'172.10.1.10'` 拥有 `db_admin` 和 `user_admin` 角色，但它们不是用户的默认角色，并且在用户连接到 StarRocks 时不会自动激活。如果 `jack@'172.10.1.10'` 需要激活 `db_admin` 和 `user_admin`，他可以运行 `SET ROLE db_admin, user_admin;`。请注意，`SET ROLE` 会覆盖原有角色。如果您想启用所有角色，请运行 `SET ROLE ALL`。

## 管理权限

具有系统定义角色 `user_admin` 的用户可以在 StarRocks 中授予或撤销权限。

### 授予权限

您可以使用 [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) 向用户或角色授予权限。

- 授予用户权限。

  下面的示例将 `sr_member` 表的 `SELECT` 权限授予用户 `jack`，并允许 `jack` 将该权限授予其他用户或角色（通过在 SQL 中指定 `WITH GRANT OPTION`）：

  ```SQL
  GRANT SELECT ON TABLE sr_member TO USER jack@'172.10.1.10' WITH GRANT OPTION;
  ```

- 授予角色权限。

  下面的示例将 `sr_member` 表的 `SELECT` 权限授予 `example_role` 角色：

  ```SQL
  GRANT SELECT ON TABLE sr_member TO ROLE example_role;
  ```

### 撤销权限

您可以使用 [REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md) 撤销用户或角色的权限。

- 撤销用户的权限。

  下面的示例撤销用户 `jack` 对 `sr_member` 表的 `SELECT` 权限，并不允许 `jack` 将此权限授予其他用户或角色：

  ```SQL
  REVOKE SELECT ON TABLE sr_member FROM USER jack@'172.10.1.10';
  ```

- 撤销角色的权限。

  下面的示例从 `example_role` 角色撤销对 `sr_member` 表的 `SELECT` 权限：

  ```SQL
  REVOKE SELECT ON TABLE sr_member FROM ROLE example_role;
  ```

## 最佳实践

### 多业务访问控制

通常，公司拥有的 StarRocks 集群由唯一的服务提供商管理，并维护多个业务线（LOB），每个业务线都使用一个或多个数据库。

如下图所示，StarRocks 集群的用户包括来自服务提供商的成员和两个 LOB（A 和 B）。每个 LOB 由两个角色运营 - 分析师和高管。分析师生成并分析业务报表，高管则查询报表。

![User Privileges](../assets/user_privilege_1.png)

LOB A 独立管理数据库 `DB_A`，LOB B 独立管理数据库 `DB_B`。LOB A 和 LOB B 使用 `DB_C` 中的不同表。`DB_PUBLIC` 可由两个 LOB 的所有成员访问。

![User Privileges](../assets/user_privilege_2.png)

由于不同的成员对不同的数据库和表执行不同的操作，因此我们建议您根据自己的业务和职位创建角色，并为每个角色赋予必要的权限，并将这些角色分配给相应的成员。如下图所示：

![User Privileges](../assets/user_privilege_3.png)

1. 将系统定义的角色 `db_admin`、`user_admin` 和 `cluster_admin` 分配给集群维护人员，将 `db_admin` 和 `user_admin` 设置为他们的默认角色进行日常维护，当需要操作集群的节点时，手动激活角色 `cluster_admin`。

   示例：

   ```SQL
   GRANT db_admin, user_admin, cluster_admin TO USER user_platform;
   ALTER USER user_platform DEFAULT ROLE db_admin, user_admin;
   ```

2. 为 LOB 中的每个成员创建用户，并为每个用户设置复杂的密码。
3. 为 LOB 中的每个职位创建角色，并将相应的权限应用于每个角色。
   对于每个 LOB 的主管，向其角色授予其 LOB 所需的最大权限集合以及相应的 `GRANT` 权限（通过在语句中指定 `WITH GRANT OPTION`）。因此，他们可以将这些权限分配给其 LOB 的成员。如果日常工作需要，则将该角色设置为他们的默认角色。
   示例：

   ```SQL
   GRANT SELECT, ALTER, INSERT, UPDATE, DELETE ON ALL TABLES IN DATABASE DB_A TO ROLE linea_admin WITH GRANT OPTION;
   GRANT SELECT, ALTER, INSERT, UPDATE, DELETE ON TABLE TABLE_C1, TABLE_C2, TABLE_C3 TO ROLE linea_admin WITH GRANT OPTION;
   GRANT linea_admin TO USER user_linea_admin;
   ALTER USER user_linea_admin DEFAULT ROLE linea_admin;
   ```
   对于分析师和高管，为他们分配具有相应权限的角色。
   示例：

   ```SQL
   GRANT SELECT ON ALL TABLES IN DATABASE DB_A TO ROLE linea_query;
   GRANT SELECT ON TABLE TABLE_C1, TABLE_C2, TABLE_C3 TO ROLE linea_query;
   GRANT linea_query TO USER user_linea_salesa;
   GRANT linea_query TO USER user_linea_salesb;
   ALTER USER user_linea_salesa DEFAULT ROLE linea_query;
   ALTER USER user_linea_salesb DEFAULT ROLE linea_query;
   ```

4. 对于所有集群用户都可以访问的数据库 `DB_PUBLIC`，将 `DB_PUBLIC` 的 `SELECT` 权限授予系统定义的角色 `public`。

   示例：

   ```SQL
   GRANT SELECT ON ALL TABLES IN DATABASE DB_PUBLIC TO ROLE public;
   ```

您可以将角色分配给其他人，以实现复杂场景下的角色继承。

例如，如果分析师需要在 `DB_PUBLIC` 中写入和查询表的权限，而高管只能查询这些表，您可以创建角色 `public_analysis` 和 `public_sales`，应用相关权限到这些角色，并将它们分配给分析师和高管的原始角色。

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

### 根据场景定制角色

<UserPrivilegeCase />
```
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