---
displayed_sidebar: English
---

# GRANT

import UserPrivilegeCase from '../../../assets/commonMarkdown/userPrivilegeCase.md'

## 説明

特定のオブジェクトに対する1つ以上の権限をユーザーまたはロールに付与します。

ユーザーまたは他のロールにロールを付与します。

付与できる権限の詳細については、[権限項目](../../../administration/privilege_item.md)を参照してください。

GRANT操作を実行した後、[SHOW GRANTS](./SHOW_GRANTS.md)を実行して詳細な権限情報を表示するか、[REVOKE](REVOKE.md)を実行して権限またはロールを取り消すことができます。

GRANT操作を実行する前に、関連するユーザーまたはロールが作成されていることを確認してください。詳細については、[CREATE USER](./CREATE_USER.md)および[CREATE ROLE](./CREATE_ROLE.md)を参照してください。

> **注記**
>
> `user_admin`ロールを持つユーザーのみが他のユーザーやロールに任意の権限を付与できます。
> 他のユーザーは、WITH GRANT OPTIONキーワードを使用して、他のユーザーやロールに権限を付与することのみができます。

## 構文

### ロールまたはユーザーに権限を付与する

#### システム

```SQL
GRANT
    { CREATE RESOURCE GROUP | CREATE RESOURCE | CREATE EXTERNAL CATALOG | REPOSITORY | BLACKLIST | FILE | OPERATE | CREATE STORAGE VOLUME } 
    ON SYSTEM
    TO { ROLE | USER } {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

#### リソースグループ

```SQL
GRANT
    { ALTER | DROP | ALL [PRIVILEGES] } 
    ON { RESOURCE GROUP <resource_group_name> [, <resource_group_name>,...] | ALL RESOURCE GROUPS } 
    TO { ROLE | USER } {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

#### リソース

```SQL
GRANT
    { USAGE | ALTER | DROP | ALL [PRIVILEGES] } 
    ON { RESOURCE <resource_name> [, <resource_name>,...] | ALL RESOURCES } 
    TO { ROLE | USER } {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

#### グローバルUDF

```SQL
GRANT
    { USAGE | DROP | ALL [PRIVILEGES] } 
    ON { GLOBAL FUNCTION <function_name>(input_data_type) [, <function_name>(input_data_type),...]    
       | ALL GLOBAL FUNCTIONS }
    TO { ROLE | USER } {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

例: `GRANT USAGE ON GLOBAL FUNCTION a(string) TO kevin;`

#### 内部カタログ

```SQL
GRANT
    { USAGE | CREATE DATABASE | ALL [PRIVILEGES] } 
    ON CATALOG default_catalog
    TO { ROLE | USER } {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

#### 外部カタログ

```SQL
GRANT
   { USAGE | DROP | ALL [PRIVILEGES] } 
   ON { CATALOG <catalog_name> [, <catalog_name>,...] | ALL CATALOGS }
   TO { ROLE | USER } {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

#### データベース

```SQL
GRANT
    { ALTER | DROP | CREATE TABLE | CREATE VIEW | CREATE FUNCTION | CREATE MATERIALIZED VIEW | ALL [PRIVILEGES] } 
    ON { DATABASE <database_name> [, <database_name>,...] | ALL DATABASES }
    TO { ROLE | USER } {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

* このコマンドを実行する前に、まずSET CATALOGを実行する必要があります。
* 外部カタログ内のデータベースでは、CREATE TABLE権限はHive（v3.1以降）とIcebergデータベース（v3.2以降）にのみ付与できます。

#### テーブル

```SQL
GRANT
    { ALTER | DROP | SELECT | INSERT | EXPORT | UPDATE | DELETE | ALL [PRIVILEGES] } 
    ON { TABLE <table_name> [, <table_name>,...]
       | ALL TABLES } IN 
           { { DATABASE <database_name> [, <database_name>,...] } | ALL DATABASES }
    TO { ROLE | USER } {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

* このコマンドを実行する前に、まずSET CATALOGを実行する必要があります。
* `<db_name>.<table_name>`を使用してテーブルを表すこともできます。
* 内部カタログと外部カタログのすべてのテーブルにSELECT権限を付与して、これらのテーブルからデータを読み取ることができます。HiveとIcebergカタログのテーブルでは、INSERT権限を付与してそのようなテーブルにデータを書き込むことができます（Icebergはv3.1以降、Hiveはv3.2以降でサポートされています）

  ```SQL
  GRANT <priv> ON TABLE <db_name>.<table_name> TO {ROLE <role_name> | USER <user_name>}
  ```

#### ビュー

```SQL
GRANT  
    { ALTER | DROP | SELECT | ALL [PRIVILEGES] } 
    ON { VIEW <view_name> [, <view_name>,...]
       | ALL VIEWS } IN 
           { { DATABASE <database_name> [, <database_name>,...] } | ALL DATABASES }
    TO { ROLE | USER } {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

* このコマンドを実行する前に、まずSET CATALOGを実行する必要があります。
* `<db_name>.<view_name>`を使用してビューを表すこともできます。
* 外部カタログのテーブルでは、HiveテーブルビューにSELECT権限のみを付与できます（v3.1以降）。

  ```SQL
  GRANT <priv> ON VIEW <db_name>.<view_name> TO {ROLE <role_name> | USER <user_name>}
  ```

#### マテリアライズドビュー

```SQL
GRANT
    { SELECT | ALTER | REFRESH | DROP | ALL [PRIVILEGES] } 
    ON { MATERIALIZED VIEW <mv_name> [, <mv_name>,...]
       | ALL MATERIALIZED VIEWS } IN 
           { { DATABASE <database_name> [, <database_name>,...] } | ALL DATABASES }
    TO { ROLE | USER } {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

* このコマンドを実行する前に、まずSET CATALOGを実行する必要があります。
* `<db_name>.<mv_name>`を使用してマテリアライズドビューを表すこともできます。

  ```SQL
  GRANT <priv> ON MATERIALIZED VIEW <db_name>.<mv_name> TO {ROLE <role_name> | USER <user_name>}
  ```

#### 関数

```SQL
GRANT
    { USAGE | DROP | ALL [PRIVILEGES] } 
    ON { FUNCTION <function_name>(input_data_type) [, <function_name>(input_data_type),...]
       | ALL FUNCTIONS } IN 
           { { DATABASE <database_name> [, <database_name>,...] } | ALL DATABASES }
    TO { ROLE | USER } {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

* このコマンドを実行する前に、まずSET CATALOGを実行する必要があります。
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
    ON { STORAGE VOLUME <name> [, <name>,...] | ALL STORAGE VOLUMES } 
    TO { ROLE | USER } {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

### ロールをロールまたはユーザーに付与する

```SQL
GRANT <role_name> [, <role_name>, ...] TO ROLE <role_name>
GRANT <role_name> [, <role_name>, ...] TO USER <user_identity>
```

## 例

例1: すべてのデータベースのすべてのテーブルからデータを読み取る権限をユーザー`jack`に付与します。

```SQL
GRANT SELECT ON *.* TO 'jack'@'%';
```

例2: データベース`db1`のすべてのテーブルにデータをロードする権限をロール`my_role`に付与します。

```SQL
GRANT INSERT ON db1.* TO ROLE 'my_role';
```

例3: データベース`db1`のテーブル`tbl1`にデータを読み取り、更新、およびロードする権限をユーザー`jack`に付与します。

```SQL
GRANT SELECT, ALTER, INSERT ON db1.tbl1 TO 'jack'@'192.8.%';
```

例4: すべてのリソースを使用する権限をユーザー`jack`に付与します。

```SQL
GRANT USAGE ON RESOURCE * TO 'jack'@'%';
```

例5: リソース`spark_resource`を使用する権限をユーザー`jack`に付与します。

```SQL
GRANT USAGE ON RESOURCE 'spark_resource' TO 'jack'@'%';
```

例6: リソース`spark_resource`を使用する権限をロール`my_role`に付与します。

```SQL
GRANT USAGE ON RESOURCE 'spark_resource' TO ROLE 'my_role';
```

例7: テーブル`sr_member`からデータを読み取る権限をユーザー`jack`に付与し、WITH GRANT OPTIONを指定することでユーザー`jack`がこの権限を他のユーザーやロールに付与できるようにします。

```SQL
GRANT SELECT ON TABLE sr_member TO USER 'jack'@'172.10.1.10' WITH GRANT OPTION;
```

例8: システム定義のロール`db_admin`、`user_admin`、および`cluster_admin`をユーザー`user_platform`に付与します。

```SQL
GRANT db_admin, user_admin, cluster_admin TO USER 'user_platform';
```

例9: ユーザー`jack`に、ユーザー`rose`として操作を実行する権限を付与します。

```SQL
GRANT IMPERSONATE ON USER 'rose' TO USER 'jack'@'%';
```

## ベストプラクティス

### シナリオに基づいてロールをカスタマイズする

<UserPrivilegeCase />

マルチサービスアクセス制御のベストプラクティスについては、[マルチサービスアクセス制御](../../../administration/User_privilege.md#multi-service-access-control)を参照してください。
