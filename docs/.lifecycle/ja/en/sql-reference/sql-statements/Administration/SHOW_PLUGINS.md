---
displayed_sidebar: English
---

# SHOW PLUGINS

## 説明

このステートメントは、インストールされているプラグインを確認するために使用されます。

:::tip

この操作にはSYSTEMレベルのPLUGIN権限が必要です。この権限を付与するには、[GRANT](../account-management/GRANT.md)の指示に従ってください。

:::

## 構文

```sql
SHOW PLUGINS
```

このコマンドは、すべての組み込みプラグインとカスタムプラグインを表示します。

## 例

1. インストールされたプラグインを表示する:

    ```sql
    SHOW PLUGINS;
    ```
