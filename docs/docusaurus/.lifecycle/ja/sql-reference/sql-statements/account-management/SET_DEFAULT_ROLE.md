---
displayed_sidebar: "Japanese"
---

# デフォルトのロールを設定

## 説明

ユーザーがサーバーに接続する際にデフォルトでアクティブ化されるロールを設定します。

このコマンドはv3.0からサポートされています。

## 構文

```SQL
-- 指定されたロールをデフォルトロールとして設定します。
SET DEFAULT ROLE <role_name>[,<role_name>,..] TO <user_identity>;
-- このユーザーに割り当てられるロールを含む、ユーザーのすべてのロールをデフォルトロールとして設定します。
SET DEFAULT ROLE ALL TO <user_identity>;
-- ユーザーログイン後にパブリックロールは有効のままですが、デフォルトロールは設定されていません。
SET DEFAULT ROLE NONE TO <user_identity>;
```

## パラメーター

`role_name`: ロール名

`user_identity`: ユーザーID

## 使用上の注意

個々のユーザーは自分自身のデフォルトロールを設定できます。`user_admin`は他のユーザーのデフォルトロールを設定できます。この操作を行う前に、ユーザーがこれらのロールにすでに割り当てられていることを確認してください。

[SHOW GRANTS](SHOW_GRANTS.md)を使用してユーザーのロールをクエリできます。

## 例

現在のユーザーのロールをクエリします。

```SQL
SHOW GRANTS FOR test;
+--------------+---------+----------------------------------------------+
| UserIdentity | Catalog | Grants                                       |
+--------------+---------+----------------------------------------------+
| 'test'@'%'   | NULL    | GRANT 'db_admin', 'user_admin' TO 'test'@'%' |
+--------------+---------+----------------------------------------------+
```

例1: ユーザー`test`のデフォルトロールとして`db_admin`と`user_admin`のロールを設定します。

```SQL
SET DEFAULT ROLE db_admin TO test;
```

例2: ユーザー`test`のすべてのロール、このユーザーに割り当てられるロールを含む、をデフォルトロールとして設定します。

```SQL
SET DEFAULT ROLE ALL TO test;
```

例3: ユーザー`test`のすべてのデフォルトロールをクリアします。

```SQL
SET DEFAULT ROLE NONE TO test;
```