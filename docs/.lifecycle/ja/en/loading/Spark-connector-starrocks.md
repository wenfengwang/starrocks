---
displayed_sidebar: English
---

# Spark コネクタを使用してデータを読み込む（推奨）

StarRocksは、Sparkを使用してStarRocksテーブルにデータをロードするための、StarRocks Connector for Apache Spark™（略してSparkコネクタ）という独自開発のコネクタを提供しています。基本原則は、データを蓄積してから、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を通じて一度にStarRocksにロードすることです。SparkコネクタはSpark DataSource V2に基づいて実装されており、Spark DataFramesまたはSpark SQLを使用してデータソースを作成できます。バッチモードと構造化ストリーミングモードの両方がサポートされています。

> **注意**
>
> StarRocksテーブルにデータをロードするには、そのテーブルに対するSELECTとINSERTの権限が必要です。ユーザーにこれらの権限を付与するには、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)の指示に従ってください。

## バージョン要件

| Spark コネクタ | Spark            | StarRocks     | Java | Scala |
| --------------- | ---------------- | ------------- | ---- | ----- |
| 1.1.1           | 3.2、3.3、または 3.4 | 2.5 以降      | 8    | 2.12  |
| 1.1.0           | 3.2、3.3、または 3.4 | 2.5 以降      | 8    | 2.12  |

