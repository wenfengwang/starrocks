---
displayed_sidebar: Chinese
---

# ユーザーの表示

## 機能

現在のシステムに存在する全てのユーザーを表示します。ここでのユーザーはユーザー名ではなく、ユーザー識別子（user identity）を指します。詳細は [CREATE USER](CREATE_USER.md) を参照してください。このコマンドはバージョン 3.0 からサポートされています。

特定のユーザーの権限を確認するには `SHOW GRANTS FOR <user_identity>;` を使用します。詳細は [SHOW GRANTS](SHOW_GRANTS.md) を参照してください。

> 注意：このコマンドを実行できるのは `user_admin` ロールのみです。

## 文法

```SQL
SHOW USERS
```

返されるフィールドの説明：

| **フィールド名** | **説明**   |
| ------------ | ---------- |
| User         | ユーザー識別子。 |

## 例

システムに存在する全てのユーザーを表示します。

```SQL
mysql> SHOW USERS;
+-----------------+
| User            |
+-----------------+
| 'lily'@'%'      |
| 'root'@'%'      |
| 'admin'@'%'     |
| 'jack'@'%'      |
| 'tom'@'%'       |
+-----------------+
```

## 関連文書

- [CREATE USER](CREATE_USER.md)
- [ALTER USER](ALTER_USER.md)
- [DROP USER](DROP_USER.md)
