---
displayed_sidebar: "Japanese"
---

# ロールの表示

## 説明

システム内のすべてのロールを表示します。特定のロールの権限を表示するには、`SHOW GRANTS FOR ROLE <role_name>;` を使用します。詳細については、[SHOW GRANTS](SHOW_GRANTS.md) を参照してください。このコマンドはv3.0からサポートされています。

> 注意: `user_admin` ロールのみがこのステートメントを実行できます。

## 構文

```SQL
SHOW ROLES
```

返されるフィールド:

| **フィールド** | **説明**       |
| --------- | --------------------- |
| Name      | ロールの名前。 |

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

## 参考

- [CREATE ROLE](CREATE_ROLE.md)
- [ALTER USER](ALTER_USER.md)
- [DROP ROLE](DROP_ROLE.md)
