---
displayed_sidebar: "Japanese"
---

# 特権FAQ

## ユーザーに必要なロールが割り当てられたにもかかわらず、エラーメッセージ "権限なし" が報告される理由は何ですか？

このエラーは、ロールがアクティブ化されていない場合に発生する可能性があります。ユーザーの現在のセッションでアクティブ化されたロールをクエリするには `SELECT current_role();` を実行できます。必要なロールがアクティブ化されていない場合は、[SET ROLE](../sql-reference/sql-statements/account-management/SET_ROLE.md) を実行してこのロールをアクティブ化し、このロールを使用して操作を行うことができます。

ログイン時にロールを自動的にアクティブ化したい場合は、`user_admin` ロールが [SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md) または [ALTER USER DEFAULT ROLE](../sql-reference/sql-statements/account-management/ALTER_USER.md) を実行して、各ユーザーにデフォルトのロールを設定できます。デフォルトのロールが設定されると、ユーザーがログインすると自動的にアクティブ化されます。

すべてのユーザーの割り当てられたロールがログイン時に自動的にアクティブ化されるようにしたい場合は、次のコマンドを実行できます。この操作には、システムレベルでのOPERATE権限が必要です。

```SQL
SET GLOBAL activate_all_roles_on_login = TRUE;
```

ただし、潜在的なリスクを防ぐために、デフォルトのロールを制限された特権で設定することで、「最小特権の原則」に従うことを推奨します。例：

- 一般ユーザーの場合、SELECT特権のみを持つ `read_only` ロールをデフォルトのロールとして設定し、ALTER、DROP、INSERTなどの特権を持つロールをデフォルトのロールとして設定しないようにします。
- 管理者の場合、`db_admin` ロールをデフォルトのロールとして設定し、ノードの追加や削除などの特権を持つ `node_admin` ロールをデフォルトのロールとして設定しないようにします。

このアプローチにより、適切な権限を持つロールがユーザーに割り当てられ、意図しない操作のリスクが低減されることが保証されます。

[GRANT](../sql-reference/sql-statements/account-management/GRANT.md) を実行して、必要な特権やロールをユーザーに割り当てることができます。

## データベース内のすべてのテーブルに対する特権をユーザーに付与しました (`GRANT ALL ON ALL TABLES IN DATABASE <db_name> TO USER <user_identity>;`) が、ユーザーはデータベース内でテーブルを作成できません。なぜですか？

データベース内でテーブルを作成するには、データベースレベルのCREATE TABLE特権が必要です。この特権をユーザーに付与する必要があります。

```SQL
GRANT CREATE TABLE ON DATABASE <db_name> TO USER <user_identity>;;
```

## `GRANT ALL ON DATABASE <db_name> TO USER <user_identity>;` を使用して、ユーザーにデータベース全体のすべての特権を付与しましたが、ユーザーがこのデータベースで `SHOW TABLES;` を実行しても何も返されません。なぜですか？

`SHOW TABLES;` は、ユーザーが任意の特権を持つテーブルのみを返します。ユーザーがテーブルに特権を持たない場合、そのテーブルは返されません。このデータベース内のすべてのテーブルに任意の特権 (例: SELECT) をユーザーに付与することができます。

```SQL
GRANT SELECT ON ALL TABLES IN DATABASE <db_name> TO USER <user_identity>;
```

上記のステートメントは、v3.0より前のバージョンで使用されていた `<user_identity>;` を使用した `db.* TO <user_identity>;` に相当します。