---
displayed_sidebar: "Japanese"
---

# GRANT（付与）

import UserPrivilegeCase from '../../../assets/commonMarkdown/userPrivilegeCase.md'

## 説明

ユーザーまたはロールに特定のオブジェクトに対する1つ以上の権限を付与します。

ユーザーや他のロールにロールを付与します。

付与できる権限の詳細については、[権限アイテム](../../../administration/privilege_item.md)を参照してください。

GRANT操作を実行した後、詳細な権限情報を表示するには[SHOW GRANTS](./SHOW_GRANTS.md)を実行するか、権限またはロールを取り消すには[REVOKE](REVOKE.md)を実行できます。

GRANT操作を実行する前に、関連するユーザーまたはロールが作成されていることを確認してください。詳細については、[CREATE USER](./CREATE_USER.md)と[CREATE ROLE](./CREATE_ROLE.md)を参照してください。

> **注意**
>
> `user_admin`ロールを持つユーザーのみが他のユーザーやロールに任意の権限を付与できます。
> 他のユーザーは、他のユーザーやロールにGRANT OPTIONキーワードを使用して権限を付与することができます。

## 構文

### ロールまたはユーザーに権限を付与する

#### システム

```SQL
GRANT
    { CREATE RESOURCE GROUP | CREATE RESOURCE | CREATE EXTERNAL CATALOG | REPOSITORY | BLACKLIST | FILE | OPERATE | CREATE STORAGE VOLUME } 
    ON SYSTEM
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

#### リソースグループ

```SQL
GRANT
    { ALTER | DROP | ALL [PRIVILEGES] } 
    ON { RESOURCE GROUP <resource_group_name> [, <resource_group_name >,...] ｜ ALL RESOURCE GROUPS} 
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

#### リソース

```SQL
GRANT
    { USAGE | ALTER | DROP | ALL [PRIVILEGES] } 
    ON { RESOURCE <resource_name> [, < resource_name >,...] ｜ ALL RESOURCES} 
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

#### グローバルUDF

```SQL
GRANT
    { USAGE | DROP | ALL [PRIVILEGES]} 
    ON { GLOBAL FUNCTION <function_name>(input_data_type) [, < function_name >(input_data_type),...]    
       | ALL GLOBAL FUNCTIONS }
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

例: `GRANT usage ON GLOBAL FUNCTION a(string) to kevin;`

#### 内部カタログ

```SQL
GRANT
    { USAGE | CREATE DATABASE | ALL [PRIVILEGES]} 
    ON CATALOG default_catalog
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

#### 外部カタログ

```SQL
GRANT
   { USAGE | DROP | ALL [PRIVILEGES] } 
   ON { CATALOG <catalog_name> [, <catalog_name>,...] | ALL CATALOGS}
   TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

#### データベース

```SQL
GRANT
    { ALTER | DROP | CREATE TABLE | CREATE VIEW | CREATE FUNCTION | CREATE MATERIALIZED VIEW | ALL [PRIVILEGES] } 
    ON { DATABASE <database_name> [, <database_name>,...] | ALL DATABASES }
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

* このコマンドを実行する前に、SET CATALOGを実行する必要があります。
* 外部カタログのデータベースでは、Hive（v3.1以降）およびIceberg（v3.2以降）データベースのCREATE TABLE権限のみを付与できます。

#### テーブル

```SQL
GRANT
    { ALTER | DROP | SELECT | INSERT | EXPORT | UPDATE | DELETE | ALL [PRIVILEGES]} 
    ON { TABLE <table_name> [, < table_name >,...]
       | ALL TABLES} IN 
           { { DATABASE <database_name> [, <database_name>,...] } | ALL DATABASES }
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

* このコマンドを実行する前に、SET CATALOGを実行する必要があります。
* `<db_name>.<table_name>`を使用してテーブルを表すこともできます。
* 内部および外部カタログのすべてのテーブルに対してSELECT権限を付与してデータを読み取ることができます。HiveおよびIcebergカタログのテーブルでは、INSERT権限を付与してデータを書き込むことができます（Icebergはv3.1以降、Hiveはv3.2以降でサポート）。

  ```SQL
  GRANT <priv> ON TABLE <db_name>.<table_name> TO {ROLE <role_name> | USER <user_name>}
  ```

#### ビュー

