---
displayed_sidebar: "Chinese"
---

# 权限概述

本主题描述了StarRocks的权限系统的基本概念。权限确定了哪些用户可以对哪些对象执行哪些操作，从而可以更安全地以细粒度的方式管理数据和资源。

> 注意：本主题中描述的权限仅从v3.0版本开始提供。在v3.0中的权限框架和语法与早期版本不兼容。升级到v3.0后，大多数原始权限都得以保留，但对于特定操作的权限除外。有关详细差异，请参阅[权限管理](privilege_item.md)中的[升级说明]。

StarRocks采用两种权限模型：

- 基于角色的访问控制（RBAC）：权限分配给角色，然后再分配给用户。在这种情况下，权限通过角色传递给用户。
- 基于身份的访问控制（IBAC）：权限直接分配给用户身份。

因此，每个用户身份的最大权限范围是其自身权限与分配给该用户身份的角色权限的并集。

了解StarRocks的权限系统的**基本概念**：

- **对象**：可以授予访问权限的实体。除非授予，否则禁止访问。对象的示例包括CATALOG、DATABASE、TABLE和VIEW。有关详细信息，请参阅[StarRocks中支持的权限](privilege_item.md)。
- **权限**：对象的定义级别访问权限。可以使用多个权限来控制对对象授予的访问的粒度。权限是特定于对象的。不同的对象可能具有不同的权限。权限的示例包括SELECT、ALTER和DROP。
- **用户身份**：用户的唯一身份，也是可以授予权限的实体。用户身份表示为`username@'userhost'`，由用户名和用户登录时所在的IP组成。用户身份简化了属性配置。共享相同用户名的用户身份共享相同的属性。如果为用户名配置属性，则此属性对共享此用户名的所有用户身份生效。
- **角色**：可以授予权限的实体。角色是权限的抽象集合。角色可以进一步分配给用户。角色还可以分配给其他角色，创建角色层次结构。为了简化数据管理，StarRocks提供了系统定义的角色。为了提供更大的灵活性，您还可以根据业务需求创建自定义角色。

以下图示显示了在RBAC和IBAC权限模型下的权限管理示例。

在这些模型中，通过分配给角色和用户的权限，允许对对象的访问。角色可以进一步分配给其他角色或用户。

![权限管理](../assets/privilege-manage.png)

## 对象和权限

对象具有逻辑层次结构，与它们所代表的概念有关。例如，Database包含在Catalog中，而Table、View、Materialized View和Function包含在Database中。以下图示显示了StarRocks系统中的对象层次结构。

![权限对象](../assets/privilege-object.png)

每个对象都有一组可以授予的权限项。这些权限定义了可以在这些对象上执行的操作。您可以通过[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)和[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md)命令向角色或用户授予和收回权限。

## 用户

### 用户身份

在StarRocks中，每个用户由唯一用户ID标识。它由IP地址（用户主机）和用户名组成，格式为 `username @'userhost'`。StarRocks将来自不同IP地址但具有相同用户名的用户视为不同的用户身份。例如，`user1@'starrocks.com'` 和 `user1@'mirrorship.com'` 是两个用户身份。

用户身份的另一个表示形式是 `username @['domain']`，其中 `domain` 是可以由DNS解析为一组IP地址的域名。 `username @['domain']` 最终表示为一组 `username@'userhost'`。您可以使用`%`来进行`userhost`部分的模糊匹配。如果未指定`userhost`，则默认为`'%'`，这意味着具有相同名称的用户从任何主机登录。

### 向用户授予权限

用户是可以授予权限的实体。权限和角色都可以分配给用户。每个用户身份的最大权限范围是其自身权限与分配给该用户身份的角色权限的并集。StarRocks确保每个用户仅能执行经授权的操作。

我们建议在大多数情况下**使用角色传递权限**。例如，创建角色后，您可以向角色授予权限，然后将角色分配给用户。如果要授予临时或特殊权限，可以直接向用户授予这些权限。这样可以简化权限管理并提供灵活性。

## 角色

角色是可以授予权限和收回权限的实体。角色可以看作是分配给用户的一组权限，以允许他们执行所需的操作。用户可以被分配多个角色，以便他们可以使用分开的权限集执行不同的操作。为了简化管理，StarRocks建议**通过角色管理权限**。特殊和临时权限可以直接授予用户。

