---
displayed_sidebar: Chinese
---

# プラグインのインストール

## 機能

この文はプラグインをインストールするために使用されます。

:::tip

この操作にはSYSTEMレベルのPLUGIN権限が必要です。ユーザーに権限を付与するには [GRANT](../account-management/GRANT.md) を参照してください。

:::

## 文法

```sql
INSTALL PLUGIN FROM [source] [PROPERTIES ("key"="value", ...)]
```

**source は以下の三つのタイプをサポートしています:**

```plain text
1. zipファイルの絶対パスを指します。
2. プラグインディレクトリの絶対パスを指します。
3. httpまたはhttpsプロトコルのzipファイルダウンロードパスを指します。
```

**PROPERTIES：**

プラグインのいくつかの設定をすることができます。例えば、zipファイルのmd5sumの値を設定するなどです。

## 例

1. ローカルのzipファイルプラグインをインストールする:

    ```sql
    INSTALL PLUGIN FROM "/home/users/starrocks/auditdemo.zip";
    ```

2. ローカルディレクトリにあるプラグインをインストールする：

    ```sql
    INSTALL PLUGIN FROM "/home/users/starrocks/auditdemo/";
    ```

3. プラグインをダウンロードしてインストールする：

    ```sql
    INSTALL PLUGIN FROM "http://mywebsite.com/plugin.zip";
    ```

4. プラグインをダウンロードしてインストールし、同時にzipファイルのmd5sumの値を設定する:

    ```sql
    INSTALL PLUGIN FROM "http://mywebsite.com/plugin.zip" PROPERTIES("md5sum" = "73877f6029216f4314d712086a146570");
    ```