```SQL
GRANT  
    { ALTER | DROP | SELECT | ALL [PRIVILEGES]} 
    ON { VIEW <view_name> [, < view_name >,...]
       ｜ ALL VIEWS} IN 
           { { DATABASE <database_name> [, <database_name>,...] } | ALL DATABASES }
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

* このコマンドを実行する前に、SET CATALOGを実行する必要があります。
* `<db_name>.<view_name>`を使用してビューを表すこともできます。
* 外部カタログのテーブルでは、Hiveテーブルビューに対してのみSELECT権限を付与できます（v3.1以降）。

  ```SQL
  GRANT <priv> ON VIEW <db_name>.<view_name> TO {ROLE <role_name> | USER <user_name>}
  ```

#### マテリアライズドビュー

```SQL
GRANT
    { SELECT | ALTER | REFRESH | DROP | ALL [PRIVILEGES]} 
    ON { MATERIALIZED VIEW <mv_name> [, < mv_name >,...]
       ｜ ALL MATERIALIZED VIEWS} IN 
           { { DATABASE <database_name> [, <database_name>,...] } | ALL DATABASES }
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

* このコマンドを実行する前に、SET CATALOGを実行する必要があります。
* `<db_name>.<mv_name>`を使用してmvを表すこともできます。

  ```SQL
  GRANT <priv> ON MATERIALIZED VIEW <db_name>.<mv_name> TO {ROLE <role_name> | USER <user_name>}
  ```

#### 関数

```SQL
GRANT
    { USAGE | DROP | ALL [PRIVILEGES]} 
    ON { FUNCTION <function_name>(input_data_type) [, < function_name >(input_data_type),...]
       ｜ ALL FUNCTIONS} IN 
           { { DATABASE <database_name> [, <database_name>,...] } | ALL DATABASES }
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

* このコマンドを実行する前に、SET CATALOGを実行する必要があります。
* `<db_name>.<function_name>`を使用して関数を表すこともできます。

  ```SQL
  GRANT <priv> ON FUNCTION <db_name>.<function_name> TO {ROLE <role_name> | USER <user_name>}
  ```

#### ユーザー

```SQL
GRANT IMPERSONATE
ON USER <user_identity>
TO USER <user_identity_1> [ WITH GRANT OPTION ]
```

#### ストレージボリューム

```SQL
GRANT  
    { USAGE | ALTER | DROP | ALL [PRIVILEGES] } 
    ON { STORAGE VOLUME < name > [, < name >,...] ｜ ALL STORAGE VOLUMES} 
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

### ロールをロールまたはユーザーに付与する

```SQL
GRANT <role_name> [,<role_name>, ...] TO ROLE <role_name>
GRANT <role_name> [,<role_name>, ...] TO USER <user_identity>
```

## 例

例1: 全てのデータベースの全てのテーブルからデータを読み取る権限をユーザー`jack`に付与します。

```SQL
GRANT SELECT ON *.* TO 'jack'@'%';
```

例2: データベース`db1`の全てのテーブルにデータをロードする権限をロール`my_role`に付与します。

```SQL
GRANT INSERT ON db1.* TO ROLE 'my_role';
```

例3: データベース`db1`のテーブル`tbl1`に対してデータの読み取り、更新、ロードの権限をユーザー`jack`に付与します。

```SQL
GRANT SELECT,ALTER,INSERT ON db1.tbl1 TO 'jack'@'192.8.%';
```

例4: 全てのリソースを使用する権限をユーザー`jack`に付与します。

```SQL
GRANT USAGE ON RESOURCE * TO 'jack'@'%';
```

例5: ユーザー`jack`にリソース`spark_resource`の使用権限を付与します。

```SQL
GRANT USAGE ON RESOURCE 'spark_resource' TO 'jack'@'%';
```

例6: ロール`my_role`にリソース`spark_resource`の使用権限を付与します。

```SQL
GRANT USAGE ON RESOURCE 'spark_resource' TO ROLE 'my_role';
```

例7: テーブル`sr_member`からデータを読み取る権限をユーザー`jack`に付与し、ユーザー`jack`に他のユーザーやロールにこの権限を付与することを許可します（WITH GRANT OPTIONを指定）。

```SQL
GRANT SELECT ON TABLE sr_member TO USER jack@'172.10.1.10' WITH GRANT OPTION;
```

例8: システム定義のロール`db_admin`、`user_admin`、`cluster_admin`をユーザー`user_platform`に付与します。

```SQL
GRANT db_admin, user_admin, cluster_admin TO USER user_platform;
```

例9: ユーザー`jack`がユーザー`rose`として操作を実行できるようにします。

```SQL
GRANT IMPERSONATE ON 'rose'@'%' TO 'jack'@'%';
```

## ベストプラクティス

### シナリオに基づいてロールをカスタマイズする

<UserPrivilegeCase />

マルチサービスアクセス制御のベストプラクティスについては、[マルチサービスアクセス制御](../../../administration/User_privilege.md#multi-service-access-control)を参照してください。
