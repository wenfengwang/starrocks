---
displayed_sidebar: "Japanese"
---

# REVOKE（取り消し）

## 説明

ユーザーまたはロールから特定の権限またはロールを取り消します。 StarRocksがサポートする権限については、[StarRocksでサポートされる権限](../../../administration/privilege_item.md)を参照してください。

> 注：この操作は `user_admin` ロールのみ実行できます。

## 構文

### 権限を取り消す

取り消すことができる権限はオブジェクトに固有です。以下に、オブジェクトに基づいた構文を示します。

#### システム

```SQL
REVOKE
    { CREATE RESOURCE GROUP | CREATE RESOURCE | CREATE EXTERNAL CATALOG | REPOSITORY | BLACKLIST | FILE | OPERATE } 
    ON SYSTEM
    FROM { ROLE | USER} {<role_name>|<user_identity>}
```

#### リソースグループ

```SQL
REVOKE
    { ALTER | DROP | ALL [PRIVILEGES] } 
    ON { RESOURCE GROUP <resourcegroup_name> [, <resourcegroup_name>,...] ｜ ALL RESOURCE GROUPS} 
    FROM { ROLE | USER} {<role_name>|<user_identity>}
```

#### リソース

```SQL
REVOKE
    { USAGE | ALTER | DROP | ALL [PRIVILEGES] } 
    ON { RESOURCE <resource_name> [, <resource_name>,...] ｜ ALL RESOURCES} 
    FROM { ROLE | USER} {<role_name>|<user_identity>}
```

#### ユーザー

```SQL
REVOKE IMPERSONATE ON USER <user_identity> FROM USER <user_identity>;
```

#### グローバルUDF

```SQL
REVOKE
    { USAGE | DROP | ALL [PRIVILEGES]} 
    ON { GLOBAL FUNCTION <function_name> [, <function_name>,...]    
       | ALL GLOBAL FUNCTIONS }
    FROM { ROLE | USER} {<role_name>|<user_identity>}
```

#### 内部カタログ

```SQL
REVOKE 
    { USAGE | CREATE DATABASE | ALL [PRIVILEGES]} 
    ON CATALOG default_catalog
    FROM { ROLE | USER} {<role_name>|<user_identity>}
```

#### 外部カタログ

```SQL
REVOKE  
   { USAGE | DROP | ALL [PRIVILEGES] } 
   ON { CATALOG <catalog_name> [, <catalog_name>,...] | ALL CATALOGS}
   FROM { ROLE | USER} {<role_name>|<user_identity>}
```

#### データベース

```SQL
REVOKE 
    { ALTER | DROP | CREATE TABLE | CREATE VIEW | CREATE FUNCTION | CREATE MATERIALIZED VIEW | ALL [PRIVILEGES] } 
    ON {{ DATABASE <database_name> [, <database_name>,...]} | ALL DATABASES }
    FROM { ROLE | USER} {<role_name>|<user_identity>}
```

* このコマンドを実行する前に、まず SET CATALOG を実行する必要があります。

#### テーブル

```SQL
REVOKE  
    { ALTER | DROP | SELECT | INSERT | EXPORT | UPDATE | DELETE | ALL [PRIVILEGES]} 
    ON { TABLE <table_name> [, < table_name >,...]
       | ALL TABLES} IN 
           { { DATABASE <database_name> [, <database_name>,...]} | ALL DATABASES }
    FROM { ROLE | USER} {<role_name>|<user_identity>}
```

* このコマンドを実行する前に、まず SET CATALOG を実行する必要があります。
* テーブルを表すには、db.tbl を使用することもできます。

  ```SQL
  REVOKE <priv> ON TABLE db.tbl FROM {ROLE <role_name> | USER <user_identity>}
  ```

#### ビュー

