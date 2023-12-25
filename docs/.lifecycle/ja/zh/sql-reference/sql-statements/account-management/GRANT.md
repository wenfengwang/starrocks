---
displayed_sidebar: Chinese
---

# GRANT

import UserPrivilegeCase from '../../../assets/commonMarkdown/userPrivilegeCase.md'

## 機能

このステートメントは、一つまたは複数の権限をロールまたはユーザーに付与するため、またはロールをユーザーや他のロールに付与するために使用されます。

権限項目の詳細については、[権限項目](../../../administration/privilege_item.md)を参照してください。

権限付与後、[SHOW GRANTS](SHOW_GRANTS.md)を通じて権限付与情報を確認することができます。また、[REVOKE](REVOKE.md)を通じて権限やロールを取り消すことができます。

GRANT操作を実行する前に、システム内にユーザーまたはロールが作成されていることを確認してください。作成に関する詳細は、[CREATE USER](./CREATE_USER.md)と[CREATE ROLE](./CREATE_ROLE.md)を参照してください。

> **注意**
>
> - `user_admin`ロールを持つユーザーのみが、任意の権限を任意のユーザーやロールに付与することができます。
> - ロールがユーザーに付与された後、ユーザーは[SET ROLE](SET_ROLE.md)を通じてロールを手動でアクティブにする必要があります。これにより、ロールの権限を利用できるようになります。ユーザーがログイン時にデフォルトでロールをアクティブにすることを望む場合は、[ALTER USER](ALTER_USER.md)または[SET DEFAULT ROLE](SET_DEFAULT_ROLE.md)を通じてユーザーのデフォルトロールを設定できます。システム内の全ユーザーがログイン時にすべての権限をデフォルトでアクティブにすることを望む場合は、グローバル変数`SET GLOBAL activate_all_roles_on_login = TRUE;`を設定できます。
> - 一般ユーザーは、WITH GRANT OPTIONキーワードを含む自身が持つ権限を他のユーザーやロールに付与することができます。例7を参照してください。

## 文法

### ユーザーまたはロールに権限を付与する

#### System関連

```SQL
GRANT
    { CREATE RESOURCE GROUP | CREATE RESOURCE | CREATE EXTERNAL CATALOG | REPOSITORY | BLACKLIST | FILE | OPERATE | CREATE STORAGE VOLUME } 
    ON SYSTEM
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

#### Resource group関連

```SQL
GRANT
    { ALTER | DROP | ALL [PRIVILEGES] } 
    ON { RESOURCE GROUP <resource_group_name> [, <resource_group_name>,...] | ALL RESOURCE GROUPS} 
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

#### Resource関連

```SQL
GRANT
    { USAGE | ALTER | DROP | ALL [PRIVILEGES] } 
    ON { RESOURCE <resource_name> [, <resource_name>,...] | ALL RESOURCES} 
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

#### Global UDF関連

```SQL
GRANT
    { USAGE | DROP | ALL [PRIVILEGES]} 
    ON { GLOBAL FUNCTION <function_name>(input_data_type) [, <function_name>(input_data_type),...]    
       | ALL GLOBAL FUNCTIONS }
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

例：`GRANT USAGE ON GLOBAL FUNCTION a(string) TO kevin;`

#### Internal catalog関連

```SQL
GRANT
    { USAGE | CREATE DATABASE | ALL [PRIVILEGES]} 
    ON CATALOG default_catalog
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

#### External catalog関連

```SQL
GRANT
   { USAGE | DROP | ALL [PRIVILEGES] } 
   ON { CATALOG <catalog_name> [, <catalog_name>,...] | ALL CATALOGS}
   TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

#### Database関連

```SQL
GRANT
    { ALTER | DROP | CREATE TABLE | CREATE VIEW | CREATE FUNCTION | CREATE MATERIALIZED VIEW | ALL [PRIVILEGES] } 
    ON { DATABASE <database_name> [, <database_name>,...] | ALL DATABASES }
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

**注意**
>
> 1. SET CATALOGを実行した後でなければ使用できません。
> 2. External Catalog下のデータベースについては、HiveとIcebergデータベースのみがCREATE TABLE権限を付与することができます。

#### Table関連

```SQL
GRANT  
    { ALTER | DROP | SELECT | INSERT | EXPORT | UPDATE | DELETE | ALL [PRIVILEGES]} 
    ON { TABLE <table_name> [, <table_name>,...]
       | ALL TABLES IN 
           { { DATABASE <database_name> [,<database_name>,...] } | ALL DATABASES }}
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

> **注意**
>
> 1. SET CATALOGを実行した後でなければ使用できません。Tableは`<db_name>.<table_name>`の形式で表現することもできます。
> 2. すべてのInternal CatalogとExternal Catalog下のテーブルはSELECT権限を付与することができます。HiveとIceberg catalog下のテーブルは、INSERT権限も付与することができます（バージョン3.1からIcebergテーブルへのINSERT権限の付与をサポートし、バージョン3.2からHiveテーブルへのINSERT権限の付与をサポートします）。

```SQL
GRANT <priv> ON TABLE <db_name>.<table_name> TO {ROLE <role_name> | USER <user_name>}
```

#### View関連

