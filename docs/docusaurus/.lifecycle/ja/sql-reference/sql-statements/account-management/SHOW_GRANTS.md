```yaml
---
displayed_sidebar: "Japanese"
---

# GRANTS 表示

## 概要

ユーザーまたはロールに付与されたすべての特権を表示します。

ロールと特権についての詳細については、[特権の概要](../../../administration/privilege_overview.md)を参照してください。

> 注意: すべてのロールとユーザーは、自分に付与された特権または割り当てられたロールを表示できます。ただし、`user_admin` ロールのみが特定のユーザーまたはロールの特権を表示できます。

## 構文

```SQL
SHOW GRANTS; -- 現在のユーザーの特権を表示します。
SHOW GRANTS FOR ROLE <role_name>; -- 特定のロールの特権を表示します。
SHOW GRANTS FOR <user_identity>; -- 特定のユーザーの特権を表示します。
```

## パラメーター

- role_name
- user_identity

戻り値:

```SQL
-- 特定のユーザーの特権を表示します。
+--------------+--------+---------------------------------------------+
|UserIdentity  |Catalog | Grants                                      |
+--------------+--------+---------------------------------------------+

-- 特定のロールの特権を表示します。
+-------------+--------+-------------------------------------------------------+
|RoleName     |Catalog | Grants                                                |
+-------------+-----------------+----------------------------------------------+
```

| **フィールド**    | **説明**                                                      |
| ------------ | ------------------------------------------------------------ |
| UserIdentity | ユーザー識別子。ユーザーの特権をクエリしたときに表示されます。      |
| RoleName     | ロール名。ロールの特権をクエリしたときに表示されます。            |
| Catalog      | カタログ名。<br />StarRocks 内部カタログに GRANT 操作を実行した場合は `default` が返されます。<br />外部カタログに GRANT 操作を実行した場合は外部カタログの名前が返されます。<br />`Grants` 列に表示される操作がロールの割り当てである場合は `NULL` が返されます。 |
| Grants       | 特定の GRANT 操作。                                           |

## 例

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

## 参照

[GRANT](GRANT.md)
```