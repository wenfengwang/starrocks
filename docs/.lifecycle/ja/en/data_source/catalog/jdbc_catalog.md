---
displayed_sidebar: English
---

# JDBC カタログ

StarRocksは、v3.0以降のJDBCカタログをサポートしています。

JDBCカタログは、JDBCを介してアクセスされるデータソースからデータを取り込むことなくクエリできる外部カタログの一種です。

また、JDBCカタログに基づいて[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)を使用して、JDBCデータソースからデータを直接変換およびロードすることもできます。

JDBCカタログは現在、MySQLとPostgreSQLをサポートしています。

## 前提条件

- StarRocksクラスター内のFEとBEは、`driver_url`パラメーターで指定されたダウンロードURLからJDBCドライバーをダウンロードできる必要があります。
- 各BEノード上の**$BE_HOME/bin/start_be.sh**ファイル内の`JAVA_HOME`は、JRE環境のパスではなく、JDK環境のパスとして適切に設定されている必要があります。例えば、`export JAVA_HOME=<JDK_absolute_path>`と設定できます。この設定をスクリプトの先頭に追加し、BEを再起動して設定を有効にする必要があります。

## JDBCカタログの作成

### 構文

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES ("key"="value", ...)
```

### パラメーター

#### `catalog_name`

JDBCカタログの名前。命名規則は次のとおりです：

- 名前には、文字、数字（0-9）、およびアンダースコア（_）を含めることができます。文字で始まる必要があります。
- 名前は大文字と小文字が区別され、長さは1023文字を超えることはできません。

#### `comment`

JDBCカタログの説明。このパラメーターはオプションです。

#### `PROPERTIES`

JDBCカタログのプロパティ。`PROPERTIES`には以下のパラメーターが含まれる必要があります：

| **パラメーター** | **説明**                                                     |
| ----------------- | ------------------------------------------------------------ |
| type              | リソースのタイプ。値を`jdbc`に設定します。                  |
| user              | ターゲットデータベースへの接続に使用されるユーザー名。     |
| password          | ターゲットデータベースへの接続に使用されるパスワード。     |
| jdbc_uri          | JDBCドライバーがターゲットデータベースへの接続に使用するURI。MySQLの場合、URIは`"jdbc:mysql://ip:port"`の形式です。PostgreSQLの場合、URIは`"jdbc:postgresql://ip:port/db_name"`の形式です。詳細については、[PostgreSQL](https://jdbc.postgresql.org/documentation/head/connect.html)を参照してください。 |
| driver_url        | JDBCドライバーJARパッケージのダウンロードURL。HTTP URLまたはファイルURLがサポートされています。例えば、`https://repo1.maven.org/maven2/org/postgresql/postgresql/42.3.3/postgresql-42.3.3.jar`や`file:///home/disk1/postgresql-42.3.3.jar`です。<br />**注記**<br />JDBCドライバーをFEノードとBEノードの任意の同じパスに配置し、`driver_url`をそのパスに設定することもできます。その場合、パスは`file:///<path>/to/the/driver`の形式である必要があります。 |
| driver_class      | JDBCドライバーのクラス名。一般的なデータベースエンジンのJDBCドライバークラス名は以下の通りです：<ul><li>MySQL: `com.mysql.jdbc.Driver`（MySQL v5.x以前）および`com.mysql.cj.jdbc.Driver`（MySQL v6.x以降）</li><li>PostgreSQL: `org.postgresql.Driver`</li></ul> |

> **注記**
>
> FEはJDBCカタログ作成時にJDBCドライバーJARパッケージをダウンロードし、BEは最初のクエリ時にJDBCドライバーJARパッケージをダウンロードします。ダウンロードにかかる時間はネットワークの状況によって異なります。

### 例

次の例では、`jdbc0`と`jdbc1`という2つのJDBCカタログを作成します。

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

[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を使用して、現在のStarRocksクラスター内のすべてのカタログを照会できます。

```SQL
SHOW CATALOGS;
```

また、[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)を使用して、外部カタログの作成ステートメントを照会することもできます。次の例では、`jdbc0`という名前のJDBCカタログの作成ステートメントを照会します。

```SQL
SHOW CREATE CATALOG jdbc0;
```

## JDBCカタログの削除

[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)を使用して、JDBCカタログを削除できます。

次の例では、`jdbc0`という名前のJDBCカタログを削除します。

```SQL
DROP CATALOG jdbc0;
```

## JDBCカタログ内のテーブルを照会する

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を使用して、JDBC互換クラスター内のデータベースを表示します。

   ```SQL
   SHOW DATABASES <catalog_name>;
   ```

2. [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用して、現在のセッションで宛先カタログに切り替えます。

    ```SQL
    SET CATALOG <catalog_name>;
    ```

    次に、[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して、現在のセッションでアクティブなデータベースを指定します。

    ```SQL
    USE <db_name>;
    ```

    または、[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して、宛先カタログ内のアクティブなデータベースを直接指定することもできます。

    ```SQL
    USE <catalog_name>.<db_name>;
    ```

3. [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用して、指定したデータベース内の目的のテーブルをクエリします。

   ```SQL
   SELECT * FROM <table_name>;
   ```

## FAQ

「データベースURLが不正です。メインURLセクションの解析に失敗しました」というエラーが表示された場合はどうすればよいですか？

このようなエラーが発生した場合、`jdbc_uri`に渡されたURIが無効です。渡されたURIを確認し、有効であることを確認してください。詳細については、このトピックの「[PROPERTIES](#properties)」セクションのパラメーター説明を参照してください。