```SQL
GRANT  
    { ALTER | DROP | SELECT | ALL [PRIVILEGES]} 
    ON { VIEW <view_name> [, <view_name>,...]
       | ALL VIEWS IN 
           { { DATABASE <database_name> [,<database_name>,...] }| ALL DATABASES }}
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

> **注意**
>
> 1. SET CATALOGを実行した後でなければ使用できません。Viewは`<db_name>.<view_name>`の形式で表現することもできます。
> 2. External Catalogについては、HiveのテーブルビューのみがSELECT権限をサポートしています（バージョン3.1以降）。

```SQL
GRANT <priv> ON VIEW <db_name>.<view_name> TO {ROLE <role_name> | USER <user_name>}
```

#### Materialized view関連

```SQL
GRANT
    { SELECT | ALTER | REFRESH | DROP | ALL [PRIVILEGES]} 
    ON { MATERIALIZED VIEW <mv_name> [, <mv_name>,...]
       | ALL MATERIALIZED VIEWS IN 
           { { DATABASE <database_name> [,<database_name>,...] }| ALL DATABASES }}
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

*注意：SET CATALOGを実行した後でなければ使用できません。マテリアライズドビューは`<db_name>.<mv_name>`の形式で表現することもできます。

```SQL
GRANT <priv> ON MATERIALIZED VIEW <db_name>.<mv_name> TO {ROLE <role_name> | USER <user_name>};
```

#### Function関連

```SQL
GRANT
    { USAGE | DROP | ALL [PRIVILEGES]} 
    ON { FUNCTION <function_name>(input_data_type) [, <function_name>(input_data_type),...]
       | ALL FUNCTIONS IN 
           { { DATABASE <database_name> [,<database_name>,...] }| ALL DATABASES }}
    TO { ROLE | USER } {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

*注意：SET CATALOGを実行した後でなければ使用できません。Functionは`<db_name>.<function_name>`の形式で表現することもできます。

```SQL
GRANT <priv> ON FUNCTION <db_name>.<function_name> TO {ROLE <role_name> | USER <user_name>}
```

#### User関連

```SQL
GRANT IMPERSONATE ON USER <user_identity> TO USER <user_identity_1> [ WITH GRANT OPTION ]
```

#### Storage volume関連

```SQL
GRANT
    { USAGE | ALTER | DROP | ALL [PRIVILEGES] } 
    ON { STORAGE VOLUME <name> [, <name>,...] | ALL STORAGE VOLUMES} 
    TO { ROLE | USER} {<role_name>|<user_identity>} [ WITH GRANT OPTION ]
```

### ユーザーまたは他のロールにロールを付与する

```SQL
GRANT <role_name> [,<role_name>, ...] TO ROLE <role_name>
GRANT <role_name> [,<role_name>, ...] TO USER <user_identity>
```

**注意：**

- ロールがユーザーに付与された後、ユーザーは[SET ROLE](SET_ROLE.md)を通じてロールを手動でアクティブにする必要があります。
- ユーザーがログイン時にデフォルトでロールをアクティブにすることを望む場合は、[ALTER USER](ALTER_USER.md)または[SET DEFAULT ROLE](SET_DEFAULT_ROLE.md)を通じてユーザーのデフォルトロールを設定できます。
- システム内の全ユーザーがログイン時にすべての権限をデフォルトでアクティブにすることを望む場合は、グローバル変数`SET GLOBAL activate_all_roles_on_login = TRUE;`を設定できます。

## 例

例1：すべてのデータベースおよびその中のすべてのテーブルの読み取り権限をユーザー`jack`に付与します。

```SQL
GRANT SELECT ON *.* TO 'jack'@'%';
```

例2：データベース`db1`およびその中のすべてのテーブルのインサート権限をロール`my_role`に付与します。

```SQL
GRANT INSERT ON db1.* TO ROLE 'my_role';
```

例3：データベース`db1`とテーブル`tbl1`の読み取り、構造変更、インサート権限をユーザー`jack`に付与します。

```SQL
GRANT SELECT,ALTER,INSERT ON db1.tbl1 TO 'jack'@'192.8.%';
```

例4：すべてのリソースの使用権限をユーザー`jack`に付与します。

```SQL
GRANT USAGE ON RESOURCE * TO 'jack'@'%';
```

例5：リソース`spark_resource`の使用権限をユーザー`jack`に付与します。

```SQL
GRANT USAGE ON RESOURCE 'spark_resource' TO 'jack'@'%';
```

例6：リソース`spark_resource`の使用権限をロール`my_role`に付与します。

```SQL
GRANT USAGE ON RESOURCE 'spark_resource' TO ROLE 'my_role';
```

例7：テーブル`sr_member`のSELECT権限をユーザー`jack`に付与し、`jack`がこの権限を他のユーザーやロールに付与できるようにします（SQLでWITH GRANT OPTIONを指定することにより）。

```SQL
GRANT SELECT ON TABLE sr_member TO USER jack@'172.10.1.10' WITH GRANT OPTION;
```

例8：システムにプリセットされたロール`db_admin`、`user_admin`、`cluster_admin`をプラットフォーム運用ロールに付与します。

```SQL
GRANT db_admin, user_admin, cluster_admin TO USER user_platform;
```

例9：ユーザー`jack`に、ユーザー`rose`の身分で操作を実行する権限を付与します。

```SQL
GRANT IMPERSONATE ON 'rose'@'%' TO 'jack'@'%';
```

## ベストプラクティス - 使用シナリオに基づいてカスタムロールを作成する

<UserPrivilegeCase />

複数のビジネスラインの権限管理に関する実践については、[複数ビジネスラインの権限管理](../../../administration/User_privilege.md#複数ビジネスラインの権限管理)を参照してください。
