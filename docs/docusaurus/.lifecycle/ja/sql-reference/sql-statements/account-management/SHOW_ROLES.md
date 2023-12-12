---
displayed_sidebar: "Japanese"
---

# ロールの表示

## 説明

システム内のすべてのロールを表示します。特定のロールの権限を表示するには、`SHOW GRANTS FOR ROLE <role_name>;` を使用できます。詳細については、[SHOW GRANTS](SHOW_GRANTS.md) を参照してください。このコマンドはv3.0からサポートされています。

> 注：`user_admin` ロールだけがこのステートメントを実行できます。

## 構文

```SQL
SHOW ROLES
```

戻り値：

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

## 参照

- [CREATE ROLE](CREATE_ROLE.md)
- [ALTER USER](ALTER_USER.md)
- [DROP ROLE](DROP_ROLE.md)