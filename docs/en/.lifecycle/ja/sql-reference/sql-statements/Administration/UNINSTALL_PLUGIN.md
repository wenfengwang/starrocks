---
displayed_sidebar: "Japanese"
---

# プラグインのアンインストール

## 説明

この文は、プラグインをアンインストールするために使用されます。

構文:

```SQL
UNINSTALL PLUGIN <プラグイン名>
```

プラグイン名は、SHOW PLUGINSコマンドで確認できます。

組み込みプラグイン以外はアンインストールできません。

## 例

1. プラグインのアンインストール:

    ```SQL
    UNINSTALL PLUGIN auditdemo;
    ```
