---
displayed_sidebar: Chinese
---

# JDBC カタログ

StarRocks はバージョン 3.0 から JDBC カタログをサポートしています。

JDBC カタログは External Catalog の一種です。JDBC カタログを使用すると、データをインポートすることなく、JDBC データソース内のデータを直接クエリできます。

さらに、JDBC カタログを基に [INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) 機能を組み合わせて、JDBC データソースのデータを変換およびインポートすることができます。

現在、JDBC カタログは MySQL と PostgreSQL をサポートしています。

## 前提条件

- FE と BE が `driver_url` で指定されたダウンロードパスを通じて、必要な JDBC ドライバーをダウンロードできることを確認してください。
- BE が稼働しているマシンの起動スクリプト **$BE_HOME/bin/start_be.sh** には `JAVA_HOME` の設定が必要です。JDK 環境に設定し、JRE 環境には設定しないでください。例えば `export JAVA_HOME=<JDK の絶対パス>` とします。この設定は BE 起動スクリプトの最初に追加する必要があり、追加後には BE を再起動する必要があります。

## JDBC カタログの作成

### 構文

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES ("key"="value", ...)
```

### パラメータ説明

#### `catalog_name`

JDBC カタログの名前です。命名規則は以下の通りです：

- 英字 (a-z または A-Z)、数字 (0-9)、アンダースコア (_) で構成され、英字で始まる必要があります。
- 全体の長さは 1023 文字を超えてはいけません。
- カタログ名は大文字と小文字を区別します。

#### `comment`

JDBC カタログの説明です。このパラメータはオプションです。

#### PROPERTIES

JDBC カタログのプロパティで、以下の必須設定項目が含まれています：

| **パラメータ** | **説明**                                                     |
| -------------- | ------------------------------------------------------------ |
| type           | リソースタイプで、固定値は `jdbc` です。                    |
| user           | 対象データベースのログインユーザー名です。                   |
| password       | 対象データベースのユーザーパスワードです。                   |
| jdbc_uri       | JDBC ドライバーが対象データベースに接続するための URI です。MySQL を使用する場合は `"jdbc:mysql://ip:port"` の形式、PostgreSQL を使用する場合は `"jdbc:postgresql://ip:port/db_name"` の形式です。 |
| driver_url     | JDBC ドライバーの JAR パッケージをダウンロードするための URL です。HTTP プロトコルまたは file プロトコルを使用できます。例えば `https://repo1.maven.org/maven2/org/postgresql/postgresql/42.3.3/postgresql-42.3.3.jar` や `file:///home/disk1/postgresql-42.3.3.jar` です。<br />**注記**<br />JDBC ドライバーを FE または BE が稼働しているノード上の任意の同じパスにデプロイし、`driver_url` をそのパスに設定することもできます。形式は `file:///<path>/to/the/driver` です。 |
| driver_class   | JDBC ドライバーのクラス名です。以下は一般的なデータベースエンジンがサポートする JDBC ドライバーのクラス名です：<ul><li>MySQL：`com.mysql.jdbc.Driver`（MySQL 5.x 以前のバージョン）、`com.mysql.cj.jdbc.Driver`（MySQL 6.x 以降のバージョン）</li><li>PostgreSQL: `org.postgresql.Driver`</li></ul> |

> **注記**
>
> FE は JDBC カタログを作成する際に JDBC ドライバーを取得し、BE は初めてクエリを実行する際にドライバーを取得します。ドライバーの取得にかかる時間はネットワーク状況に依存します。

### 作成例

以下の例では、`jdbc0` と `jdbc1` の二つの JDBC カタログを作成しています。

```SQL
CREATE EXTERNAL CATALOG jdbc0
PROPERTIES
(
    "type"="jdbc",
    "user"="postgres",
    "password"="changeme",
    "jdbc_uri"="jdbc:postgresql://127.0.0.1:5432/jdbc_test",
    "driver_url"="https://repo1.maven.org/maven2/org/postgresql/postgresql/42.3.3/postgresql-42.3.3.jar",
    "driver_class"="org.postgresql.Driver"
);

CREATE EXTERNAL CATALOG jdbc1
PROPERTIES
(
    "type"="jdbc",
    "user"="root",
    "password"="changeme",
    "jdbc_uri"="jdbc:mysql://127.0.0.1:3306",
    "driver_url"="https://repo1.maven.org/maven2/mysql/mysql-connector-java/8.0.28/mysql-connector-java-8.0.28.jar",
    "driver_class"="com.mysql.cj.jdbc.Driver"
);
```

## JDBC カタログの表示

[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md) を使用して、現在の StarRocks クラスター内のすべてのカタログを照会できます。

```SQL
SHOW CATALOGS;
```

また、[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md) を使用して、特定の External Catalog の作成ステートメントを照会することもできます。例えば、以下のコマンドで JDBC カタログ `jdbc0` の作成ステートメントを照会できます：

```SQL
SHOW CREATE CATALOG jdbc0;
```

## JDBC カタログの削除

[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md) を使用して JDBC カタログを削除できます。

例えば、以下のコマンドで JDBC カタログ `jdbc0` を削除します：

```SQL
DROP CATALOG jdbc0;
```

## JDBC カタログ内のテーブルデータのクエリ

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md) を使用して、特定のカタログに属するクラスター内のデータベースを表示します：

   ```SQL
   SHOW DATABASES FROM <catalog_name>;
   ```

2. [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md) を使用して、現在のセッションで有効なカタログを切り替えます：

    ```SQL
    SET CATALOG <catalog_name>;
    ```

    次に [USE](../../sql-reference/sql-statements/data-definition/USE.md) を使用して、現在のセッションで有効なデータベースを指定します：

    ```SQL
    USE <db_name>;
    ```

    または、[USE](../../sql-reference/sql-statements/data-definition/USE.md) を使用して、セッションを目的のカタログの特定のデータベースに直接切り替えることもできます：

    ```SQL
    USE <catalog_name>.<db_name>;
    ```

3. [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用して、対象データベース内の対象テーブルをクエリします：

   ```SQL
   SELECT * FROM <table_name>;
   ```

## よくある質問

システムが "不正な形式のデータベース URL, main URL セクションの解析に失敗しました" というエラーを返した場合、どのように対処すればよいですか？

このエラーは通常、`jdbc_uri` に入力された URI が誤っているために発生します。入力された URI が正しいことを確認してください。詳細は本文の "[PROPERTIES](#properties)" セクションを参照してください。
