---
displayed_sidebar: English
---

# 显示角色

## 描述

展示系统中所有的角色。您可以使用 `SHOW GRANTS FOR ROLE \\_\<role\_name\\_\>;` 来查看特定角色的权限。更多信息，请参见 [SHOW GRANTS](SHOW_GRANTS.md)。该命令从 v3.0 版本开始支持。

> 注意：只有 user_admin 角色能够执行此语句。

## 语法

```SQL
SHOW ROLES
```

返回字段：

|字段|描述|
|---|---|
|名称|角色的名称。|

## 示例

展示系统中的所有角色。

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

## 参考资料

- [CREATE ROLE](CREATE_ROLE.md)（创建角色）
- [ALTER USER](ALTER_USER.md)（修改用户）
- [DROP ROLE](DROP_ROLE.md)（删除角色）