为了简化管理，StarRocks提供了几种具有特定权限的**系统定义角色**，这有助于满足日常管理和维护需求。您还可以灵活地**自定义角色**以满足特定业务需求和安全需求。请注意，系统定义角色的权限范围无法修改。

激活角色后，用户可以执行由角色授权的操作。您可以设置在用户登录时自动激活的**默认角色**。用户还可以在当前会话中手动激活此用户拥有的角色。

### 系统定义角色

StarRocks提供了几种类型的系统定义角色。

![角色](../assets/privilege-role.png)

- `root`: 具有全局权限。默认情况下，`root`用户拥有`root`角色。
   创建StarRocks集群后，系统会自动生成具有root权限的root用户。因为root用户和角色拥有系统的所有权限，我们建议您在后续操作中创建新用户和角色以防止任何风险操作。请妥善保管root用户的密码。
- `cluster_admin`: 具有集群管理权限，可执行与节点相关的操作，如添加或删除节点。
  `cluster_admin`具有添加、删除和退出集群节点的权限。我们建议不将`cluster_admin`或包含此角色的任何自定义角色分配为任何用户的默认角色，以防止意外的节点更改。
- `db_admin`: 具有数据库管理权限，包括在目录、数据库、表、视图、物化视图、函数、全局函数、资源组和插件上执行所有操作的权限。
- `user_admin`: 具有对用户和角色的管理权限，包括创建用户、角色和权限的权限。

  上述系统定义角色旨在聚合复杂的数据库权限，以便于日常管理。**上述角色的权限范围无法修改。**

  此外，如果您需要向所有用户授予特定权限，StarRocks还提供了一个系统定义角色`public`。

- `public`: 任何用户都拥有此角色，并且在任何会话中默认激活，包括添加新用户。`public`角色默认不具备任何权限。您可以修改此角色的权限范围。

### 自定义角色

您可以创建自定义角色以满足特定的业务需求，并修改其权限范围。同时，为了方便管理，您可以将角色分配给其他角色，以创建权限层次结构和继承。然后，与角色相关联的权限将被另一个角色继承。

#### 角色层次结构和权限继承

以下图示显示了权限继承的示例。

> 注意：角色的最大继承级别为16。继承关系不能是双向的。

![角色继承](../assets/privilege-role_inheri.png)

如图所示：

- `role_s` 被分配给 `role_p`。`role_p`隐式继承`role_s`的`priv_1`。
- `role_p` 被分配给 `role_g`，`role_g`隐式继承`role_p`的`priv_2`和`role_s`的`priv_1`。
- 当角色被分配给用户后，用户也具有此角色的权限。

### 激活角色

激活角色允许用户在当前会话下应用角色的权限。您可以使用 `SELECT CURRENT_ROLE();` 来查看当前会话中的激活角色。有关详细信息，请参阅[current_role](../sql-reference/sql-functions/utility-functions/current_role.md)。

#### 默认角色

用户在登录到集群时会自动激活默认角色。它可以是由一个或多个用户拥有的角色。管理员可以使用[CREATE USER](../sql-reference/sql-statements/account-management/CREATE_USER.md)中的`DEFAULT ROLE`关键字设置默认角色，并可以使用[ALTER USER](../sql-reference/sql-statements/account-management/ALTER_USER.md)更改默认角色。

用户还可以使用[SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md)更改其默认角色。

Default roles provide basic privilege protection for users. For example, User A has `role_query` and `role_delete`, which has query and delete privilege respectively. We recommend that you only use `role_query` as the default role to prevent data loss caused by high-risk operations such as `DELETE` or `TRUNCATE`. If you need to perform these operations, you can do it after manually setting active roles.

A user who does not have a default role still has the `public` role, which is automatically activated after the user logs in to the cluster.

#### Manually activate roles

In addition to default roles, users can also manually activate one or more existing roles within a session. You can use [SHOW GRANTS](../sql-reference/sql-statements/account-management/SHOW_GRANTS.md) to view the privileges and roles that can be activated, and use [SET ROLE](../sql-reference/sql-statements/account-management/SET_ROLE.md) to configure active roles that are effective in the current session.

Note that the SET ROLE command overwrites each other. For example, after a user logs in, the `default_role` is activated by default. Then the user runs `SET ROLE role_s`. At this time, the user has only the privileges of `role_s` and their own privileges. `default_role` is overwritten.

## References

- [Privileges supported by StarRocks](privilege_item.md)
- [Manage user privileges](User_privilege.md)