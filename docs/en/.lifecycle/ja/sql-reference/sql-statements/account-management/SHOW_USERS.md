---
displayed_sidebar: "Japanese"
---

# ユーザーの表示

## 説明

システム内のすべてのユーザーを表示します。ここで言及されているユーザーはユーザーの名前ではなく、ユーザーの識別子です。ユーザーの識別子についての詳細は、[CREATE USER](CREATE_USER.md)を参照してください。このコマンドはv3.0からサポートされています。

特定のユーザーの権限を表示するには、`SHOW GRANTS FOR <user_identity>;`を使用します。詳細については、[SHOW GRANTS](SHOW_GRANTS.md)を参照してください。

> 注意: `user_admin`ロールのみがこのステートメントを実行できます。

## 構文

```SQL
SHOW USERS
```

返されるフィールド:

| **フィールド** | **説明**    |
| --------- | ------------------ |
| User      | ユーザーの識別子。 |

## 例

システム内のすべてのユーザーを表示します。

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

## 参考

[CREATE USER](CREATE_USER.md), [ALTER USER](ALTER_USER.md), [DROP USER](DROP_USER.md)
