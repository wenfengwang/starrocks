---
displayed_sidebar: English
---

# SHOW GRANTS

## 説明

ユーザーやロールに付与されたすべての権限を表示します。

ロールと権限に関する詳細は、[権限の概要](../../../administration/privilege_overview.md)を参照してください。

> 注意: すべてのロールとユーザーは自分に付与された権限や割り当てられたロールを確認できます。しかし、`user_admin` ロールのみが特定のユーザーやロールの権限を確認することができます。

## 構文

```SQL
SHOW GRANTS; -- 現在のユーザーの権限を表示します。
SHOW GRANTS FOR ROLE <role_name>; -- 特定のロールの権限を表示します。
SHOW GRANTS FOR <user_identity>; -- 特定のユーザーの権限を表示します。
```

## パラメーター

- role_name
- user_identity

戻り値のフィールド:

```SQL
-- 特定のユーザーの権限を表示します。
+--------------+--------+---------------------------------------------+
|UserIdentity  |Catalog | Grants                                      |
+--------------+--------+---------------------------------------------+

-- 特定のロールの権限を表示します。
+-------------+--------+-------------------------------------------------------+
|RoleName     |Catalog | Grants                                                |
+-------------+--------+-------------------------------------------------------+
```

| **フィールド** | **説明**                                              |
| ------------ | ------------------------------------------------------------ |
| UserIdentity | ユーザーの権限を問い合わせる際に表示されるユーザー識別子です。 |
| RoleName     | ロールの権限を問い合わせる際に表示されるロール名です。 |
| Catalog      | カタログ名。<br />`default` は、StarRocks 内部カタログに対する GRANT 操作が行われた場合に返されます。<br />外部カタログに対する GRANT 操作が行われた場合は、その外部カタログの名前が返されます。<br />`NULL` は、`Grants` 列に表示される操作がロールの割り当てである場合に返されます。 |
| Grants       | 特定の GRANT 操作です。                                |

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
+-------------+-------------+-----------------------------------------------------------------------------------------------+
|'user_g'@'%' |NULL         |GRANT role_g, public TO `user_g`@`%`;                                                          | 
|'user_g'@'%' |NULL         |GRANT IMPERSONATE ON `user_a`@`%`, `user_b`@`%` TO `user_g`@`%`;                               |    
|'user_g'@'%' |default      |GRANT CREATE_DATABASE ON CATALOG default_catalog TO USER `user_g`@`%`;                         | 
|'user_g'@'%' |default      |GRANT ALTER, DROP, CREATE_TABLE ON DATABASE db1 TO USER `user_g`@`%`;                          | 
|'user_g'@'%' |default      |GRANT CREATE_VIEW ON DATABASE db1 TO USER `user_g`@`%` WITH GRANT OPTION;                      | 
|'user_g'@'%' |default      |GRANT ALTER, DROP, SELECT, INGEST, EXPORT, DELETE, UPDATE ON TABLE db1.* TO USER `user_g`@`%`; | 
|'user_g'@'%' |default      |GRANT ALTER, DROP, SELECT ON VIEW db2.view TO USER `user_g`@`%`;                               | 
|'user_g'@'%' |Hive_catalog |GRANT USAGE ON CATALOG Hive_catalog TO USER `user_g`@`%`                                       |
+-------------+-------------+-----------------------------------------------------------------------------------------------+

mysql> SHOW GRANTS FOR ROLE role_g;
+-------------+--------+-------------------------------------------------------+
|RoleName     |Catalog | Grants                                                |
+-------------+--------+-------------------------------------------------------+
|role_g       |NULL    | GRANT role_p, role_test TO ROLE role_g;               | 
|role_g       |default | GRANT SELECT ON *.* TO ROLE role_g WITH GRANT OPTION; | 
+-------------+--------+-------------------------------------------------------+
```

## 参照

[GRANT](GRANT.md)
