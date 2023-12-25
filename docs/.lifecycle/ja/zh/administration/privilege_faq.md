---
displayed_sidebar: Chinese
---

# 権限システム FAQ

## ユーザーにロールを割り当てたにも関わらず、SQLを実行すると「権限がない」というエラーが出るのはなぜですか？

ユーザーがそのロールをアクティブにしていない可能性があります。各ユーザーは `select current_role();` コマンドを使用して、現在のセッションでアクティブになっているロールを確認できます。

ロールを手動でアクティブにしたい場合は、現在のセッションで [SET ROLE](../sql-reference/sql-statements/account-management/SET_ROLE.md) コマンドを使用して、そのロールをアクティブにして関連操作を行うことができます。

ログイン時に自動的にロールをアクティブにしたい場合、user_admin 管理者は [SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md) または [ALTER USER DEFAULT ROLE](../sql-reference/sql-statements/account-management/ALTER_USER.md) を使用して、各ユーザーにデフォルトロールを設定できます。設定に成功すると、ユーザーが再ログインするとデフォルトロールが自動的にアクティブになります。

システム内のすべてのユーザーがログイン時に所有しているすべてのロールをデフォルトでアクティブにしたい場合は、以下の操作を実行できます。この操作には System レベルの OPERATE 権限が必要です。

```SQL
SET GLOBAL activate_all_roles_on_login = TRUE;
```

ただし、最小限の権限原則を使用して、ユーザーに相対的に少ない権限を持つデフォルトロールを設定し、潜在的な誤操作を回避することをお勧めします。例えば、一般ユーザーには SELECT 権限のみを持つ read_only ロールをデフォルトロールとして設定し、ALTER、DROP、INSERT などの権限を持つロールをデフォルトロールとして設定しないことです。管理者には db_admin をデフォルトロールとして設定し、ノードのオンライン化やオフライン化ができる node_admin をデフォルトロールとして設定しないことです。

[GRANT](../sql-reference/sql-statements/account-management/GRANT.md) コマンドを使用して権限を付与できます。

## ユーザーにデータベース内のすべてのテーブルに対する権限を与えた `GRANT ALL ON ALL TABLES IN DATABASE <db_name> TO USER <user_identity>;` にもかかわらず、ユーザーはデータベース内でテーブルを作成できないのはなぜですか？

データベース内でテーブルを作成するにはデータベースレベルの権限が必要です。以下のように権限を付与する必要があります：

```SQL
GRANT CREATE TABLE ON DATABASE <db_name> TO USER <user_identity>;
```

## ユーザーにデータベースのすべての権限を与えた `GRANT ALL ON DATABASE <db_name> TO USER <user_identity>;` にもかかわらず、データベースで `SHOW TABLES;` を実行しても何も返されないのはなぜですか？

`SHOW TABLES;` は、現在のユーザーが任意の権限を持つテーブルのみを返します。SELECT 権限を例にとると、以下のように権限を付与できます：

```SQL
GRANT SELECT ON ALL TABLES IN DATABASE <db_name> TO USER <user_identity>;
```

このステートメントはバージョン 3.0 以前の `GRANT select_priv ON db.* TO <user_identity>;` と同等です。
