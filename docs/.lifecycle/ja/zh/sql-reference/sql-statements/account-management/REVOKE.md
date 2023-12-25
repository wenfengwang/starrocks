---
displayed_sidebar: Chinese
---

# REVOKE

## 機能

ユーザーやロールから特定の権限やロールを取り消します。StarRocksがサポートする権限項目については、[権限項目](../../../administration/privilege_item.md)を参照してください。

> **注意**
>
> - 一般ユーザーは、WITH GRANT OPTION キーワードを含む自分が持っている権限を他のユーザーやロールから取り消すことができます。WITH GRANT OPTIONについては、[GRANT](GRANT.md)を参照してください。
> - `user_admin` ロールを持つユーザーのみが他のユーザーの権限を取り消すことができます。

## 文法

### 特定の権限を取り消す

取り消せる権限はオブジェクトによって異なります。以下に異なるオブジェクトで取り消せる権限について説明します。

#### システム関連

```SQL
REVOKE
    { CREATE RESOURCE GROUP | CREATE RESOURCE | CREATE EXTERNAL CATALOG | REPOSITORY | BLACKLIST | FILE | OPERATE | CREATE STORAGE VOLUME } 
    ON SYSTEM
    FROM { ROLE | USER } {<role_name>|<user_identity>}
```

#### リソースグループ関連

```SQL
REVOKE
    { ALTER | DROP | ALL [PRIVILEGES] } 
    ON { RESOURCE GROUP <resourcegroup_name> [, <resourcegroup_name>,...] | ALL RESOURCE GROUPS}
    FROM { ROLE | USER } {<role_name>|<user_identity>}
```

#### リソース関連

```SQL
REVOKE
    { USAGE | ALTER | DROP | ALL [PRIVILEGES] } 
    ON { RESOURCE <resource_name> [, <resource_name>,...] | ALL RESOURCES } 
    FROM { ROLE | USER } {<role_name>|<user_identity>}
```

#### ユーザー関連

```SQL
REVOKE IMPERSONATE ON USER <user_identity> FROM USER <user_identity_1>
```

#### グローバルUDF関連

```SQL
REVOKE
    { USAGE | DROP | ALL [PRIVILEGES]} 
    ON { GLOBAL FUNCTION <function_name>(input_data_type) [, <function_name>(input_data_type),...]    
       | ALL GLOBAL FUNCTIONS }
    FROM { ROLE | USER } {<role_name>|<user_identity>}
```

#### インターナルカタログ関連

```SQL
REVOKE
    { USAGE | CREATE DATABASE | ALL [PRIVILEGES]} 
    ON CATALOG default_catalog
    FROM { ROLE | USER } {<role_name>|<user_identity>}
```

#### エクスターナルカタログ関連

```SQL
REVOKE  
   { USAGE | DROP | ALL [PRIVILEGES] }
   ON { CATALOG <catalog_name> [, <catalog_name>,...] | ALL CATALOGS}
   FROM { ROLE | USER } {<role_name>|<user_identity>}
```

#### データベース関連

```SQL
REVOKE 
    { ALTER | DROP | CREATE TABLE | CREATE VIEW | CREATE FUNCTION | CREATE MATERIALIZED VIEW | ALL [PRIVILEGES] } 
    ON { DATABASE <database_name> [, <database_name>,...] | ALL DATABASES }
    FROM { ROLE | USER } {<role_name>|<user_identity>}
```

注意：SET CATALOGを実行した後でないと使用できません。

#### テーブル関連

```SQL
REVOKE
    { ALTER | DROP | SELECT | INSERT | EXPORT | UPDATE | DELETE | ALL [PRIVILEGES]} 
    ON { TABLE <table_name> [, <table_name>,...]
       | ALL TABLES } IN 
           { DATABASE <database_name> [, <database_name>,...] | ALL DATABASES }
    FROM { ROLE | USER } {<role_name>|<user_identity>}
```

注意：SET CATALOGを実行した後でないと使用できません。テーブルは `db.tbl` の形式で指定することもできます。

