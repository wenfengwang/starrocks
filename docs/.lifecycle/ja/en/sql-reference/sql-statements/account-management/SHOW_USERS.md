---
displayed_sidebar: English
---

# ユーザーの表示

## 説明

システム内の全てのユーザーを表示します。ここで言及されるユーザーはユーザー名ではなく、ユーザー識別子です。ユーザー識別子についての詳細は、[CREATE USER](CREATE_USER.md)を参照してください。このコマンドはv3.0からサポートされています。

`SHOW GRANTS FOR <user_identity>;` を使用して、特定のユーザーの権限を確認できます。詳細は、[SHOW GRANTS](SHOW_GRANTS.md)を参照してください。

> 注意: このステートメントを実行できるのは `user_admin` ロールのみです。

## 構文

```SQL
SHOW USERS
```

返されるフィールド:

| **フィールド** | **説明**    |
| --------- | ------------------ |
| User      | ユーザー識別子。 |

## 例

システム内の全てのユーザーを表示します。

```SQL
mysql> SHOW USERS;
+-----------------+
| User            |
+-----------------+
| 'wybing5'@'%'   |
| 'root'@'%'      |
| 'admin'@'%'     |
| 'star'@'%'      |
| 'wybing_30'@'%' |
| 'simo'@'%'      |
| 'wybing1'@'%'   |
| 'wybing2'@'%'   |
+-----------------+
```

## 参照

[CREATE USER](CREATE_USER.md)、[ALTER USER](ALTER_USER.md)、[DROP USER](DROP_USER.md)
