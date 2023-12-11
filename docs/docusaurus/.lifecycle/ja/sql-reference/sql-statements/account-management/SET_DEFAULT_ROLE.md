---
displayed_sidebar: "Japanese"
---

# デフォルトの役割を設定する

## 説明

ユーザーがサーバーに接続する際にデフォルトでアクティブになる役割を設定します。

このコマンドはv3.0からサポートされています。

## 構文

```SQL
-- 指定された役割をデフォルトの役割として設定します。
SET DEFAULT ROLE <role_name>[,<role_name>,..] TO <user_identity>;
-- このユーザに割り当てられる役割を含む、ユーザのすべての役割をデフォルトの役割として設定します。
SET DEFAULT ROLE ALL TO <user_identity>;
-- デフォルトの役割は設定されませんが、ユーザのログイン後にはpublic役割が有効のままです。
SET DEFAULT ROLE NONE TO <user_identity>;
```

## パラメーター

`role_name`: 役割名

`user_identity`: ユーザー識別子

## 使用上の注意

個々のユーザーは自分自身のためにデフォルトの役割を設定できます。`user_admin`は他のユーザーのためにデフォルトの役割を設定できます。この操作を実行する前に、ユーザーがこれらの役割にすでに割り当てられていることを確認してください。

[SHOW GRANTS](SHOW_GRANTS.md)を使用してユーザーの役割をクエリできます。

## 例

現在のユーザーの役割をクエリします。

```SQL
SHOW GRANTS FOR test;
+--------------+---------+----------------------------------------------+
| UserIdentity | Catalog | Grants                                       |
+--------------+---------+----------------------------------------------+
| 'test'@'%'   | NULL    | GRANT 'db_admin', 'user_admin' TO 'test'@'%' |
+--------------+---------+----------------------------------------------+
```

例1: `test`ユーザーのデフォルトの役割として`db_admin`および`user_admin`役割を設定します。

```SQL
SET DEFAULT ROLE db_admin TO test;
```

例2: `test`ユーザーのすべての役割と、このユーザに割り当てられる役割をデフォルトの役割として設定します。

```SQL
SET DEFAULT ROLE ALL TO test;
```

例3: `test`ユーザーのすべてのデフォルトの役割をクリアします。

```SQL
SET DEFAULT ROLE NONE TO test;
```