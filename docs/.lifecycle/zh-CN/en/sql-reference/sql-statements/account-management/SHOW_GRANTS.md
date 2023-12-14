---
displayed_sidebar: "Chinese"
---

# 显示授予权限

## 描述

显示已授予用户或角色的所有权限。

有关角色和权限的更多信息，请参阅[权限概述](../../../administration/privilege_overview.md)。

> 注意: 所有角色和用户都可以查看授予它们的权限或分配给它们的角色。然而，只有`user_admin`角色才能查看特定用户或角色的权限。

## 语法

```SQL
SHOW GRANTS; -- 查看当前用户的权限。
SHOW GRANTS FOR ROLE <role_name>; -- 查看特定角色的权限。
SHOW GRANTS FOR <user_identity>; -- 查看特定用户的权限。
```

## 参数

- role_name
- user_identity

返回字段信息：

```SQL
-- 查看特定用户的权限。
+--------------+--------+---------------------------------------------+
|用户标识       |目录    | 授权                                        |
+--------------+--------+---------------------------------------------+

-- 查看特定角色的权限。
+-------------+--------+-------------------------------------------------------+
|角色名称     |目录    | 授权                                                |
+-------------+-----------------+----------------------------------------------+
```

| **字段**       | **描述**                                                    |
| -------------- | ------------------------------------------------------------ |
| 用户标识       | 查询用户权限时显示的用户标识。                                |
| 角色名称       | 查询角色权限时显示的角色名称。                                |
| 目录           | 目录名称。<br />如果在StarRocks内部目录上执行GRANT操作，则返回`default`。<br />如果在外部目录上执行GRANT操作，则返回外部目录的名称。<br />如果在`Grants`栏中显示的操作是分配角色，则返回`NULL`。 |
| 授权           | 具体的GRANT操作。                                            |

## 示例

```SQL
mysql> SHOW GRANTS;
+--------------+---------+----------------------------------------+
| 用户标识      | 目录    | 授权                                   |
+--------------+---------+----------------------------------------+
| 'root'@'%'   | NULL    | GRANT 'root', 'testrole' TO 'root'@'%' |
+--------------+---------+----------------------------------------+

mysql> SHOW GRANTS FOR 'user_g'@'%';
+-------------+-------------+-----------------------------------------------------------------------------------------------+
|用户标识       |目录         |授权                                                                                         |
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
|角色名称     |目录    | 授权                                                |
+-------------+-----------------+----------------------------------------------+
|role_g       |NULL    | GRANT role_p, role_test TO ROLE role_g;               | 
|role_g       |default | GRANT SELECT ON *.* TO ROLE role_g WITH GRANT OPTION; | 
+-------------+--------+--------------------------------------------------------+
```

## 参考

[授权](GRANT.md)