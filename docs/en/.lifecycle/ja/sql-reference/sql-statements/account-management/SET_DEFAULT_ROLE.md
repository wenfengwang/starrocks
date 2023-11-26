---
displayed_sidebar: "Japanese"
---

# デフォルトのロールを設定する

## 説明

ユーザーがサーバーに接続した際にデフォルトで有効になるロールを設定します。

このコマンドはv3.0からサポートされています。

## 構文

```SQL
-- 指定されたロールをデフォルトのロールとして設定します。
SET DEFAULT ROLE <role_name>[,<role_name>,..] TO <user_identity>;
-- このユーザーに割り当てられるロールを含む、ユーザーのすべてのロールをデフォルトのロールとして設定します。
SET DEFAULT ROLE ALL TO <user_identity>;
-- デフォルトのロールは設定されませんが、ユーザーログイン後にはパブリックロールが有効になります。
SET DEFAULT ROLE NONE TO <user_identity>; 
```

## パラメーター

`role_name`: ロール名

`user_identity`: ユーザーの識別子

## 使用上の注意

個々のユーザーは自分自身のデフォルトのロールを設定できます。`user_admin`は他のユーザーのデフォルトのロールを設定できます。この操作を実行する前に、ユーザーにこれらのロールがすでに割り当てられていることを確認してください。

[SHOW GRANTS](SHOW_GRANTS.md)を使用して、ユーザーのロールをクエリできます。

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

例1: ユーザー`test`のデフォルトのロールとして`db_admin`と`user_admin`を設定します。

```SQL
SET DEFAULT ROLE db_admin TO test;
```

例2: ユーザー`test`のすべてのロール、このユーザーに割り当てられるロールをデフォルトのロールとして設定します。

```SQL
SET DEFAULT ROLE ALL TO test;
```

例3: ユーザー`test`のすべてのデフォルトのロールをクリアします。

```SQL
SET DEFAULT ROLE NONE TO test;
```
