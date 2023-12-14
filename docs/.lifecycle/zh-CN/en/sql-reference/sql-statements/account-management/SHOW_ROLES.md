---
displayed_sidebar: "Chinese"
---

# 显示角色

## 描述

显示系统中的所有角色。您可以使用 `SHOW GRANTS FOR ROLE <role_name>;` 来查看特定角色的权限。有关更多信息，请参阅[SHOW GRANTS](SHOW_GRANTS.md)。该命令从 v3.0 开始支持。


> 注：只有 `user_admin` 角色可以执行此语句。

## 语法

```SQL
SHOW ROLES
```

返回字段：

| **字段** | **描述**       |
| --------- | --------------------- |
| Name      | 角色的名称。 |

## 示例

显示系统中的所有角色。

```SQL
mysql> SHOW ROLES;
+---------------+
| Name          |
+---------------+
| root          |
| db_admin      |
| cluster_admin |
| user_admin    |
| public        |
| testrole      |
+---------------+
```

## 参考

- [CREATE ROLE](CREATE_ROLE.md)
- [ALTER USER](ALTER_USER.md)
- [DROP ROLE](DROP_ROLE.md)