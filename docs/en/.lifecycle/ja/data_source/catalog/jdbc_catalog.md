---
displayed_sidebar: "Japanese"
---

# JDBCカタログ

StarRocksはv3.0以降、JDBCカタログをサポートしています。

JDBCカタログは、インジェストなしでJDBCを介してアクセスするデータソースからデータをクエリするための外部カタログの一種です。

また、JDBCカタログを基にして[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)を使用して、JDBCデータソースから直接データを変換およびロードすることもできます。

現在、JDBCカタログはMySQLとPostgreSQLをサポートしています。

## 前提条件

- StarRocksクラスタのFEとBEは、`driver_url`パラメータで指定されたダウンロードURLからJDBCドライバをダウンロードできる必要があります。
- 各BEノードの**$BE_HOME/bin/start_be.sh**ファイルの`JAVA_HOME`がJRE環境のパスではなく、JDK環境のパスとして正しく設定されている必要があります。たとえば、`export JAVA_HOME=<JDKの絶対パス>`と設定することができます。この設定はスクリプトの先頭に追加し、BEを再起動して設定が有効になるようにする必要があります。

## JDBCカタログの作成

### 構文

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES ("key"="value", ...)
```

### パラメータ

#### `catalog_name`

JDBCカタログの名前です。命名規則は次のとおりです：

- 名前には、文字、数字（0-9）、アンダースコア（_）を含めることができます。ただし、文字で始める必要があります。
- 名前は大文字と小文字を区別し、長さが1023文字を超えることはできません。

#### `comment`

JDBCカタログの説明です。このパラメータはオプションです。

#### `PROPERTIES`

JDBCカタログのプロパティです。`PROPERTIES`には次のパラメータを含める必要があります：

| **パラメータ** | **説明**                                                                 |
| -------------- | ------------------------------------------------------------------------ |
| type           | リソースのタイプです。値を`jdbc`に設定します。                           |
| user           | ターゲットデータベースに接続するために使用されるユーザー名です。           |
| password       | ターゲットデータベースに接続するために使用されるパスワードです。           |
| jdbc_uri       | JDBCドライバがターゲットデータベースに接続するために使用するURIです。MySQLの場合、URIは`"jdbc:mysql://ip:port"`の形式です。PostgreSQLの場合、URIは`"jdbc:postgresql://ip:port/db_name"`の形式です。詳細については、[PostgreSQL](https://jdbc.postgresql.org/documentation/head/connect.html)を参照してください。 |
| driver_url     | JDBCドライバのJARパッケージのダウンロードURLです。HTTP URLまたはファイルURLがサポートされています。たとえば、`https://repo1.maven.org/maven2/org/postgresql/postgresql/42.3.3/postgresql-42.3.3.jar`や`file:///home/disk1/postgresql-42.3.3.jar`です。<br />**注意**<br />JDBCドライバをFEおよびBEノードの同じパスに配置し、`driver_url`をそのパスに設定することもできます。このパスは`file:///<path>/to/the/driver`の形式である必要があります。 |
| driver_class   | JDBCドライバのクラス名です。一般的なデータベースエンジンのJDBCドライバのクラス名は次のとおりです：<ul><li>MySQL：`com.mysql.jdbc.Driver`（MySQL v5.xおよびそれ以前）および`com.mysql.cj.jdbc.Driver`（MySQL v6.xおよびそれ以降）</li><li>PostgreSQL：`org.postgresql.Driver`</li></ul> |

> **注意**
>
> FEはJDBCカタログ作成時にJDBCドライバのJARパッケージをダウンロードし、BEは最初のクエリ時にJDBCドライバのJARパッケージをダウンロードします。ダウンロードにかかる時間はネットワークの状況によって異なります。

### 例

次の例では、`jdbc0`と`jdbc1`という2つのJDBCカタログを作成しています。

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

## JDBCカタログの表示

[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を使用して、現在のStarRocksクラスタのすべてのカタログをクエリできます：

```SQL
SHOW CATALOGS;
```

また、[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)を使用して、外部カタログの作成ステートメントをクエリすることもできます。次の例では、`jdbc0`という名前のJDBCカタログの作成ステートメントをクエリしています：

```SQL
SHOW CREATE CATALOG jdbc0;
```

## JDBCカタログの削除

[JDBCカタログの削除](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)を使用して、JDBCカタログを削除できます。

次の例では、`jdbc0`という名前のJDBCカタログを削除しています：

```SQL
DROP Catalog jdbc0;
```

## JDBCカタログ内のテーブルのクエリ

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を使用して、JDBC互換のクラスタ内のデータベースを表示します：

   ```SQL
   SHOW DATABASES <catalog_name>;
   ```

2. [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用して、現在のセッションで目的のカタログに切り替えます：

    ```SQL
    SET CATALOG <catalog_name>;
    ```

    次に、[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して、現在のセッションでアクティブなデータベースを指定します：

    ```SQL
    USE <db_name>;
    ```

    または、[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して、目的のカタログでアクティブなデータベースを直接指定することもできます：

    ```SQL
    USE <catalog_name>.<db_name>;
    ```

3. [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用して、指定したデータベースの目的のテーブルをクエリします：

   ```SQL
   SELECT * FROM <table_name>;
   ```

## FAQ

「Malformed database URL, failed to parse the main URL sections」というエラーが表示された場合はどうすればよいですか？

このようなエラーが発生した場合、`jdbc_uri`に渡したURIが無効です。渡すURIを確認し、有効であることを確認してください。詳細については、このトピックの"[PROPERTIES](#properties)"セクションのパラメータの説明を参照してください。
