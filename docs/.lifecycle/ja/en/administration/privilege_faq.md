---
displayed_sidebar: English
---

# 特権に関するFAQ

## 必要なロールがユーザーに割り当てられた後でも、「権限がありません」というエラーメッセージが報告されるのはなぜですか?

このエラーは、ロールがアクティブ化されていない場合に発生することがあります。`select current_role();` を実行して、現在のセッションでユーザーにアクティブ化されているロールを照会できます。必要なロールがアクティブ化されていない場合は、[SET ROLE](../sql-reference/sql-statements/account-management/SET_ROLE.md) を実行してこのロールをアクティブ化し、このロールを使用して操作を実行してください。

ログイン時にロールを自動的にアクティブ化するには、`user_admin` ロールが [SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md) または [ALTER USER DEFAULT ROLE](../sql-reference/sql-statements/account-management/ALTER_USER.md) を実行して、各ユーザーにデフォルトロールを設定できます。デフォルトロールが設定されると、ユーザーがログインする際に自動的にアクティブ化されます。

すべてのユーザーに割り当てられたすべてのロールをログイン時に自動的にアクティブ化するには、以下のコマンドを実行します。この操作には、システムレベルでのOPERATE権限が必要です。

```SQL
SET GLOBAL activate_all_roles_on_login = TRUE;
```

しかし、潜在的なリスクを防ぐために、限定された権限を持つデフォルトロールを設定することで、「最小限の特権」の原則に従うことを推奨します。例えば：

- 一般ユーザーには、SELECT権限のみを持つ `read_only` ロールをデフォルトロールとして設定し、ALTER、DROP、INSERTなどの権限を持つロールをデフォルトロールとして設定することを避けます。
- 管理者には、`db_admin` ロールをデフォルトロールとして設定し、ノードの追加や削除の権限を持つ `node_admin` ロールをデフォルトロールとして設定することを避けます。

このアプローチは、ユーザーに適切な権限を持つロールが割り当てられ、意図しない操作のリスクを軽減するのに役立ちます。

[GRANT](../sql-reference/sql-statements/account-management/GRANT.md) を実行して、必要な権限またはロールをユーザーに割り当てることができます。

## データベース内のすべてのテーブルに対する権限をユーザーに付与しました (`GRANT ALL ON ALL TABLES IN DATABASE <db_name> TO USER <user_identity>;`) が、ユーザーはデータベースにテーブルを作成できません。なぜでしょうか。

データベース内にテーブルを作成するには、データベースレベルのCREATE TABLE権限が必要です。ユーザーにその権限を付与する必要があります。

```SQL
GRANT CREATE TABLE ON DATABASE <db_name> TO USER <user_identity>;
```

## `GRANT ALL ON DATABASE <db_name> TO USER <user_identity>;` を使用してデータベースに対するすべての権限をユーザーに付与しましたが、ユーザーがこのデータベースで `SHOW TABLES;` を実行しても何も返されません。なぜでしょうか。

`SHOW TABLES;` は、ユーザーが権限を持っているテーブルのみを返します。ユーザーがテーブルに対する権限を持っていない場合、そのテーブルは返されません。このデータベース内のすべてのテーブルに対する任意の権限（たとえばSELECT）をユーザーに付与できます。

```SQL
GRANT SELECT ON ALL TABLES IN DATABASE <db_name> TO USER <user_identity>;
```

上記のステートメントは、v3.0より前のバージョンで使用されていた `GRANT select_priv ON db.* TO <user_identity>;` と同等です。
