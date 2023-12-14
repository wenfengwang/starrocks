```markdown
      + {R}
      + {R}
    + {R}
  + {R}
```

```SQL
GRANT <role_name> [,<role_name>, ...] TO ROLE <role_name>
GRANT <role_name> [,<role_name>, ...] TO USER <user_identity>
```

**注意：**

- 在角色被赋予给用户之后，用户需要通过 [SET ROLE](SET_ROLE.md) 手动激活角色，方可利用角色的权限。
- 如果希望用户登录时就默认激活角色，则可以通过 [ALTER USER](ALTER_USER.md) 或者 [SET DEFAULT ROLE](SET_DEFAULT_ROLE.md) 为用户设置默认角色。
- 如果希望系统内所有用户都能够在登录时默认激活所有权限，则可以设置全局变量 `SET GLOBAL activate_all_roles_on_login = TRUE;`。

## 示例

示例一：将所有数据库及库中所有表的读取权限授予用户 `jack` 。

```SQL
GRANT SELECT ON *.* TO 'jack'@'%';
```

示例二：将数据库 `db1` 及库中所有表的导入权限授予角色 `my_role`。

```SQL
GRANT INSERT ON db1.* TO ROLE 'my_role';
```

示例三：将数据库 `db1` 和表 `tbl1` 的读取、结构变更和导入权限授予用户 `jack`。

```SQL
GRANT SELECT,ALTER,INSERT ON db1.tbl1 TO 'jack'@'192.8.%';
```

示例四：将所有资源的使用权限授予用户 `jack`。

```SQL
GRANT USAGE ON RESOURCE * TO 'jack'@'%';
```

示例五：将资源 `spark_resource` 的使用权限授予用户 `jack`。

```SQL
GRANT USAGE ON RESOURCE 'spark_resource' TO 'jack'@'%';
```

示例六：将资源 `spark_resource` 的使用权限授予角色 `my_role` 。

```SQL
GRANT USAGE ON RESOURCE 'spark_resource' TO ROLE 'my_role';
```

示例七：将表 `sr_member` 的 SELECT 权限授予给用户 `jack`，并允许 `jack` 将此权限授予其他用户或角色（通过在 SQL 中指定 WITH GRANT OPTION）：

```SQL
GRANT SELECT ON TABLE sr_member TO USER jack@'172.10.1.10' WITH GRANT OPTION;
```

示例八：将系统预置角色 `db_admin`、`user_admin` 以及 `cluster_admin` 赋予给平台运维角色。

```SQL
GRANT db_admin, user_admin, cluster_admin TO USER user_platform;
```

示例九：授予用户 `jack` 以用户 `rose` 的身份执行操作的权限。

```SQL
GRANT IMPERSONATE ON 'rose'@'%' TO 'jack'@'%';
```

## 最佳实践 - 基于使用场景创建自定义角色

<UserPrivilegeCase />

有关多业务线权限管理的相关实践，参见 [多业务线权限管理](../../../administration/User_privilege.md#多业务线权限管理)。