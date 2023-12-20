---
displayed_sidebar: English
---

# 特权概览

本主题介绍 StarRocks 权限系统的基本概念。权限确定了哪些用户可以对哪些对象执行哪些操作，以便您能够更安全、更精细地管理数据和资源。

> 注意：本主题所描述的权限仅从 v3.0 版本开始提供。v3.0 中的权限框架和语法与早期版本不兼容。升级到 v3.0 后，除了特定操作的权限外，大多数原有权限仍将保留。详细差异请参见[升级注意事项]中的[Privileges supported in StarRocks](privilege_item.md)。

StarRocks 采用两种权限模型：

- 基于角色的访问控制（RBAC）：将权限分配给角色，然后将角色分配给用户。在这种情况下，用户通过角色获得权限。
- 基于身份的访问控制（IBAC）：将权限直接分配给用户身份。

因此，每个用户身份的最大权限范围是其自身权限和分配给该用户身份的角色权限的并集。

**理解** StarRocks 权限系统的基本概念：

- **对象**：可以授予访问权限的实体。除非获得授权，否则访问将被拒绝。对象包括 CATALOG、DATABASE、TABLE 和 VIEW 等。更多信息请参阅 [StarRocks 支持的权限](privilege_item.md)。
- **权限**：对对象定义的访问级别。可以使用多个权限来控制授予对象的访问细节。权限是针对特定对象的。不同对象可能具有不同权限。权限示例包括**SELECT**、**ALTER**和**DROP**。
- **用户身份**：用户的唯一标识，也是可以授予权限的实体。用户身份表示为 `username@'userhost'`，包括用户名和用户登录的 IP 地址。使用身份简化了属性配置。共享相同用户名的用户身份共享相同的属性。如果为用户名配置了属性，该属性对共享该用户名的所有用户身份生效。
- **角色**：可以授予权限的实体。角色是权限的抽象集合。角色可以转而分配给用户。角色还可以分配给其他角色，创建角色层次结构。为了便于数据管理，StarRocks提供了系统定义的角色。您也可以根据业务需求创建自定义角色，以获得更大的灵活性。

下图展示了在 RBAC 和 IBAC 权限模型下的权限管理示例。

在这些模型中，通过分配给角色和用户的权限允许访问对象。角色进而被分配给其他角色或用户。

![privilege management](../assets/privilege-manage.png)

## 对象和权限

对象具有逻辑层次结构，与它们代表的概念相关。例如，数据库（Database）包含在目录（Catalog）中，而表（Table）、视图（View）、物化视图（Materialized View）和函数（Function）包含在数据库中。下图显示了 StarRocks 系统中的对象层次结构。

![privilege objects](../assets/privilege-object.png)

每个对象都有一组可以被授予的权限项。这些权限定义了可以在这些对象上执行的操作。您可以通过 [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) 和 [REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md) 命令从角色或用户那里授予和撤销权限。

## 用户

### 用户身份

在 StarRocks 中，每个用户都由唯一的用户 ID 标识。它由 IP 地址（用户主机）和用户名组成，格式为 username@'userhost'。StarRocks 将具有相同用户名但来自不同 IP 地址的用户识别为不同的用户身份。例如，user1@'starrocks.com' 和 user1@'mirrorship.com' 是两个不同的用户身份。

用户身份的另一种表示方式是 username@['domain']，其中 domain 是可以通过 DNS 解析为一组 IP 地址的域名。username@['domain'] 最终表示为一组 username@'userhost'。您可以使用 % 作为 userhost 部分进行模糊匹配。如果未指定 userhost，它默认为 '%'，表示同名用户从任意主机登录。

### 授予用户权限

用户是可以被授予权限的实体。可以将权限和角色分配给用户。每个用户身份的最大权限范围是其自身权限和分配给该用户身份的角色权限的并集。StarRocks 确保每个用户只能执行其被授权的操作。

我们建议您在大多数情况下**使用角色来传递权限**。例如，创建角色后，您可以向该角色授予权限，然后将该角色分配给用户。如果您想授予临时或特殊权限，您可以直接授予用户。这样做简化了权限管理，并提供了灵活性。

## 角色

角色是可以被授予和撤销权限的实体。角色可以被视为一组可以分配给用户的权限，以便他们执行所需的操作。用户可以被分配多个角色，这样他们就可以使用不同的权限集来执行不同的操作。为了简化管理，StarRocks建议通过**角色来管理权限**。特殊和临时权限可以直接授予用户。

