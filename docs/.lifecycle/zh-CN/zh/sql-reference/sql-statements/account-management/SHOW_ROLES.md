---
displayed_sidebar: "Chinese"
---

# 显示角色

## 功能

显示当前系统中的所有角色。如果要查看角色的权限信息，可以使用 `SHOW GRANTS FOR ROLE <role_name>;`，具体参见 [显示授权](SHOW_GRANTS.md)。

该命令从 3.0 版本开始支持。

> 说明：只有 `user_admin` 角色有权限执行该语句。

## 语法

```SQL
SHOW ROLES
```

返回字段说明：

| **字段** | **描述**   |
| -------- | ---------- |
| Name     | 角色名称。 |

## 示例

显示当前系统中的所有角色。

```Plain
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

## 相关参考

- [创建角色](CREATE_ROLE.md)
- [修改用户](ALTER_USER.md)
- [设置角色](SET_ROLE.md)
- [删除角色](DROP_ROLE.md)