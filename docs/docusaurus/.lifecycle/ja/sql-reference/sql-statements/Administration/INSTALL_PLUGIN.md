---
displayed_sidebar: "Japanese"
---

# プラグインのインストール

## 説明

この文は、プラグインをインストールするために使用されます。

構文：

```sql
INSTALL PLUGIN FROM [ソース] [PROPERTIES("key"="value", ...)]
```

3種類のソースがサポートされています：

```plain text
1. zipファイルを指す絶対パス
2. プラグインディレクトリを指す絶対パス
3. zipファイルを指すhttpまたはhttpsのダウンロードリンク
```

PROPERTIESは、zipファイルのmd5sum値の設定など、プラグインのいくつかの設定をサポートしています。

## 例

1. ローカルのzipファイルからプラグインをインストールする：

    ```sql
    INSTALL PLUGIN FROM "/home/users/starrocks/auditdemo.zip";
    ```

2. ローカル内のパスからプラグインをインストールする：

    ```sql
    INSTALL PLUGIN FROM "/home/users/starrocks/auditdemo/";
    ```

3. プラグインをダウンロードしてインストールする：

    ```sql
    INSTALL PLUGIN FROM "http://mywebsite.com/plugin.zip";
    ```

4. プラグインをダウンロードしてインストールします。同時に、zipファイルのmd5sum値を設定します：

    ```sql
    INSTALL PLUGIN FROM "http://mywebsite.com/plugin.zip" PROPERTIES("md5sum" = "73877f6029216f4314d712086a146570");
    ```