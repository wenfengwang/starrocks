---
displayed_sidebar: English
---

# プラグインのインストール

## 説明

このステートメントは、プラグインをインストールするために使用されます。

:::tip

この操作には、SYSTEMレベルのPLUGIN権限が必要です。[GRANT](../account-management/GRANT.md)の指示に従って、この権限を付与することができます。

:::

## 構文

```sql
INSTALL PLUGIN FROM [source] [PROPERTIES ("key"="value", ...)]
```

3種類のソースがサポートされています:

```plain text
1. zipファイルを指す絶対パス
2. プラグインディレクトリを指す絶対パス
3. zipファイルを指すhttpまたはhttpsのダウンロードリンク
```

PROPERTIESは、プラグインのいくつかの設定をサポートしています。例えば、zipファイルのmd5sum値を設定するなどです。

## 例

1. ローカルのzipファイルからプラグインをインストールする:

    ```sql
    INSTALL PLUGIN FROM "/home/users/starrocks/auditdemo.zip";
    ```

2. ローカルのインパスからプラグインをインストールする:

    ```sql
    INSTALL PLUGIN FROM "/home/users/starrocks/auditdemo/";
    ```

3. プラグインをダウンロードしてインストールする:

    ```sql
    INSTALL PLUGIN FROM "http://mywebsite.com/plugin.zip";
    ```

4. プラグインをダウンロードしてインストールし、同時にzipファイルのmd5sum値を設定する:

    ```sql
    INSTALL PLUGIN FROM "http://mywebsite.com/plugin.zip" PROPERTIES("md5sum" = "73877f6029216f4314d712086a146570");
    ```
