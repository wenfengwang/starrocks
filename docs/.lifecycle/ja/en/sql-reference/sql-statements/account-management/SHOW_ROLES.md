---
displayed_sidebar: English
---

# SHOW ROLES

## 説明

システム内のすべてのロールを表示します。`SHOW GRANTS FOR ROLE <role_name>;` を使用して特定のロールの権限を確認できます。詳細は [SHOW GRANTS](SHOW_GRANTS.md) を参照してください。このコマンドはv3.0からサポートされています。


> 注意: このステートメントを実行できるのは `user_admin` ロールのみです。

## 構文

```SQL
SHOW ROLES
```

返されるフィールド:

| **フィールド** | **説明**             |
| -------------- | -------------------- |
| Name           | ロールの名前です。   |

## 例

システム内のすべてのロールを表示します。

```SQL
mysql> SHOW ROLES;
+---------------+
| Name          |
+---------------+
| root          |
| db_admin      |
| cluster_admin |
| user_admin    |
| public        |
| testrole      |
+---------------+
```

## 参照

- [CREATE ROLE](CREATE_ROLE.md)
- [ALTER USER](ALTER_USER.md)
- [DROP ROLE](DROP_ROLE.md)
