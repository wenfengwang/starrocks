---
displayed_sidebar: English
---

# 权限常见问题解答

## 为什么在为用户分配了所需角色之后，还是会出现“无权限”的错误提示？

这个错误可能是因为角色没有被激活。您可以执行 `select current_role();` 来查询当前会话中已激活的用户角色。如果所需角色未激活，请执行 [SET ROLE](../sql-reference/sql-statements/account-management/SET_ROLE.md) 来激活该角色，并用它来进行操作。

如果您希望用户登录时角色能够自动激活，`user_admin` 角色可以运行 [SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md) 或 [ALTER USER DEFAULT ROLE](../sql-reference/sql-statements/account-management/ALTER_USER.md) 来为每个用户设置默认角色。设置了默认角色之后，用户登录时会自动激活该角色。

如果您想要所有用户在登录时自动激活他们被分配的所有角色，您可以执行以下命令。这一操作需要系统级别的 OPERATE 权限。

```SQL
SET GLOBAL activate_all_roles_on_login = TRUE;
```

然而，我们建议您按照“最小权限”原则设定权限有限的默认角色，以此来防范潜在的风险。例如：

- 对于一般用户，您可以将仅具有 SELECT 权限的 read_only 角色设为默认角色，避免将具备 ALTER、DROP、INSERT 等权限的角色设为默认角色。
- 对于管理员，您可以将 db_admin 角色设为默认角色，但应避免将具有增加和删除节点权限的 node_admin 角色设为默认角色。

这种做法有助于确保用户被分配到合适权限的角色，减少非预期操作的风险。

您可以运行[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)来分配所需的权限或角色给用户。

## 我已经授予用户数据库中所有表的权限（GRANT ALL ON ALL TABLES IN DATABASE <db_name> TO USER <user_identity>;），但用户仍然无法在数据库中创建表格。这是为什么？

在数据库中创建表格需要数据库级别的 CREATE TABLE 权限。您需要授予用户这项权限。

```SQL
GRANT CREATE TABLE ON DATABASE <db_name> TO USER <user_identity>;;
```

## 我已经使用 GRANT ALL ON DATABASE <db_name> TO USER <user_identity>; 授予用户数据库的所有权限，但当用户在这个数据库中执行 SHOW TABLES; 时，没有任何内容返回。这是为什么？

SHOW TABLES; 只返回用户至少拥有一项权限的表格。如果用户对某张表没有任何权限，那么这张表就不会被返回。您可以为用户授予对这个数据库中所有表的任何权限（例如可以使用 SELECT）：

```SQL
GRANT SELECT ON ALL TABLES IN DATABASE <db_name> TO USER <user_identity>;
```

上述语句等同于在 v3.0 版本之前使用的 GRANT select_priv ON db.* TO <user_identity>;。
