---
displayed_sidebar: English
---

# 特权常见问题解答

## 为什么在分配了所需角色给用户之后，还是报告“没有权限”的错误信息？

这个错误可能发生在角色没有被激活的情况下。您可以运行 `select current_role();` 来查询当前会话中已经激活的角色。如果所需角色没有被激活，运行 [SET ROLE](../sql-reference/sql-statements/account-management/SET_ROLE.md) 来激活这个角色，并使用它来执行操作。

如果您希望用户登录时角色能够自动激活，`user_admin` 角色可以执行 [SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md) 或 [ALTER USER DEFAULT ROLE](../sql-reference/sql-statements/account-management/ALTER_USER.md) 来为每个用户设置一个默认角色。设置默认角色后，用户登录时会自动激活该角色。

如果您希望所有用户在登录时自动激活所有已分配的角色，您可以执行以下命令。这个操作需要在系统级别的 OPERATE 权限。

```SQL
SET GLOBAL activate_all_roles_on_login = TRUE;
```

然而，我们建议您遵循“最小权限”原则，为用户设置具有限制性权限的默认角色，以预防潜在的风险。例如：

- 对于普通用户，您可以设置只有 SELECT 权限的 `read_only` 角色作为默认角色，同时避免将具有 ALTER、DROP、INSERT 等权限的角色设为默认角色。
- 对于管理员，您可以设置 `db_admin` 角色作为默认角色，同时避免将具有增加和删除节点权限的 `node_admin` 角色设为默认角色。

这种做法有助于确保用户被分配到合适权限的角色，减少了不预期操作的风险。

您可以执行 [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) 来为用户分配所需的权限或角色。

## 我已经授予了用户对一个数据库中所有表的权限（`GRANT ALL ON ALL TABLES IN DATABASE <db_name> TO USER <user_identity>;`），但用户仍然不能在该数据库中创建表。这是为什么？

在数据库中创建表需要数据库级别的 CREATE TABLE 权限。您需要授予用户这个权限。

```SQL
GRANT CREATE TABLE ON DATABASE <db_name> TO USER <user_identity>;
```

## 我已经使用 `GRANT ALL ON DATABASE <db_name> TO USER <user_identity>;` 授予了用户数据库的所有权限，但当用户在这个数据库中运行 `SHOW TABLES;` 时，没有任何内容被返回。这是为什么？

`SHOW TABLES;` 只返回用户有权限的表。如果用户对某个表没有任何权限，那么这个表将不会被返回。您可以授予用户对这个数据库中所有表的任何权限（例如使用 SELECT）：

```SQL
GRANT SELECT ON ALL TABLES IN DATABASE <db_name> TO USER <user_identity>;
```

上述语句相当于在 v3.0 之前版本中使用的 `GRANT select_priv ON db.* TO <user_identity>;`。