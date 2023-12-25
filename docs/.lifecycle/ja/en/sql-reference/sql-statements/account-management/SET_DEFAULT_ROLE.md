---
displayed_sidebar: English
---

# デフォルトロールの設定

## 説明

ユーザーがサーバーに接続したときにデフォルトでアクティブ化されるロールを設定します。

このコマンドは v3.0 からサポートされています。

## 構文

```SQL
-- 指定されたロールをデフォルトロールとして設定します。
SET DEFAULT ROLE <role_name>[,<role_name>,..] TO <user_identity>;
-- ユーザーに割り当てられるロールを含む、ユーザーのすべてのロールをデフォルトロールとして設定します。
SET DEFAULT ROLE ALL TO <user_identity>;
-- デフォルトロールは設定されませんが、ユーザーログイン後もpublicロールは有効です。
SET DEFAULT ROLE NONE TO <user_identity>; 
```

## パラメーター

`role_name`: ロール名

`user_identity`: ユーザー識別子

## 使用上の注意

個々のユーザーは自分自身のデフォルトロールを設定できます。`user_admin`は他のユーザーのデフォルトロールを設定することができます。この操作を実行する前に、ユーザーにこれらのロールがすでに割り当てられていることを確認してください。

ユーザーのロールを照会するには、[SHOW GRANTS](SHOW_GRANTS.md)を使用できます。

## 例

現在のユーザーのロールを照会します。

```SQL
SHOW GRANTS FOR test;
+--------------+---------+----------------------------------------------+
| UserIdentity | Catalog | Grants                                       |
+--------------+---------+----------------------------------------------+
| 'test'@'%'   | NULL    | GRANT 'db_admin', 'user_admin' TO 'test'@'%' |
+--------------+---------+----------------------------------------------+
```

例 1: ユーザー`test`に対して`db_admin`と`user_admin`をデフォルトロールとして設定します。

```SQL
SET DEFAULT ROLE db_admin, user_admin TO test;
```

例 2: ユーザー`test`に割り当てられるロールを含む、ユーザーのすべてのロールをデフォルトロールとして設定します。

```SQL
SET DEFAULT ROLE ALL TO test;
```

例 3: ユーザー`test`のすべてのデフォルトロールをクリアします。

```SQL
SET DEFAULT ROLE NONE TO test;
```