```SQL
REVOKE <priv> ON TABLE db.tbl FROM {ROLE <role_name> | USER <user_identity>}
```

#### ビュー関連

```SQL
REVOKE
    { ALTER | DROP | SELECT | ALL [PRIVILEGES]} 
    ON { VIEW <view_name> [, <view_name>,...]
       | ALL VIEWS } IN 
           { DATABASE <database_name> [, <database_name>,...] | ALL DATABASES }
    FROM { ROLE | USER } {<role_name>|<user_identity>}
```

注意：SET CATALOGを実行した後でないと使用できません。ビューは `db.view` の形式で指定することもできます。

```SQL
REVOKE <priv> ON VIEW db.view FROM {ROLE <role_name> | USER <user_identity>}
```

#### マテリアライズドビュー関連

```SQL
REVOKE
    { SELECT | ALTER | REFRESH | DROP | ALL [PRIVILEGES]} 
    ON { MATERIALIZED VIEW <mv_name> [, <mv_name>,...]
       | ALL MATERIALIZED VIEWS } IN 
           { DATABASE <database_name> [, <database_name>,...] | ALL DATABASES }
    FROM { ROLE | USER } {<role_name>|<user_identity>}
```

注意：SET CATALOGを実行した後でないと使用できません。マテリアライズドビューは `db.mv` の形式で指定することもできます。

```SQL
REVOKE <priv> ON MATERIALIZED VIEW db.mv FROM {ROLE <role_name> | USER <user_identity>};
```

#### 関数関連

```SQL
REVOKE
    { USAGE | DROP | ALL [PRIVILEGES]} 
    ON { FUNCTION <function_name>(input_data_type) [, <function_name>(input_data_type),...]
       | ALL FUNCTIONS } IN 
           { DATABASE <database_name> [, <database_name>,...] | ALL DATABASES }
    FROM { ROLE | USER } {<role_name>|<user_identity>}
```

注意：SET CATALOGを実行した後でないと使用できません。関数は `db.function` の形式で指定することもできます。

```SQL
REVOKE <priv> ON FUNCTION db.function FROM {ROLE <role_name> | USER <user_identity>}
```

#### ストレージボリューム関連

```SQL
REVOKE
    { USAGE | ALTER | DROP | ALL [PRIVILEGES] } 
    ON { STORAGE VOLUME <name> [, <name>,...] | ALL STORAGE VOLUMES } 
    FROM { ROLE | USER } {<role_name>|<user_identity>}
```

### 特定のロールを取り消す

```SQL
REVOKE <role_name> [, <role_name>, ...] FROM ROLE <role_name>
REVOKE <role_name> [, <role_name>, ...] FROM USER <user_identity>
```

## パラメータ説明

| **パラメータ**     | **説明**                        |
| ------------------ | ------------------------------- |
| role_name          | ロール名                        |
| user_identity      | ユーザー識別子、例：'jack'@'192.%'。 |
| resourcegroup_name | リソースグループ名              |
| resource_name      | リソース名                      |
| function_name      | 関数名                          |
| catalog_name       | エクスターナルカタログ名        |
| database_name      | データベース名                  |
| table_name         | テーブル名                      |
| view_name          | ビュー名                        |
| mv_name            | マテリアライズドビュー名        |

## 例

### 権限を取り消す

ユーザー `jack` からテーブル `sr_member` の SELECT 権限を取り消します。

```SQL
REVOKE SELECT ON TABLE sr_member FROM USER 'jack'@'172.10.1.10';
```

ロール `test_role` からリソース `spark_resource` の使用権限を取り消します。

```SQL
REVOKE USAGE ON RESOURCE 'spark_resource' FROM ROLE 'test_role';
```

### ロールを取り消す

以前ユーザー `jack` に付与された `example_role` ロールを取り消します。

```SQL
REVOKE example_role FROM 'jack'@'%';
```

以前ロール `test_role` に付与された `example_role` ロールを取り消します。

```SQL
REVOKE example_role FROM ROLE 'test_role';
```
