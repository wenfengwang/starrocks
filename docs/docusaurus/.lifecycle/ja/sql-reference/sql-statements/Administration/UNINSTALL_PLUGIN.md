---
displayed_sidebar: "Japanese"
---

# プラグインのアンインストール

## 説明

このステートメントはプラグインをアンインストールするために使用されます。

構文:

```SQL
UNINSTALL PLUGIN <plugin_name>
```

プラグイン名はSHOW PLUGINSコマンドで表示できます

組み込みプラグイン以外はアンインストールできません。

## 例

1. プラグインをアンインストールする:

    ```SQL
    UNINSTALL PLUGIN auditdemo;
    ```