---
displayed_sidebar: "Japanese"
---

# JDBCカタログ

StarRocksはv3.0以降でJDBCカタログをサポートしています。

JDBCカタログは、インジェストなしでJDBC経由でアクセスされるデータソースからデータをクエリする外部カタログの一種です。

また、JDBCカタログに基づいて[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)を使用してJDBCデータソースから直接データを変換およびロードすることができます。

JDBCカタログは現在、MySQLとPostgreSQLをサポートしています。

## 前提条件

- StarRocksクラスタのFEおよびBEは、`driver_url`パラメータによって指定されたダウンロードURLからJDBCドライバをダウンロードできます。
- 各BEノードの**$BE_HOME/bin/start_be.sh**ファイルの`JAVA_HOME`が、JRE環境のパスではなくJDK環境のパスに適切に設定されていること。たとえば、`export JAVA_HOME = <JDKの絶対パス>`と設定できます。この構成をスクリプトの先頭に追加し、構成が有効になるようにBEを再起動する必要があります。

## JDBCカタログの作成

### 構文

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES ("key"="value", ...)
```

### パラメータ

#### `catalog_name`

JDBCカタログの名前。命名規則は以下の通りです。

- 名前には、文字、数字（0〜9）、アンダースコア（_）を含めることができます。文字で始まる必要があります。
- 名前は大文字と小文字を区別し、長さが1023文字を超えることはできません。

#### `comment`

JDBCカタログの説明。このパラメータはオプションです。

#### `PROPERTIES`

JDBCカタログのプロパティ。`PROPERTIES`には次のパラメータが含まれている必要があります。

| **Parameter**     | **Description**                                                    |
| ----------------- | --------------------------------------------------------------- |
| type              | リソースのタイプ。値を`jdbc`に設定します。                       |
| user              | ターゲットデータベースに接続するために使用されるユーザー名。   |
| password          | ターゲットデータベースに接続するために使用されるパスワード。   |
| jdbc_uri          | JDBCドライバがターゲットデータベースに接続するために使用するURI。詳細については、該当するデータベース用のURI形式を参照してください。[PostgreSQL](https://jdbc.postgresql.org/documentation/head/connect.html) |
| driver_url        | JDBCドライバJARパッケージのダウンロードURL。HTTP URLやファイルURLがサポートされています。たとえば、`https://repo1.maven.org/maven2/org/postgresql/postgresql/42.3.3/postgresql-42.3.3.jar`および`file:///home/disk1/postgresql-42.3.3.jar`。<br />**注意**<br />JDBCドライバをFEおよびBEノードの同じパスに配置し、`driver_url`をそのパスに設定し、`file:///<path>/to/the/driver`形式である必要があります。 |
| driver_class      | JDBCドライバのクラス名。一般的なデータベースエンジンのJDBCドライバクラス名は次のとおりです。<ul><li>MySQL: `com.mysql.jdbc.Driver`（MySQL v5.xおよびそれ以前）、`com.mysql.cj.jdbc.Driver`（MySQL v6.xおよびそれ以降）</li><li>PostgreSQL: `org.postgresql.Driver`</li></ul> |

> **注意**
>
> FEはJDBCカタログの作成時にJDBCドライバJARパッケージをダウンロードし、BEは最初のクエリの実行時にJDBCドライバJARパッケージをダウンロードします。ダウンロードにかかる時間はネットワーク状況によって異なります。

### 例

次の例では、`jdbc0`および`jdbc1`という2つのJDBCカタログが作成されています。

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

現在のStarRocksクラスタにあるすべてのカタログをクエリするには、[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を使用できます。

```SQL
SHOW CATALOGS;
```

さらに、外部カタログの作成ステートメントをクエリするには、[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)を使用できます。次の例では、`jdbc0`という名前のJDBCカタログの作成ステートメントをクエリしています。

```SQL
SHOW CREATE CATALOG jdbc0;
```

## JDBCカタログを削除する

JDBCカタログを削除するには、[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)を使用できます。

次の例では、`jdbc0`という名前のJDBCカタログを削除しています。

```SQL
DROP Catalog jdbc0;
```

## JDBCカタログ内のテーブルをクエリする

1. JDBC互換のクラスタ内のデータベースを表示するには、[SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を使用します。

   ```SQL
   SHOW DATABASES <catalog_name>;
   ```

2. 現在のセッションで宛先カタログに切り替えるには、[SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用します。

    ```SQL
    SET CATALOG <catalog_name>;
    ```

    次に、[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して現在のセッションでアクティブなデータベースを指定します。

    ```SQL
    USE <db_name>;
    ```

    または、[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して、宛先カタログ内のアクティブなデータベースを直接指定できます。

    ```SQL
    USE <catalog_name>.<db_name>;
    ```

3. 指定したデータベース内のテーブルをクエリするには、[SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用します。

   ```SQL
   SELECT * FROM <table_name>;
   ```

## よくある質問

「Malformed database URL, failed to parse the main URL sections」というエラーが示された場合はどうすればよいですか？

そのようなエラーが発生した場合、`jdbc_uri`に渡したURIが無効です。渡すURIを確認し、有効なものであることを確認してください。詳細については、このトピックの「[PROPERTIES](#properties)」セクション内のパラメータの説明を参照してください。