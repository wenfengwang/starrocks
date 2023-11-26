---
displayed_sidebar: "Japanese"
---

# 特権のFAQ

## 必要なロールがユーザーに割り当てられた後でも、「権限がありません」というエラーメッセージが表示されるのはなぜですか？

このエラーは、ロールがアクティブ化されていない場合に発生する可能性があります。`select current_role();` を実行して、現在のセッションでユーザーにアクティブ化されているロールをクエリすることができます。必要なロールがアクティブ化されていない場合は、[SET ROLE](../sql-reference/sql-statements/account-management/SET_ROLE.md) を実行してこのロールをアクティブ化し、このロールを使用して操作を行います。

ログイン時にロールが自動的にアクティブ化されるようにする場合、`user_admin` ロールは [SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md) または [ALTER USER DEFAULT ROLE](../sql-reference/sql-statements/account-management/ALTER_USER.md) を実行して、各ユーザーにデフォルトのロールを設定することができます。デフォルトのロールが設定されると、ユーザーがログインすると自動的にアクティブ化されます。

すべてのユーザーの割り当てられたロールがログイン時に自動的にアクティブ化されるようにするには、次のコマンドを実行します。この操作には、システムレベルでの OPERATE 権限が必要です。

```SQL
SET GLOBAL activate_all_roles_on_login = TRUE;
```

ただし、潜在的なリスクを防ぐために、デフォルトのロールには制限された権限を持つロールを設定することで、「最小特権」の原則に従うことをお勧めします。例えば：

- 一般ユーザーの場合、デフォルトのロールとして SELECT 権限のみを持つ `read_only` ロールを設定し、ALTER、DROP、INSERT などの権限を持つロールをデフォルトのロールとして設定しないようにします。
- 管理者の場合、デフォルトのロールとして `db_admin` ロールを設定し、ノードの追加や削除の権限を持つ `node_admin` ロールをデフォルトのロールとして設定しないようにします。

この方法により、ユーザーに適切な権限を持つロールが割り当てられ、意図しない操作のリスクが低減されます。

必要な権限やロールをユーザーに割り当てるには、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md) を実行します。

## データベース内のすべてのテーブルに対してユーザーに特権を付与しました (`GRANT ALL ON ALL TABLES IN DATABASE <db_name> TO USER <user_identity>;`) が、ユーザーはまだデータベース内でテーブルを作成できません。なぜですか？

データベース内でテーブルを作成するには、データベースレベルの CREATE TABLE 特権が必要です。この特権をユーザーに付与する必要があります。

```SQL
GRANT CREATE TABLE ON DATABASE <db_name> TO USER <user_identity>;;
```

## `GRANT ALL ON DATABASE <db_name> TO USER <user_identity>;` を使用してユーザーにデータベースのすべての特権を付与しましたが、ユーザーがこのデータベースで `SHOW TABLES;` を実行しても何も返されません。なぜですか？

`SHOW TABLES;` は、ユーザーが特権を持つテーブルのみを返します。ユーザーがテーブルに特権を持たない場合、そのテーブルは返されません。このデータベースのすべてのテーブルに対して任意の特権（たとえば SELECT）をユーザーに付与することができます：

```SQL
GRANT SELECT ON ALL TABLES IN DATABASE <db_name> TO USER <user_identity>;
```

上記の文は、v3.0 より前のバージョンで使用される `GRANT select_priv ON db.* TO <user_identity>;` と同等です。