为了便于管理，StarRocks提供了几种具有特定权限的**系统定义角色**，以帮助您满足日常管理和维护需求。您还可以灵活地**自定义角色**以满足特定的业务和安全需求。请注意，系统定义角色的权限范围不能被修改。

激活角色后，用户可以执行角色授权的操作。您可以设置**默认角色**，该角色在用户登录时自动激活。用户也可以在当前会话中手动激活自己拥有的角色。

### 系统定义的角色

StarRocks 提供了几种系统定义的角色类型。

![roles](../assets/privilege-role.png)

- root：具有全局权限。默认情况下，root 用户拥有 root 角色。在 StarRocks 集群创建后，系统会自动生成一个具有 root 权限的 root 用户。因为 root 用户和角色拥有系统的全部权限，我们建议您为后续操作创建新用户和角色，以防止任何风险操作。请妥善保管 root 用户的密码。
- cluster_admin：具有集群管理权限，可以执行与节点相关的操作，如添加或删除节点。cluster_admin 拥有添加、删除和下线集群节点的权限。我们建议不将 cluster_admin 或任何包含此角色的自定义角色作为默认角色分配给任何用户，以防止意外的节点变更。
- db_admin：具有数据库管理权限，包括对目录、数据库、表、视图、物化视图、函数、全局函数、资源组和插件等进行所有操作的权限。
- user_admin：具有管理用户和角色的权限，包括创建用户、角色和权限的权限。
  上述系统定义的角色旨在汇总复杂的数据库权限，以便于您的日常管理。**这些角色的权限范围不可修改。**
  此外，如果您需要向所有用户授予特定权限，StarRocks 还提供了系统定义的角色 public。

- public：这个角色由任何用户拥有，并在任何会话中默认激活，包括添加新用户。public 角色默认没有任何权限。您可以修改此角色的权限范围。

### 自定义角色

您可以创建自定义角色以满足特定的业务需求，并修改它们的权限范围。同时，为了方便管理，您可以将角色分配给其他角色，以创建权限层次和继承。这样，与一个角色关联的权限将被另一个角色继承。

#### 角色层次结构和权限继承

下图显示了权限继承的一个示例。

> 注意：角色的最大继承层级数为 16。继承关系不能是双向的。

![role inheritance](../assets/privilege-role_inheri.png)

如图所示：

- role_s 被分配给 role_p。role_p 隐式地继承了 role_s 的 priv_1。
- role_p 被分配给 role_g，role_g 隐式地继承了 role_p 的 priv_2 和 role_s 的 priv_1。
- 角色分配给用户后，用户也拥有该角色的权限。

### 活跃角色

活跃角色允许用户在当前会话中应用角色的权限。您可以使用 `SELECT CURRENT_ROLE();` 来查看当前会话中的活跃角色。更多信息请参阅[current_role](../sql-reference/sql-functions/utility-functions/current_role.md)。

#### 默认角色

当用户登录集群时，默认角色会自动激活。它可以是一个或多个用户所拥有的角色。管理员可以使用`CREATE USER`命令中的`DEFAULT ROLE`关键词来设置默认角色，并可以使用`ALTER USER`命令更改默认角色。

用户也可以使用[SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md)命令更改自己的默认角色。

默认角色为用户提供了基本的权限保护。例如，用户 A 拥有 role_query 和 role_delete，它们分别具有查询和删除权限。我们建议仅将 role_query 设置为默认角色，以防止 DELETE 或 TRUNCATE 等高风险操作导致数据丢失。如果需要执行这些操作，可以在手动设置活跃角色后进行。

没有默认角色的用户仍然有 public 角色，该角色在用户登录集群后会自动激活。

#### 手动激活角色

除了默认角色，用户还可以在会话中手动激活一个或多个现有角色。您可以使用 [SHOW GRANTS](../sql-reference/sql-statements/account-management/SHOW_GRANTS.md) 命令查看可以激活的权限和角色，并使用 [SET ROLE](../sql-reference/sql-statements/account-management/SET_ROLE.md) 命令配置当前会话中有效的活跃角色。

请注意，SET ROLE 命令会相互覆盖。例如，用户登录后，默认激活了 default_role。然后用户执行 SET ROLE role_s。此时，用户仅拥有 role_s 和自己的权限，default_role 被覆盖了。

## 参考资料

- [StarRocks 支持的权限](privilege_item.md)
- [管理用户权限](User_privilege.md)
