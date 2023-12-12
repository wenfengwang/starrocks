---
displayed_sidebar: "Japanese"
---

# プラグインのアンインストール

## 説明

このステートメントは、プラグインをアンインストールするために使用されます。

構文:

```SQL
UNINSTALL PLUGIN <プラグイン名>
```

プラグイン名は、SHOW PLUGINS コマンドを使用して表示することができます。

組み込みのプラグイン以外は、アンインストールすることができます。

## 例

1. プラグインをアンインストールする:

    ```SQL
    UNINSTALL PLUGIN auditdemo;
    ```