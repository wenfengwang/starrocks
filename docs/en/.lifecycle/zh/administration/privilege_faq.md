---
displayed_sidebar: English
---

# 特权常见问题

## 为什么即使已经为用户分配了所需的角色，用户在执行操作时仍然报告“无权限”错误消息？

如果角色未被激活，就可能会出现这个错误。您可以运行 `select current_role();` 来查询当前会话中已激活的用户角色。如果所需角色未被激活，可以运行 [SET ROLE](../sql-reference/sql-statements/account-management/SET_ROLE.md) 来激活该角色，然后使用该角色执行操作。

如果希望角色在登录时自动激活，`user_admin` 角色可以运行 [SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md) 或 [ALTER USER DEFAULT ROLE](../sql-reference/sql-statements/account-management/ALTER_USER.md) 来为每个用户设置默认角色。设置默认角色后，用户登录时将自动激活该角色。

如果希望所有用户被分配的角色在登录时自动激活，可以运行以下命令。此操作需要系统级别的 OPERATE 权限。

```SQL
SET GLOBAL activate_all_roles_on_login = TRUE;
```

但是，我们建议您遵循“最小权限”原则，通过设置默认角色并限制权限，以防止潜在风险。例如：

- 对于普通用户，可以将只具有 SELECT 权限的 `read_only` 角色设置为默认角色，同时避免将具有 ALTER、DROP 和 INSERT 等权限的角色设置为默认角色。
- 对于管理员，可以将 `db_admin` 角色设置为默认角色，同时避免将具有添加和删除节点权限的 `node_admin` 角色设置为默认角色。

这种方法有助于确保为用户分配适当权限的角色，从而降低意外操作的风险。

您可以运行 [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) 来为用户分配所需的权限或角色。

## 我已经向用户授予了在数据库中所有表上的权限（`GRANT ALL ON ALL TABLES IN DATABASE <db_name> TO USER <user_identity>;`），但用户仍然无法在数据库中创建表。为什么？

在数据库中创建表需要数据库级别的 CREATE TABLE 权限。您需要向用户授予这个权限。

```SQL
GRANT CREATE TABLE ON DATABASE <db_name> TO USER <user_identity>;;
```

## 我已经使用 `GRANT ALL ON DATABASE <db_name> TO USER <user_identity>;` 授予用户对数据库的所有权限，但当用户在该数据库中运行 `SHOW TABLES;` 时，却没有返回任何内容。为什么？

`SHOW TABLES;` 仅返回用户具有任何权限的表。如果用户对某个表没有权限，那么这个表就不会被返回。您可以向用户授予对该数据库中所有表的任何权限（例如，使用 SELECT）：

```SQL
GRANT SELECT ON ALL TABLES IN DATABASE <db_name> TO USER <user_identity>;
```

上述语句相当于在 v3.0 之前的版本中使用 `GRANT select_priv ON db.* TO <user_identity>;`。