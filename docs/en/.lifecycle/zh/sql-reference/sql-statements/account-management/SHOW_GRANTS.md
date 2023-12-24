---
displayed_sidebar: English
---

# 显示授权

## 描述

显示已授予用户或角色的所有权限。

有关角色和权限的详细信息，请参阅[权限概述](../../../administration/privilege_overview.md)。

> 注意：所有角色和用户都可以查看授予他们的权限或分配给他们的角色。但是，只有`user_admin`角色可以查看特定用户或角色的权限。

## 语法

```SQL
SHOW GRANTS; -- 查看当前用户的权限。
SHOW GRANTS FOR ROLE <role_name>; -- 查看特定角色的权限。
SHOW GRANTS FOR <user_identity>; -- 查看特定用户的权限。
```

## 参数

- role_name
- user_identity

返回字段：

```SQL
-- 查看特定用户的权限。
+--------------+--------+---------------------------------------------+
|UserIdentity  |Catalog | Grants                                      |
+--------------+--------+---------------------------------------------+

-- 查看特定角色的权限。
+-------------+--------+-------------------------------------------------------+
|RoleName     |Catalog | Grants                                                |
+-------------+-----------------+----------------------------------------------+
```

| **字段**    | **描述**                                              |
| ------------ | ------------------------------------------------------------ |
| UserIdentity | 用户标识，在查询用户权限时显示。 |
| RoleName     | 角色名称，查询角色权限时显示。 |
| Catalog      | 目录名称。<br />如果在 StarRocks 内部目录执行 GRANT 操作，则返回`default`。<br />如果在外部目录执行 GRANT 操作，则返回外部目录的名称。<br />如果`Grants`列中显示的操作是分配角色，则返回`NULL`。|
| Grants       | 特定的 GRANT 操作。                                |

## 例子

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
|'user_g'@'%' |NULL         |GRANT role_g, public TO `user_g`@`%`;                                                          | 
|'user_g'@'%' |NULL         |GRANT IMPERSONATE ON `user_a`@`%`, `user_b`@`%` TO `user_g`@`%`;                                |    
|'user_g'@'%' |default      |GRANT CREATE_DATABASE ON CATALOG default_catalog TO USER `user_g`@`%`;                         | 
|'user_g'@'%' |default      |GRANT ALTER, DROP, CREATE TABLE ON DATABASE db1 TO USER `user_g`@`%`;                          | 
|'user_g'@'%' |default      |GRANT CREATE VIEW ON DATABASE db1 TO USER `user_g`@`%` WITH GRANT OPTION;                      | 
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

## 引用

[GRANT](GRANT.md)
