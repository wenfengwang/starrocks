---
displayed_sidebar: English
---

# プラグインのアンインストール

## 説明

この文は、プラグインをアンインストールするために使用されます。

:::tip

この操作には、SYSTEMレベルのPLUGIN権限が必要です。[GRANT](../account-management/GRANT.md)の指示に従って、この権限を付与することができます。

:::

## 構文

```SQL
UNINSTALL PLUGIN <plugin_name>
```

plugin_nameはSHOW PLUGINSコマンドを使用して確認できます。

組み込みでないプラグインのみがアンインストール可能です。

## 例

1. プラグインをアンインストールする:

    ```SQL
    UNINSTALL PLUGIN auditdemo;
    ```
