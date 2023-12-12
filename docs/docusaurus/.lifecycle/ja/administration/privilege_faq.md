---
displayed_sidebar: "Japanese"
---

# 特権FAQ

## ユーザーに必要なロールが割り当てられたにもかかわらず、エラーメッセージ「権限がありません」が報告されるのはなぜですか？

このエラーは、ロールがアクティブ化されていない場合に発生することがあります。`select current_role();`を実行して、現在のセッションでユーザーにアクティブ化されたロールをクエリできます。必要なロールがアクティブ化されていない場合は、[SET ROLE](../sql-reference/sql-statements/account-management/SET_ROLE.md)を実行して、このロールをアクティブ化し、このロールを使用して操作を実行します。

ログイン時にロールが自動的にアクティブ化されるようにする場合は、`user_admin`ロールで[SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md)または[ALTER USER DEFAULT ROLE](../sql-reference/sql-statements/account-management/ALTER_USER.md)を実行して、各ユーザーのデフォルトロールを設定できます。デフォルトのロールが設定されてからは、ユーザーがログインすると自動的にアクティブ化されます。

すべてのユーザーの割り当てられたロールがログイン時に自動的にアクティブ化されるようにする場合は、次のコマンドを実行できます。この操作は、システムレベルでのOPERATE権限が必要です。

```SQL
SET GLOBAL activate_all_roles_on_login = TRUE;
```

ただし、潜在的なリスクを防ぐためにデフォルトのロールを制限された特権で設定することによって、「最小特権の原則」に従うことをお勧めします。例：

- 一般ユーザーに対しては、SELECT権限のみを持つ`read_only`ロールをデフォルトロールとして設定し、ALTER、DROP、INSERTなどの特権を持つロールをデフォルトロールとして設定しないでください。
- 管理者に対しては、`db_admin`ロールをデフォルトロールとして設定し、ノードを追加および削除する特権を持つ`node_admin`ロールをデフォルトロールとして設定しないでください。

この方法により、適切な権限を持つロールがユーザーに割り当てられ、意図しない操作のリスクが低減されます。

[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)を実行して、必要な特権やロールをユーザーに割り当てることができます。

## ユーザーにデータベース内のすべてのテーブルに特権を付与しました（`GRANT ALL ON ALL TABLES IN DATABASE <db_name> TO USER <user_identity>;`），しかし、ユーザーはまだデータベース内でテーブルを作成できません。なぜでしょうか？

データベース内でテーブルを作成するには、データベースレベルのCREATE TABLE特権が必要です。ユーザーに特権を付与する必要があります。

```SQL
GRANT CREATE TABLE ON DATABASE <db_name> TO USER <user_identity>;;
```

## `GRANT ALL ON DATABASE <db_name> TO USER <user_identity>;`を使用してデータベースに対してユーザーにすべての特権を与えましたが、ユーザーがこのデータベースで`SHOW TABLES;`を実行しても何も返されません。なぜでしょうか？

`SHOW TABLES;`は、ユーザーが特権を持つテーブルのみを返します。ユーザーがテーブルに特権を持っていない場合、そのテーブルは返されません。このデータベース内のすべてのテーブルに対して任意の特権（たとえばSELECT）をユーザーに付与できます。

```SQL
GRANT SELECT ON ALL TABLES IN DATABASE <db_name> TO USER <user_identity>;
```

上記のステートメントは、v3.0より前のバージョンで使用されていた`GRANT select_priv ON db.* TO <user_identity>;`と同等です。