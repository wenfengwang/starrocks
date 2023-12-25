---
displayed_sidebar: Chinese
---

# デフォルトロールの設定

## 機能

ユーザーがログイン時にデフォルトでアクティブになるロールを設定します。このコマンドはバージョン3.0からサポートされています。

## 構文

```SQL
-- 指定したロールをユーザーのデフォルトロールとして設定します。
SET DEFAULT ROLE <role_name>[,<role_name>,..] TO <user_identity>;
-- ユーザーが持っているすべてのロールをデフォルトロールとして設定します。将来ユーザーに与えられるロールも含みます。
SET DEFAULT ROLE ALL TO <user_identity>;
-- どのロールもデフォルトロールとして設定しませんが、この場合ユーザーはデフォルトでpublicロールをアクティブにします。
SET DEFAULT ROLE NONE TO <user_identity>; 
```

## パラメータ説明

`role_name`: ユーザーが持っているロール名。

`user_identity`: ユーザー識別子。

## 注意事項

一般ユーザーは自分のデフォルトロールを設定することができ、`user_admin`は他のユーザーのデフォルトロールを設定することができます。設定する際には、ユーザーが対応するロールを既に持っていることを確認してください。

[SHOW GRANTS](SHOW_GRANTS.md) を使用して、持っているロールを確認できます。

## 例

現在のユーザーが持っているロールを確認します。

```SQL
SHOW GRANTS FOR test;
+--------------+---------+----------------------------------------------+
| UserIdentity | Catalog | Grants                                       |
+--------------+---------+----------------------------------------------+
| 'test'@'%'   | NULL    | GRANT 'db_admin', 'user_admin' TO 'test'@'%' |
+--------------+---------+----------------------------------------------+
```

例1：`db_admin`、`user_admin`ロールを`test`ユーザーのデフォルトロールとして設定します。

```SQL
SET DEFAULT ROLE db_admin TO test;
```

例2：`test`ユーザーが持っているすべてのロールをデフォルトロールとして設定します。将来ユーザーに与えられるロールも含みます。

```SQL
SET DEFAULT ROLE ALL TO test;
```

例3：`test`ユーザーのデフォルトロールをクリアします。

```SQL
SET DEFAULT ROLE NONE TO test;
```
