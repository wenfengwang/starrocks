---
displayed_sidebar: "Japanese"
---

# JDBCカタログ

StarRocksはv3.0以降でJDBCカタログをサポートしています。

JDBCカタログは、インジェストなしでJDBCを介してアクセスされるデータソースからデータをクエリできる外部カタログの一種です。

また、JDBCカタログを使用して[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)に基づいてJDBCデータソースから直接データを変換してロードすることができます。

JDBCカタログは現在、MySQLとPostgreSQLをサポートしています。

## 前提条件

- StarRocksクラスタ内のFEおよびBEは、`driver_url`パラメータで指定されたダウンロードURLからJDBCドライバをダウンロードできます。
- 各BEノードの**$BE_HOME/bin/start_be.sh**ファイルで`JAVA_HOME`がJDK環境のパスであり、JRE環境のパスではないように適切に構成されている必要があります。たとえば、`export JAVA_HOME = <JDK_absolute_path>`を構成することができます。この構成は、スクリプトの先頭に追加し、構成が有効になるようにBEを再起動する必要があります。

## JDBCカタログの作成

### 構文

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES ("key"="value", ...)
```

### パラメータ

#### `catalog_name`

JDBCカタログの名前です。命名規則は次のとおりです。

- 名前には、文字、数字（0-9）、アンダースコア（_）を含めることができます。文字で始める必要があります。
- 名前は大文字と小文字が区別され、1023文字を超えることはできません。

#### `comment`

JDBCカタログの説明です。このパラメータはオプションです。

#### `PROPERTIES`

JDBCカタログのプロパティ。`PROPERTIES`には、次のパラメータが含まれている必要があります。

| **パラメータ**     | **説明**                                                     |
| ----------------- | ------------------------------------------------------------ |
| type              | リソースのタイプ。値を `jdbc` に設定します。           |
| user              | 対象データベースに接続するために使用されるユーザー名です。 |
| password          | 対象データベースに接続するために使用されるパスワードです。 |
| jdbc_uri          | JDBCドライバが対象データベースに接続するために使用するURIです。MySQLの場合、URIは `"jdbc:mysql://ip:port"` の形式です。PostgreSQLの場合、URIは `"jdbc:postgresql://ip:port/db_name"` の形式です。詳細については、[PostgreSQL](https://jdbc.postgresql.org/documentation/head/connect.html)を参照してください。 |
| driver_url        | JDBCドライバJARパッケージのダウンロードURLです。HTTP URLまたはファイルURLがサポートされています。たとえば、`https://repo1.maven.org/maven2/org/postgresql/postgresql/42.3.3/postgresql-42.3.3.jar`や`file:///home/disk1/postgresql-42.3.3.jar`です。<br />**注意**<br />JDBCドライバをFEおよびBEノードの同じパスに配置して、 `driver_url` をそのパスに設定し、`file:///<path>/to/the/driver`の形式である必要があります。 |
| driver_class      | JDBCドライバのクラス名です。一般的なデータベースエンジンのJDBCドライバクラス名は次のとおりです：<ul><li>MySQL: `com.mysql.jdbc.Driver`（MySQL v5.xおよびそれ以前）および `com.mysql.cj.jdbc.Driver`（MySQL v6.xおよびそれ以降）</li><li>PostgreSQL: `org.postgresql.Driver`</li></ul> |

> **注意**
>
> FEは、JDBCカタログの作成時にJDBCドライバJARパッケージをダウンロードし、BEは最初のクエリのタイミングでJDBCドライバJARパッケージをダウンロードします。ダウンロードにかかる時間は、ネットワーク状況に応じて異なります。

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

[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を使用して、現在のStarRocksクラスタ内のすべてのカタログをクエリできます。

```SQL
SHOW CATALOGS;
```

また、[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)を使用して、外部カタログの作成ステートメントをクエリできます。次の例は、`jdbc0`という名前のJDBCカタログの作成ステートメントをクエリしています。

```SQL
SHOW CREATE CATALOG jdbc0;
```

## JDBCカタログの削除

[JDBCカタログ](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)を使用して、JDBCカタログを削除できます。

次の例では、`jdbc0`という名前のJDBCカタログを削除しています。

```SQL
DROP Catalog jdbc0;
```

## JDBCカタログのテーブルのクエリ

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を使用して、JDBC互換クラスタ内のデータベースを表示します。

   ```SQL
   SHOW DATABASES <catalog_name>;
   ```

2. [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用して、現在のセッションで送信先カタログに切り替えます。

    ```SQL
    SET CATALOG <catalog_name>;
    ```

    次に、[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して、現在のセッションでアクティブなデータベースを指定します。

    ```SQL
    USE <db_name>;
    ```

    または、[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して、送信先カタログで直接アクティブなデータベースを指定できます。

    ```SQL
    USE <catalog_name>.<db_name>;
    ```

3. [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用して、指定したデータベースの送信先テーブルをクエリします。

   ```SQL
   SELECT * FROM <table_name>;
   ```

## よくある質問

"Malformed database URL, failed to parse the main URL sections"というエラーが示された場合はどうすればよいですか？

そのようなエラーが発生した場合は、`jdbc_uri`で渡したURIが無効です。渡すURIを確認し、有効であることを確認してください。詳細については、このトピックの「[PROPERTIES](#properties)」セクション内のパラメータの説明を参照してください。