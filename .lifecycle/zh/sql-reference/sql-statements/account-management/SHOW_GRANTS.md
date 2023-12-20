---
displayed_sidebar: English
---

# 展示权限

## 描述

显示已授予给用户或角色的所有权限。

如需了解更多关于角色和权限的信息，请查看[权限概览](../../../administration/privilege_overview.md)。

> 注意：所有角色和用户均可查看授予给自己的权限或分配给自己的角色。然而，只有 user_admin 角色能够查看特定用户或角色的权限。

## 语法

```SQL
SHOW GRANTS; -- View the privileges of the current user.
SHOW GRANTS FOR ROLE <role_name>; -- View the privileges of a specific role.
SHOW GRANTS FOR <user_identity>; -- View the privileges of a specific user.
```

## 参数

- role_name：角色名称
- user_identity：用户标识

返回字段：

```SQL
-- View the privileges of a specific user.
+--------------+--------+---------------------------------------------+
|UserIdentity  |Catalog | Grants                                      |
+--------------+--------+---------------------------------------------+

-- View the privileges of a specific role.
+-------------+--------+-------------------------------------------------------+
|RoleName     |Catalog | Grants                                                |
+-------------+-----------------+----------------------------------------------+
```

|字段|描述|
|---|---|
|UserIdentity|用户身份，查询用户权限时显示。|
|RoleName|角色名称，查询角色权限时显示。|
|Catalog|目录名称。如果对 StarRocks 内部目录执行 GRANT 操作，则返回默认值。如果对外部目录执行 GRANT 操作，则返回外部目录的名称。如果执行中所示的操作，则返回 NULL授予列正在分配角色。|
|Grants|具体的 GRANT 操作。|

## 示例

```SQL
mysql> SHOW GRANTS;
+--------------+---------+----------------------------------------+
| UserIdentity | Catalog | Grants                                 |
+--------------+---------+----------------------------------------+
| 'root'@'%'   | NULL    | GRANT 'root', 'testrole' TO 'root'@'%' |
+--------------+---------+----------------------------------------+

mysql> SHOW GRANTS FOR 'user_g'@'%';
+-------------+-------------+-----------------------------------------------------------------------------------------------+
|UserIdentity |Catalog      |Grants                                                                                         |
+-------------+-------------------------------------------------------------------------------------------------------------+
|'user_g'@'%' |NULL         |GRANT role_g, public to `user_g`@`%`;                                                          | 
|'user_g'@'%' |NULL         |GRANT IMPERSONATE ON `user_a`@`%`, `user_b`@`%`TO `user_g`@`%`;                                |    
|'user_g'@'%' |default      |GRANT CREATE_DATABASE ON CATALOG default_catalog TO USER `user_g`@`%`;                         | 
|'user_g'@'%' |default      |GRANT ALTER, DROP, CREATE_TABLE ON DATABASE db1 TO USER `user_g`@`%`;                          | 
|'user_g'@'%' |default      |GRANT CREATE_VIEW ON DATABASE db1 TO USER `user_g`@`%` WITH GRANT OPTION;                      | 
|'user_g'@'%' |default      |GRANT ALTER, DROP, SELECT, INGEST, EXPORT, DELETE, UPDATE ON TABLE db.* TO USER `user_g`@`%`;  | 
|'user_g'@'%' |default      |GRANT ALTER, DROP, SELECT ON VIEW db2.view TO USER `user_g`@`%`;                               | 
|'user_g'@'%' |Hive_catalog |GRANT USAGE ON CATALOG Hive_catalog TO USER `user_g`@`%`                                       |
+-------------+--------------+-----------------------------------------------------------------------------------------------+

mysql> SHOW GRANTS FOR ROLE role_g;
+-------------+--------+-------------------------------------------------------+
|RoleName     |Catalog | Grants                                                |
+-------------+-----------------+----------------------------------------------+
|role_g       |NULL    | GRANT role_p, role_test TO ROLE role_g;               | 
|role_g       |default | GRANT SELECT ON *.* TO ROLE role_g WITH GRANT OPTION; | 
+-------------+--------+--------------------------------------------------------+
```

## 参考资料

[GRANT](GRANT.md)
