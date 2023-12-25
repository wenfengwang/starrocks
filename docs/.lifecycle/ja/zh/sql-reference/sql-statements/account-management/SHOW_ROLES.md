---
displayed_sidebar: Chinese
---

# SHOW ROLES

## 機能

現在のシステムに存在するすべてのロールを表示します。ロールの権限情報を確認するには、`SHOW GRANTS FOR ROLE <role_name>;`を使用してください。詳細は [SHOW GRANTS](SHOW_GRANTS.md) を参照してください。

このコマンドはバージョン3.0からサポートされています。

> 注意：`user_admin` ロールを持つユーザーのみがこのコマンドを実行できます。

## 文法

```SQL
SHOW ROLES
```

返されるフィールドの説明：

| **フィールド** | **説明**   |
| -------------- | ---------- |
| Name           | ロール名。 |

## 例

システムに存在するすべてのロールを表示します。

```Plain
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

## 関連リファレンス

- [CREATE ROLE](CREATE_ROLE.md)
- [ALTER USER](ALTER_USER.md)
- [SET ROLE](SET_ROLE.md)
- [DROP ROLE](DROP_ROLE.md)