> **注意**
>
> - Sparkコネクタの異なるバージョン間の動作変更については、「[Sparkコネクタのアップグレード](#upgrade-spark-connector)」を参照してください。
> - Sparkコネクタはバージョン1.1.1以降、MySQL JDBCドライバーを提供していません。ドライバーはSparkクラスパスに手動でインポートする必要があります。ドライバーは[MySQLのサイト](https://dev.mysql.com/downloads/connector/j/)または[Maven Central](https://repo1.maven.org/maven2/mysql/mysql-connector-java/)で見つけることができます。

## Spark コネクタの取得

SparkコネクタのJARファイルは、以下の方法で入手できます：

- コンパイル済みのSparkコネクタJARファイルを直接ダウンロードします。
- MavenプロジェクトでSparkコネクタを依存関係に追加し、JARファイルをダウンロードします。
- Sparkコネクタのソースコードを自分でJARファイルにコンパイルします。

SparkコネクタJARファイルの命名形式は`starrocks-spark-connector-${spark_version}_${scala_version}-${connector_version}.jar`です。

例えば、環境にSpark 3.2とScala 2.12をインストールしており、Sparkコネクタ1.1.0を使用したい場合は、`starrocks-spark-connector-3.2_2.12-1.1.0.jar`を使用できます。

> **注意**
>
> 一般に、Sparkコネクタの最新バージョンは、Sparkの最新3バージョンのみと互換性を保持します。

### コンパイル済みJarファイルのダウンロード

[Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks)から対応するバージョンのSparkコネクタJARを直接ダウンロードします。

### Maven依存関係

1. Mavenプロジェクトの`pom.xml`ファイルに、以下の形式でSparkコネクタを依存関係として追加します。`spark_version`、`scala_version`、`connector_version`をそれぞれのバージョンに置き換えてください。

    ```xml
    <dependency>
    <groupId>com.starrocks</groupId>
    <artifactId>starrocks-spark-connector-${spark_version}_${scala_version}</artifactId>
    <version>${connector_version}</version>
    </dependency>
    ```

2. 例えば、環境のSparkバージョンが3.2で、Scalaバージョンが2.12で、Sparkコネクタ1.1.0を選択した場合、以下の依存関係を追加する必要があります。

    ```xml
    <dependency>
    <groupId>com.starrocks</groupId>
    <artifactId>starrocks-spark-connector-3.2_2.12</artifactId>
    <version>1.1.0</version>
    </dependency>
    ```

### 自分でコンパイル

1. [Sparkコネクタパッケージ](https://github.com/StarRocks/starrocks-connector-for-apache-spark)をダウンロードします。
2. 以下のコマンドを実行して、SparkコネクタのソースコードをJARファイルにコンパイルします。`spark_version`は対応するSparkバージョンに置き換えてください。

      ```bash
      sh build.sh <spark_version>
      ```

   例えば、環境のSparkバージョンが3.2の場合、以下のコマンドを実行します。

      ```bash
      sh build.sh 3.2
      ```

3. `target/`ディレクトリに移動して、コンパイル時に生成されたSparkコネクタJARファイル（例：`starrocks-spark-connector-3.2_2.12-1.1.0-SNAPSHOT.jar`）を探します。

> **注記**
>
> 正式にリリースされていないSparkコネクタの名前には`SNAPSHOT`接尾辞が含まれます。

## パラメータ

| パラメータ                                      | 必須 | 既定値 | 説明                                                  |
| ---------------------------------------------- | -------- | ------------- | ------------------------------------------------------------ |
| starrocks.fe.http.url                          | はい      | なし          | StarRocksクラスタ内のFEのHTTP URL。複数のURLを指定する場合はコンマ（,）で区切ります。形式：`<fe_host1>:<fe_http_port1>,<fe_host2>:<fe_http_port2>`。バージョン1.1.1以降、URLの前に`http://`プレフィックスを追加できます。例：`http://<fe_host1>:<fe_http_port1>,http://<fe_host2>:<fe_http_port2>`。|
| starrocks.fe.jdbc.url                          | はい      | なし          | FEのMySQLサーバーに接続するためのアドレス。形式：`jdbc:mysql://<fe_host>:<fe_query_port>`。 |
| starrocks.table.identifier                     | はい      | なし          | StarRocksテーブルの名前。形式：`<database_name>.<table_name>`。 |
| starrocks.user                                 | はい      | なし          | StarRocksクラスタアカウントのユーザー名。ユーザーはStarRocksテーブルに対する[SELECTおよびINSERT権限](../sql-reference/sql-statements/account-management/GRANT.md)が必要です。             |
| starrocks.password                             | はい      | なし          | StarRocksクラスタアカウントのパスワード。              |
| starrocks.write.label.prefix                   | いいえ       | spark-        | Stream Loadに使用されるラベルのプレフィックス。                        |
| starrocks.write.enable.transaction-stream-load | いいえ | TRUE | [Stream Loadトランザクションインターフェース](../loading/Stream_Load_transaction_interface.md)を使用してデータをロードするかどうか。StarRocks v2.5以降が必要です。この機能は、より少ないメモリ使用量でトランザクションにより多くのデータをロードし、パフォーマンスを向上させます。<br/> **注意:** 1.1.1以降、このパラメータは`starrocks.write.max.retries`の値が非正の場合にのみ有効です。Stream Loadトランザクションインターフェースは再試行をサポートしていないためです。 |
| starrocks.write.buffer.size                    | いいえ       | 104857600     | 一度にStarRocksに送信される前にメモリに蓄積できるデータの最大サイズ。このパラメータを大きく設定すると、ロード性能は向上しますが、ロード待ち時間が増加する可能性があります。 |
| starrocks.write.buffer.rows                    | いいえ | Integer.MAX_VALUE | バージョン1.1.1以降でサポートされています。一度にStarRocksに送信される前にメモリに蓄積できる最大行数。 |
| starrocks.write.flush.interval.ms              | いいえ       | 300000        | StarRocksにデータが送信される間隔。このパラメータはロードレイテンシーを制御するために使用されます。 |
| starrocks.write.max.retries                    | いいえ       | 3             | バージョン1.1.1以降でサポートされています。ロードが失敗した場合に、コネクタが同じデータバッチのStream Loadを再試行する回数。<br/> **注意:** Stream Loadトランザクションインターフェースは再試行をサポートしていないため、このパラメータが正の値の場合、コネクタは常にStream Loadインターフェースを使用し、`starrocks.write.enable.transaction-stream-load`の値は無視されます。|

| starrocks.write.retry.interval.ms              | NO       | 10000         | バージョン1.1.1以降でサポートされています。ロードが失敗した場合に、同じデータバッチのストリームロードを再試行する間隔です。|
| starrocks.columns                              | NO       | None          | StarRocksテーブルにロードしたい列。複数の列を指定することができ、コンマ(,)で区切る必要があります。例えば、`"col0,col1,col2"`のように指定します。 |
| starrocks.column.types                         | NO       | None          | バージョン1.1.1以降でサポートされています。StarRocksテーブルから推測されるデフォルトの型ではなく、Sparkで列のデータ型をカスタマイズします。パラメータ値は、Sparkの[StructType#toDDL](https://github.com/apache/spark/blob/master/sql/api/src/main/scala/org/apache/spark/sql/types/StructType.scala#L449)の出力と同じDDL形式のスキーマです。例：`col0 INT, col1 STRING, col2 BIGINT`。カスタマイズが必要な列のみを指定する必要があります。一つの使用例は、[BITMAP](#load-data-into-columns-of-bitmap-type)型または[HLL](#load-data-into-columns-of-hll-type)型の列にデータをロードする場合です。|
| starrocks.write.properties.*                   | NO       | None          | ストリームロードの動作を制御するために使用されるパラメータ。例えば、`starrocks.write.properties.format`パラメータは、ロードするデータの形式を指定します。CSVやJSONなどがあります。サポートされるパラメータとその説明のリストについては、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。 |
| starrocks.write.properties.format              | NO       | CSV           | SparkコネクタがデータをStarRocksに送信する前に各データバッチを変換するためのファイル形式。有効な値はCSVとJSONです。 |
| starrocks.write.properties.row_delimiter       | NO       | \n            | CSV形式のデータの行区切り文字です。                    |
| starrocks.write.properties.column_separator    | NO       | \t            | CSV形式のデータの列区切り文字です。                 |
| starrocks.write.num.partitions                 | NO       | None          | Sparkが並行してデータを書き込むことができるパーティションの数です。データ量が少ない場合は、ロードの並行性と頻度を下げるためにパーティション数を減らすことができます。このパラメータのデフォルト値はSparkによって決定されますが、Spark Shuffleのコストがかかる可能性があります。 |
| starrocks.write.partition.columns              | NO       | None          | Sparkでのパーティショニング列です。このパラメータは`starrocks.write.num.partitions`が指定されている場合のみ有効です。指定されていない場合は、書き込まれるすべての列がパーティショニングに使用されます。 |
| starrocks.timezone                             | NO       | JVMのデフォルトタイムゾーン | バージョン1.1.1以降でサポートされています。Sparkの`TimestampType`をStarRocksの`DATETIME`に変換するために使用されるタイムゾーンです。デフォルトは`ZoneId#systemDefault()`によって返されるJVMのタイムゾーンです。形式は`Asia/Shanghai`のようなタイムゾーン名、または`+08:00`のようなゾーンオフセットが可能です。 |

## SparkとStarRocksの間のデータ型マッピング

- デフォルトのデータ型マッピングは以下の通りです：

  | Sparkデータ型 | StarRocksデータ型                                          |
  | --------------- | ------------------------------------------------------------ |
  | BooleanType     | BOOLEAN                                                      |
  | ByteType        | TINYINT                                                      |
  | ShortType       | SMALLINT                                                     |
  | IntegerType     | INT                                                          |
  | LongType        | BIGINT                                                       |
  | StringType      | LARGEINT                                                     |
  | FloatType       | FLOAT                                                        |
  | DoubleType      | DOUBLE                                                       |
  | DecimalType     | DECIMAL                                                      |
  | StringType      | CHAR                                                         |
  | StringType      | VARCHAR                                                      |
  | StringType      | STRING                                                       |
  | DateType        | DATE                                                         |
  | TimestampType   | DATETIME                                                     |
  | ArrayType       | ARRAY <br /> **注:** <br /> **バージョン1.1.1以降でサポートされています**。詳細な手順については、[ARRAY型の列にデータをロードする](#load-data-into-columns-of-array-type)を参照してください。 |

- データ型のマッピングをカスタマイズすることもできます。

  例えば、StarRocksテーブルにBITMAPとHLLの列が含まれていますが、Sparkはこれら2つのデータ型をサポートしていません。Sparkで対応するデータ型をカスタマイズする必要があります。詳細な手順については、[BITMAP](#load-data-into-columns-of-bitmap-type)列と[HLL](#load-data-into-columns-of-hll-type)列にデータをロードするを参照してください。**BITMAPとHLLはバージョン1.1.1以降でサポートされています**。

## Sparkコネクタのアップグレード

### バージョン1.1.0から1.1.1へのアップグレード

- バージョン1.1.1以降、Sparkコネクタは`mysql-connector-java`を提供しません。これは`mysql-connector-java`が使用するGPLライセンスの制限によるものです。しかし、SparkコネクタはStarRocksのテーブルメタデータに接続するためにMySQL JDBCドライバが必要ですので、ドライバをSparkのクラスパスに手動で追加する必要があります。ドライバは[MySQLのサイト](https://dev.mysql.com/downloads/connector/j/)または[Maven Central](https://repo1.maven.org/maven2/mysql/mysql-connector-java/)で見つけることができます。
- バージョン1.1.1以降、コネクタはデフォルトでStream Loadインターフェースを使用し、バージョン1.1.0のStream Loadトランザクションインターフェースを使用しません。Stream Loadトランザクションインターフェースを引き続き使用したい場合は、`starrocks.write.max.retries`を`0`に設定してください。`starrocks.write.enable.transaction-stream-load`と`starrocks.write.max.retries`の詳細については、それぞれの説明を参照してください。

## 例

以下の例は、Spark DataFramesまたはSpark SQLを使用してStarRocksテーブルにデータをロードする方法を示しています。Spark DataFramesはバッチモードと構造化ストリーミングモードの両方をサポートしています。

より多くの例については、[Sparkコネクタの例](https://github.com/StarRocks/starrocks-connector-for-apache-spark/tree/main/src/test/java/com/starrocks/connector/spark/examples)を参照してください。

### 準備

#### StarRocksテーブルを作成する

`test`データベースを作成し、`score_board`というプライマリキーテーブルを作成します。

```sql
CREATE DATABASE `test`;

CREATE TABLE `test`.`score_board`
(
    `id` int(11) NOT NULL COMMENT "",
    `name` varchar(65533) NULL DEFAULT "" COMMENT "",
    `score` int(11) NOT NULL DEFAULT "0" COMMENT ""
)
ENGINE=OLAP
PRIMARY KEY(`id`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`id`);
```

#### Spark環境を設定する

以下の例はSpark 3.2.4で実行され、`spark-shell`、`pyspark`、`spark-sql`を使用します。例を実行する前に、SparkコネクタのJARファイルを`$SPARK_HOME/jars`ディレクトリに配置してください。

### Spark DataFramesを使用してデータをロードする

次の2つの例は、Spark DataFramesのバッチモードまたは構造化ストリーミングモードでデータをロードする方法を説明します。

#### バッチ

メモリ内のデータを構築し、StarRocksテーブルにデータをロードします。

1. SparkアプリケーションはScalaまたはPythonで記述できます。

  Scalaを使用する場合は、`spark-shell`で以下のコードスニペットを実行します：

  ```Scala
  // 1. Create a DataFrame from a sequence.
  val data = Seq((1, "starrocks", 100), (2, "spark", 100))
  val df = data.toDF("id", "name", "score")

  // 2. Write to StarRocks by configuring the format as "starrocks" and the following options. 
  // You need to modify the options according to your own environment.
  df.write.format("starrocks")
      .option("starrocks.fe.http.url", "127.0.0.1:8030")
      .option("starrocks.fe.jdbc.url", "jdbc:mysql://127.0.0.1:9030")
      .option("starrocks.table.identifier", "test.score_board")
      .option("starrocks.user", "root")
      .option("starrocks.password", "")
      .mode("append")
      .save()
  ```

  Pythonを使用する場合、`pyspark`で次のコードスニペットを実行します。

   ```python
   from pyspark.sql import SparkSession
   
   spark = SparkSession \
        .builder \
        .appName("StarRocks Example") \
        .getOrCreate()
   
    # 1. シーケンスからDataFrameを作成します。
    data = [(1, "starrocks", 100), (2, "spark", 100)]
    df = spark.sparkContext.parallelize(data) \
            .toDF(["id", "name", "score"])

    # 2. "starrocks"フォーマットを指定し、以下のオプションを設定してStarRocksに書き込みます。
    # 環境に応じてオプションを変更する必要があります。
    df.write.format("starrocks") \
        .option("starrocks.fe.http.url", "127.0.0.1:8030") \
        .option("starrocks.fe.jdbc.url", "jdbc:mysql://127.0.0.1:9030") \
        .option("starrocks.table.identifier", "test.score_board") \
        .option("starrocks.user", "root") \
        .option("starrocks.password", "") \
        .mode("append") \
        .save()
    ```

2. StarRocksテーブル内のデータをクエリします。

    ```sql
    MySQL [test]> SELECT * FROM `score_board`;
    +------+-----------+-------+
    | id   | name      | score |
    +------+-----------+-------+
    |    1 | starrocks |   100 |
    |    2 | spark     |   100 |
    +------+-----------+-------+
    2 rows in set (0.00 sec)
    ```

#### 構造化ストリーミング

CSVファイルからデータのストリーミング読み取りを構築し、StarRocksテーブルにデータをロードします。

1. `csv-data`ディレクトリに、次のデータを含むCSVファイル`test.csv`を作成します。

    ```csv
    3,starrocks,100
    4,spark,100
    ```

2. SparkアプリケーションはScalaまたはPythonを使用して記述できます。

  Scalaを使用する場合、`spark-shell`で次のコードスニペットを実行します。

    ```scala
    import org.apache.spark.sql.types.StructType

    // 1. CSVからDataFrameを作成します。
    val schema = new StructType()
            .add("id", "integer")
            .add("name", "string")
            .add("score", "integer")
    val df = spark.readStream
            .option("sep", ",")
            .schema(schema)
            .format("csv") 
            // "csv-data"ディレクトリへのパスに置き換えてください。
            .load("/path/to/csv-data")
    
    // 2. "starrocks"フォーマットを指定し、以下のオプションを設定してStarRocksに書き込みます。
    // 環境に応じてオプションを変更する必要があります。
    val query = df.writeStream.format("starrocks")
            .option("starrocks.fe.http.url", "127.0.0.1:8030")
            .option("starrocks.fe.jdbc.url", "jdbc:mysql://127.0.0.1:9030")
            .option("starrocks.table.identifier", "test.score_board")
            .option("starrocks.user", "root")
            .option("starrocks.password", "")
            // チェックポイントディレクトリに置き換えてください
            .option("checkpointLocation", "/path/to/checkpoint")
            .outputMode("append")
            .start()
    ```

  Pythonを使用する場合、`pyspark`で次のコードスニペットを実行します。
   
   ```python
   from pyspark.sql import SparkSession
   from pyspark.sql.types import IntegerType, StringType, StructType, StructField
   
   spark = SparkSession \
        .builder \
        .appName("StarRocks SS Example") \
        .getOrCreate()
   
    # 1. CSVからDataFrameを作成します。
    schema = StructType([
            StructField("id", IntegerType()),
            StructField("name", StringType()),
            StructField("score", IntegerType())
        ])
    df = spark.readStream
        .option("sep", ",")
        .schema(schema)
        .format("csv")
        # "csv-data"ディレクトリへのパスに置き換えてください。
        .load("/path/to/csv-data")
    
    # 2. "starrocks"フォーマットを指定し、以下のオプションを設定してStarRocksに書き込みます。
    # 環境に応じてオプションを変更する必要があります。
    query = df.writeStream.format("starrocks")
        .option("starrocks.fe.http.url", "127.0.0.1:8030")
        .option("starrocks.fe.jdbc.url", "jdbc:mysql://127.0.0.1:9030")
        .option("starrocks.table.identifier", "test.score_board")
        .option("starrocks.user", "root")
        .option("starrocks.password", "")
        # チェックポイントディレクトリに置き換えてください
        .option("checkpointLocation", "/path/to/checkpoint")
        .outputMode("append")
        .start()
    ```

3. StarRocksテーブル内のデータをクエリします。

    ```sql
    MySQL [test]> select * from score_board;
    +------+-----------+-------+
    | id   | name      | score |
    +------+-----------+-------+
    |    4 | spark     |   100 |
    |    3 | starrocks |   100 |
    +------+-----------+-------+
    2 rows in set (0.67 sec)
    ```

### Spark SQLを使用したデータのロード

次の例では、[Spark SQL CLI](https://spark.apache.org/docs/latest/sql-distributed-sql-engine-spark-sql-cli.html)の`INSERT INTO`ステートメントを使用してSpark SQLでデータをロードする方法について説明します。

1. `spark-sql`で次のSQLステートメントを実行します。

    ```sql
    -- 1. データソースを`starrocks`として、以下のオプションを設定してテーブルを作成します。
    -- 環境に応じてオプションを変更する必要があります。
    CREATE TABLE `score_board`
    USING starrocks
    OPTIONS(
    "starrocks.fe.http.url"="127.0.0.1:8030",
    "starrocks.fe.jdbc.url"="jdbc:mysql://127.0.0.1:9030",
    "starrocks.table.identifier"="test.score_board",
    "starrocks.user"="root",
    "starrocks.password"=""
    );

    -- 2. テーブルに2行を挿入します。
    INSERT INTO `score_board` VALUES (5, "starrocks", 100), (6, "spark", 100);
    ```

2. StarRocksテーブル内のデータをクエリします。

    ```sql
    MySQL [test]> select * from score_board;
    +------+-----------+-------+
    | id   | name      | score |
    +------+-----------+-------+
    |    6 | spark     |   100 |
    |    5 | starrocks |   100 |
    +------+-----------+-------+
    2 rows in set (0.00 sec)
    ```

## ベストプラクティス

### 主キーテーブルへのデータロード

このセクションでは、StarRocksの主キーテーブルにデータをロードして、部分更新および条件付き更新を実現する方法について説明します。
これらの機能の詳細な紹介については、[ロードによるデータの変更](../loading/Load_to_Primary_Key_tables.md)を参照してください。
これらの例ではSpark SQLを使用します。

#### 準備

`test`データベースを作成し、StarRocksに主キーテーブル`score_board`を作成します。

```sql
CREATE DATABASE `test`;

CREATE TABLE `test`.`score_board`
(
    `id` int(11) NOT NULL COMMENT "",
    `name` varchar(65533) NULL DEFAULT "" COMMENT "",
    `score` int(11) NOT NULL DEFAULT "0" COMMENT ""
)
ENGINE=OLAP
PRIMARY KEY(`id`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`id`) PARTITIONS 10;
```

#### 部分更新

この例では、ロードによって列`name`のデータのみを更新する方法を示します。

1. MySQLクライアントでStarRocksテーブルに初期データを挿入します。

   ```sql
   mysql> INSERT INTO `score_board` VALUES (1, 'starrocks', 100), (2, 'spark', 100);

   mysql> select * from score_board;
   +------+-----------+-------+
   | id   | name      | score |
   +------+-----------+-------+
   |    1 | starrocks |   100 |
   |    2 | spark     |   100 |
   +------+-----------+-------+
   2 rows in set (0.02 sec)
   ```

2. Spark SQLクライアントでSparkテーブル`score_board`を作成します。

   - コネクタに部分更新を行うよう指示するために、オプション`starrocks.write.properties.partial_update`を`true`に設定します。
   - 書き込む列をコネクタに指示するために、オプション`starrocks.columns`を`"id,name"`に設定します。

   ```sql
   CREATE TABLE `score_board`
   USING starrocks
   OPTIONS(
       "starrocks.fe.http.url"="127.0.0.1:8030",
       "starrocks.fe.jdbc.url"="jdbc:mysql://127.0.0.1:9030",
       "starrocks.table.identifier"="test.score_board",
       "starrocks.user"="root",
       "starrocks.password"="",
       "starrocks.write.properties.partial_update"="true",
       "starrocks.columns"="id,name"
    );
   ```

3. Spark SQLクライアントでテーブルにデータを挿入し、列`name`のみを更新します。

   ```sql
   INSERT INTO `score_board` VALUES (1, 'starrocks-update'), (2, 'spark-update');
   ```

4. MySQLクライアントでStarRocksテーブルをクエリします。

   `name`の値のみが変更され、`score`の値は変更されていないことがわかります。

   ```sql
   mysql> select * from score_board;
   +------+------------------+-------+
   | id   | name             | score |
   +------+------------------+-------+
   |    1 | starrocks-update |   100 |
   |    2 | spark-update     |   100 |
   +------+------------------+-------+
   2 rows in set (0.02 sec)
   ```

#### 条件付き更新

この例では、列`score`の値に基づいて条件付き更新を行う方法を示します。新しい`score`の値が古い値以上の場合にのみ、`id`の更新が有効になります。

1. MySQLクライアントでStarRocksテーブルに初期データを挿入します。

    ```sql
    mysql> INSERT INTO `score_board` VALUES (1, 'starrocks', 100), (2, 'spark', 100);

    mysql> select * from score_board;
    +------+-----------+-------+
    | id   | name      | score |
    +------+-----------+-------+
    |    1 | starrocks |   100 |
    |    2 | spark     |   100 |
    +------+-----------+-------+
    2 rows in set (0.02 sec)
    ```

2. Spark SQLクライアントでSparkテーブル`score_board`を作成します。

   - コネクタに列`score`を条件として使用するよう指示するために、オプション`starrocks.write.properties.merge_condition`を`"score"`に設定します。

```sql
CREATE TABLE `score_board`
USING starrocks
OPTIONS(
    "starrocks.fe.http.url"="127.0.0.1:8030",
    "starrocks.fe.jdbc.url"="jdbc:mysql://127.0.0.1:9030",
    "starrocks.table.identifier"="test.score_board",
    "starrocks.user"="root",
    "starrocks.password"="",
    "starrocks.write.properties.merge_condition"="score >= VALUES(score)"
);
```
   - Spark コネクタがデータをロードする際には、Stream Load トランザクション インターフェースではなく Stream Load インターフェースを使用してください。後者はこの機能をサポートしていません。

   ```SQL
   CREATE TABLE `score_board`
   USING starrocks
   OPTIONS(
       "starrocks.fe.http.url"="127.0.0.1:8030",
       "starrocks.fe.jdbc.url"="jdbc:mysql://127.0.0.1:9030",
       "starrocks.table.identifier"="test.score_board",
       "starrocks.user"="root",
       "starrocks.password"="",
       "starrocks.write.properties.merge_condition"="score"
    );
   ```

3. Spark SQL クライアントでテーブルにデータを挿入し、`id` が 1 の行をより小さいスコア値で、`id` が 2 の行をより大きなスコア値で更新します。

   ```SQL
   INSERT INTO `score_board` VALUES (1, 'starrocks-update', 99), (2, 'spark-update', 101);
   ```

4. MySQL クライアントで StarRocks テーブルをクエリします。

   `id` が 2 の行のみが変更され、`id` が 1 の行は変更されないことが確認できます。

   ```SQL
   mysql> select * from score_board;
   +------+--------------+-------+
   | id   | name         | score |
   +------+--------------+-------+
   |    1 | starrocks    |   100 |
   |    2 | spark-update |   101 |
   +------+--------------+-------+
   2 rows in set (0.03 sec)
   ```

### BITMAP 型のカラムにデータをロードする

[`BITMAP`](../sql-reference/sql-statements/data-types/BITMAP.md) は、例えば UV のカウントなど、カウントディスティンクトを高速化するためによく使用されます。詳細は [ビットマップを使用した正確なカウントディスティンクト](../using_starrocks/Using_bitmap.md) を参照してください。
ここでは UV のカウントを例に、`BITMAP` 型のカラムにデータをロードする方法を紹介します。**`BITMAP` はバージョン 1.1.1 以降でサポートされています**。

1. StarRocks 集約テーブルを作成します。

   `test` データベースに、カラム `visit_users` が `BITMAP` 型で、集約関数 `BITMAP_UNION` を使用する集約テーブル `page_uv` を作成します。

    ```SQL
    CREATE TABLE `test`.`page_uv` (
      `page_id` INT NOT NULL COMMENT 'ページ ID',
      `visit_date` datetime NOT NULL COMMENT '訪問日時',
      `visit_users` BITMAP BITMAP_UNION NOT NULL COMMENT 'ユーザー ID'
    ) ENGINE=OLAP
    AGGREGATE KEY(`page_id`, `visit_date`)
    DISTRIBUTED BY HASH(`page_id`);
    ```

2. Spark テーブルを作成します。

   Spark テーブルのスキーマは StarRocks テーブルから推測されますが、Spark は `BITMAP` 型をサポートしていません。そのため、オプション `"starrocks.column.types"="visit_users BIGINT"` を設定して、Spark で対応するカラムのデータ型を `BIGINT` などにカスタマイズする必要があります。Stream Load を使用してデータを取り込む際、コネクタは [`to_bitmap`](../sql-reference/sql-functions/bitmap-functions/to_bitmap.md) 関数を使用して `BIGINT` 型のデータを `BITMAP` 型に変換します。

    `spark-sql` で以下の DDL を実行します。

    ```SQL
    CREATE TABLE `page_uv`
    USING starrocks
    OPTIONS(
       "starrocks.fe.http.url"="127.0.0.1:8030",
       "starrocks.fe.jdbc.url"="jdbc:mysql://127.0.0.1:9030",
       "starrocks.table.identifier"="test.page_uv",
       "starrocks.user"="root",
       "starrocks.password"="",
       "starrocks.column.types"="visit_users BIGINT"
    );
    ```

3. StarRocks テーブルにデータをロードします。

    `spark-sql` で以下の DML を実行します。

    ```SQL
    INSERT INTO `page_uv` VALUES
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 13),
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 23),
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 33),
       (1, CAST('2020-06-23 02:30:30' AS TIMESTAMP), 13),
       (2, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 23);
    ```

4. StarRocks テーブルからページ UV を計算します。

    ```SQL
    MySQL [test]> SELECT `page_id`, COUNT(DISTINCT `visit_users`) FROM `page_uv` GROUP BY `page_id`;
    +---------+-----------------------------+
    | page_id | count(DISTINCT visit_users) |
    +---------+-----------------------------+
    |       2 |                           1 |
    |       1 |                           3 |
    +---------+-----------------------------+
    2 rows in set (0.01 sec)
    ```

> **注意：**
>
> コネクタは [`to_bitmap`](../sql-reference/sql-functions/bitmap-functions/to_bitmap.md) 関数を使用して、Spark の `TINYINT`、`SMALLINT`、`INTEGER`、`BIGINT` 型のデータを StarRocks の `BITMAP` 型に変換し、他の Spark データ型には [`bitmap_hash`](../sql-reference/sql-functions/bitmap-functions/bitmap_hash.md) 関数を使用します。

### HLL 型のカラムにデータをロードする

[`HLL`](../sql-reference/sql-statements/data-types/HLL.md) は、近似的なカウントディスティンクトに使用できます。詳細は [HLL を使用した近似カウントディスティンクト](../using_starrocks/Using_HLL.md) を参照してください。

ここでは UV のカウントを例に、`HLL` 型のカラムにデータをロードする方法を紹介します。**`HLL` はバージョン 1.1.1 以降でサポートされています**。

1. StarRocks 集約テーブルを作成します。

   `test` データベースに、カラム `visit_users` が `HLL` 型で、集約関数 `HLL_UNION` を使用する集約テーブル `hll_uv` を作成します。

    ```SQL
    CREATE TABLE `hll_uv` (
    `page_id` INT NOT NULL COMMENT 'ページ ID',
    `visit_date` datetime NOT NULL COMMENT '訪問日時',
    `visit_users` HLL HLL_UNION NOT NULL COMMENT 'ユーザー ID'
    ) ENGINE=OLAP
    AGGREGATE KEY(`page_id`, `visit_date`)
    DISTRIBUTED BY HASH(`page_id`);
    ```

2. Spark テーブルを作成します。

   Spark テーブルのスキーマは StarRocks テーブルから推測されますが、Spark は `HLL` 型をサポートしていません。そのため、オプション `"starrocks.column.types"="visit_users BIGINT"` を設定して、Spark で対応するカラムのデータ型を `BIGINT` などにカスタマイズする必要があります。Stream Load を使用してデータを取り込む際、コネクタは [`hll_hash`](../sql-reference/sql-functions/aggregate-functions/hll_hash.md) 関数を使用して `BIGINT` 型のデータを `HLL` 型に変換します。

    `spark-sql` で以下の DDL を実行します。

    ```SQL
    CREATE TABLE `hll_uv`
    USING starrocks
    OPTIONS(
       "starrocks.fe.http.url"="127.0.0.1:8030",
       "starrocks.fe.jdbc.url"="jdbc:mysql://127.0.0.1:9030",
       "starrocks.table.identifier"="test.hll_uv",
       "starrocks.user"="root",
       "starrocks.password"="",
       "starrocks.column.types"="visit_users BIGINT"
    );
    ```

3. StarRocks テーブルにデータをロードします。

    `spark-sql` で以下の DML を実行します。

    ```SQL
    INSERT INTO `hll_uv` VALUES
       (3, CAST('2023-07-24 12:00:00' AS TIMESTAMP), 78),
       (4, CAST('2023-07-24 13:20:10' AS TIMESTAMP), 2),
       (3, CAST('2023-07-24 12:30:00' AS TIMESTAMP), 674);
    ```

4. StarRocks テーブルからページ UV を計算します。

    ```SQL
    MySQL [test]> SELECT `page_id`, COUNT(DISTINCT `visit_users`) FROM `hll_uv` GROUP BY `page_id`;
    +---------+-----------------------------+
    | page_id | count(DISTINCT visit_users) |
    +---------+-----------------------------+
    |       4 |                           1 |
    |       3 |                           2 |
    +---------+-----------------------------+
    2 rows in set (0.01 sec)
    ```

### ARRAY 型のカラムにデータをロードする

以下の例では、`ARRAY` 型のカラムにデータをロードする方法について説明します。

1. StarRocks テーブルを作成します。

   `test` データベースに、`INT` 型のカラムと `ARRAY` 型のカラムを2つ含むプライマリキーテーブル `array_tbl` を作成します。

   ```SQL
   CREATE TABLE `array_tbl` (
       `id` INT NOT NULL,
       `a0` ARRAY<STRING>,
       `a1` ARRAY<ARRAY<INT>>
    )
    ENGINE=OLAP
    PRIMARY KEY(`id`)
    DISTRIBUTED BY HASH(`id`);
    ```

2. StarRocks にデータを書き込みます。

   StarRocks の一部のバージョンでは `ARRAY` 型のカラムのメタデータが提供されていないため、コネクタはこのカラムに対応する Spark データ型を推測できません。しかし、オプション `starrocks.column.types` を使用して、カラムの対応する Spark データ型を明示的に指定することができます。この例では、オプションを `a0 ARRAY<STRING>,a1 ARRAY<ARRAY<INT>>` として設定できます。

   `spark-shell` で以下のコードを実行します。

   ```scala
    val data = Seq(
       (1, Seq("hello", "starrocks"), Seq(Seq(1, 2), Seq(3, 4))),
       (2, Seq("hello", "spark"), Seq(Seq(5, 6, 7), Seq(8, 9, 10)))
    )
    val df = data.toDF("id", "a0", "a1")
    df.write
         .format("starrocks")
         .option("starrocks.fe.http.url", "127.0.0.1:8030")
         .option("starrocks.fe.jdbc.url", "jdbc:mysql://127.0.0.1:9030")
         .option("starrocks.table.identifier", "test.array_tbl")
         .option("starrocks.user", "root")
         .option("starrocks.password", "")
         .option("starrocks.column.types", "a0 ARRAY<STRING>,a1 ARRAY<ARRAY<INT>>")
         .mode("append")
         .save()
    ```

3. StarRocksテーブルでデータをクエリする。

   ```SQL
   MySQL [test]> SELECT * FROM `array_tbl`;
   +------+-----------------------+--------------------+
   | id   | a0                    | a1                 |
   +------+-----------------------+--------------------+
   |    1 | ["hello","starrocks"] | [[1,2],[3,4]]      |
   |    2 | ["hello","spark"]     | [[5,6,7],[8,9,10]] |
   +------+-----------------------+--------------------+
   2行がセットされました (0.01秒)
   ```

