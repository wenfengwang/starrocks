---
displayed_sidebar: English
---

# 显示角色

## 描述

显示系统中的所有角色。您可以使用 `SHOW GRANTS FOR ROLE <role_name>;` 命令查看特定角色的权限。有关详细信息，请参阅 [SHOW GRANTS](SHOW_GRANTS.md)。此命令从v3.0版本开始得到支持。


> 注意：只有 `user_admin` 角色可以执行此语句。

## 语法

```SQL
SHOW ROLES
```

返回字段：

| **字段** | **描述**       |
| --------- | --------------------- |
| Name      | 角色的名称。 |

## 例子

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

## 引用

- [创建角色](CREATE_ROLE.md)
- [修改用户](ALTER_USER.md)
- [删除角色](DROP_ROLE.md)
