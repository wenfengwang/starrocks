---
displayed_sidebar: Chinese
---

# プラグインのアンインストール

## 機能

このステートメントはプラグインをアンインストールするために使用されます。

:::tip

この操作にはSYSTEMレベルのPLUGIN権限が必要です。ユーザーに権限を付与するには [GRANT](../account-management/GRANT.md) を参照してください。

:::

## 文法

```SQL
UNINSTALL PLUGIN plugin_name
```

plugin_name は `SHOW PLUGINS;` コマンドで確認できます。

builtinではないプラグインのみアンインストールできます。

## 例

1. プラグインをアンインストールする：

    ```SQL
    UNINSTALL PLUGIN auditdemo;
    ```
