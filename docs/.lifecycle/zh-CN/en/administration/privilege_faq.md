---
displayed_sidebar: "Chinese"
---

# 特权常见问题解答

## 即使向用户分配了所需的角色，为什么还是会报告“无权限”错误消息？

如果角色未激活，则可能会发生此错误。您可以运行 `select current_role();` 查询在当前会话中已激活的用户的角色。如果所需的角色未激活，请运行 [SET ROLE](../sql-reference/sql-statements/account-management/SET_ROLE.md) 以激活此角色，并使用此角色执行操作。

如果希望在登录时自动激活角色，`user_admin` 角色可以运行 [SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md) 或 [ALTER USER DEFAULT ROLE](../sql-reference/sql-statements/account-management/ALTER_USER.md) 为每个用户设置默认角色。设置默认角色后，用户登录时将自动激活该角色。

如果希望在用户登录时自动激活所有用户的所有分配角色，可以运行以下命令。此操作需要在系统级别具有 OPERATE 权限。

```SQL
SET GLOBAL activate_all_roles_on_login = TRUE;
```

但是，我们建议您遵循“最小特权”原则，通过为每个用户设置具有有限权限的默认角色来防止潜在风险。例如：

- 对于普通用户，可以将仅具有 SELECT 权限的 `read_only` 角色设置为默认角色，同时避免将具有 ALTER、DROP 和 INSERT 权限的角色设置为默认角色。
- 对于管理员，可以将 `db_admin` 角色设置为默认角色，同时避免将具有添加和删除节点权限的 `node_admin` 角色设置为默认角色。

这种方法有助于确保为用户分配适当权限的角色，从而降低意外操作的风险。

您可以运行 [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) 为用户分配所需的权限或角色。

## 我已经向用户在数据库中的所有表授予权限（`GRANT ALL ON ALL TABLES IN DATABASE <db_name> TO USER <user_identity>;`），但用户仍然无法在数据库中创建表。为什么？

在数据库中创建表需要数据库级别的 CREATE TABLE 权限。您需要向用户授予此权限。

```SQL
GRANT CREATE TABLE ON DATABASE <db_name> TO USER <user_identity>;;
```

## 我已经使用 `GRANT ALL ON DATABASE <db_name> TO USER <user_identity>;` 为用户授予数据库的所有权限，但当用户在该数据库中运行 `SHOW TABLES;` 时，什么也没有返回。为什么？

`SHOW TABLES;` 仅返回对其具有任何权限的用户的表。如果用户对表没有任何权限，则不会返回此表。您可以向用户授予在此数据库中所有表的任何权限（例如使用 SELECT）：

```SQL
GRANT SELECT ON ALL TABLES IN DATABASE <db_name> TO USER <user_identity>;
```

上述语句相当于在 v3.0 之前的版本中使用的 `GRANT select_priv ON db.* TO <user_identity>;`。