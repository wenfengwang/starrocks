---
displayed_sidebar: "Japanese"
---

# プラグインのインストール

## 説明

このステートメントは、プラグインをインストールするために使用されます。

構文:

```sql
INSTALL PLUGIN FROM [ソース] [PROPERTIES ("key"="value", ...)]
```

3つの種類のソースがサポートされています:

```plain text
1. zipファイルへの絶対パス
2. プラグインディレクトリへの絶対パス
3. zipファイルへのhttpまたはhttpsのダウンロードリンク
```

PROPERTIESは、プラグインのいくつかの設定を行うことができます。例えば、zipファイルのmd5sum値の設定などがあります。

## 例

1. ローカルのzipファイルからプラグインをインストールする場合:

    ```sql
    INSTALL PLUGIN FROM "/home/users/starrocks/auditdemo.zip";
    ```

2. ローカルのinpathからプラグインをインストールする場合:

    ```sql
    INSTALL PLUGIN FROM "/home/users/starrocks/auditdemo/";
    ```

3. プラグインをダウンロードしてインストールする場合:

    ```sql
    INSTALL PLUGIN FROM "http://mywebsite.com/plugin.zip";
    ```

4. プラグインをダウンロードしてインストールします。同時に、zipファイルのmd5sum値を設定します:

    ```sql
    INSTALL PLUGIN FROM "http://mywebsite.com/plugin.zip" PROPERTIES("md5sum" = "73877f6029216f4314d712086a146570");
    ```