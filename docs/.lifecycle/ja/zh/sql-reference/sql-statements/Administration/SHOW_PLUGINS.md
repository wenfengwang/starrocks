---
displayed_sidebar: Chinese
---

# プラグインを表示

## 機能

この文は、インストールされたプラグインを表示するために使用されます。

:::tip

この操作にはSYSTEMレベルのPLUGIN権限が必要です。ユーザーに権限を付与するには [GRANT](../account-management/GRANT.md) を参照してください。

:::

## 文法

```sql
SHOW PLUGINS
```

このコマンドは、ユーザーがインストールしたプラグインとシステムに組み込まれているプラグインを表示します。

## 例

1. インストールされたプラグインを表示する：

    ```sql
    SHOW PLUGINS;
    ```