```SQL
REVOKE  
    { ALTER | DROP | SELECT | ALL [PRIVILEGES]} 
    ON { VIEW <view_name> [, < view_name >,...]
       ｜ ALL VIEWS} IN 
           { { DATABASE <database_name> [, <database_name>,...]}  | ALL DATABASES }
    FROM { ROLE | USER} {<role_name>|<user_identity>}
```

* このコマンドを実行する前に、まず SET CATALOG を実行する必要があります。
* ビューを表すには、db.view を使用することもできます。

  ```SQL
  REVOKE <priv> ON VIEW db.view FROM {ROLE <role_name> | USER <user_identity>}
  ```

#### マテリアライズドビュー

```SQL
REVOKE
    { SELECT | ALTER | REFRESH | DROP | ALL [PRIVILEGES]} 
    ON { MATERIALIZED VIEW <mv_name> [, < mv_name >,...]
       ｜ ALL MATERIALIZED VIEWS} IN 
           { { DATABASE <database_name> [, <database_name>,...] } | ALL [DATABASES] }
    FROM { ROLE | USER} {<role_name>|<user_identity>}
```

* このコマンドを実行する前に、まず SET CATALOG を実行する必要があります。
* マテリアライズドビューを表すには、db.mv を使用することもできます。

  ```SQL
  REVOKE <priv> ON MATERIALIZED VIEW db.mv FROM {ROLE <role_name> | USER <user_identity>}
  ```

#### 関数

```SQL
REVOKE
    { USAGE | DROP | ALL [PRIVILEGES]} 
    ON { FUNCTION <function_name> [, < function_name >,...]
       ｜ ALL FUNCTIONS} IN 
           { { DATABASE <database_name> [, <database_name>,...] } | ALL DATABASES }
    FROM { ROLE | USER} {<role_name>|<user_identity>}
```

* このコマンドを実行する前に、まず SET CATALOG を実行する必要があります。
* 関数を表すには、db.function を使用することもできます。

  ```SQL
  REVOKE <priv> ON FUNCTION db.function FROM {ROLE <role_name> | USER <user_identity>}
  ```

#### ストレージボリューム

```SQL
REVOKE
    CREATE STORAGE VOLUME 
    ON SYSTEM
    FROM { ROLE | USER} {<role_name>|<user_identity>}

REVOKE
    { USAGE | ALTER | DROP | ALL [PRIVILEGES] } 
    ON { STORAGE VOLUME < name > [, < name >,...] ｜ ALL STORAGE VOLUME} 
    FROM { ROLE | USER} {<role_name>|<user_identity>}
```

### ロールを取り消す

```SQL
REVOKE <role_name> [,<role_name>, ...] FROM ROLE <role_name>
REVOKE <role_name> [,<role_name>, ...] FROM USER <user_identity>
```

## パラメーター

| **パラメーター**  | **説明**                           |
| ----------------- | ---------------------------------- |
| role_name         | ロール名                           |
| user_identity     | 例：'jack'@'192.%' のようなユーザーID  |
| resourcegroup_name| リソースグループ名                 |
| resource_name     | リソース名                         |
| function_name     | 関数名                             |
| catalog_name      | 外部カタログ名                     |
| database_name     | データベース名                     |
| table_name        | テーブル名                         |
| view_name         | ビュー名                           |
| mv_name           | マテリアライズドビュー名           |

## 例

### 権限の取り消し

テーブル `sr_member` からユーザー `jack` の SELECT 権限を取り消す：

```SQL
REVOKE SELECT ON TABLE sr_member FROM USER 'jack'@'192.%'
```

リソース `spark_resource` からロール `test_role` の USAGE 権限を取り消す：

```SQL
REVOKE USAGE ON RESOURCE 'spark_resource' FROM ROLE 'test_role';
```

### ロールの取り消し

ユーザー `jack` からロール `example_role` を取り消す：

```SQL
REVOKE example_role FROM 'jack'@'%';
```

ロール `test_role` からロール `example_role` を取り消す：

```SQL
REVOKE example_role FROM ROLE 'test_role';
```

## 参照

[GRANT（付与）](GRANT.md)