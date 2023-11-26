---
displayed_sidebar: "Japanese"
---

# Sparkコネクタを使用してデータをロードする（推奨）

StarRocksでは、Apache Spark™（以下、Sparkコネクタ）用のStarRocksコネクタという独自のコネクタを提供しており、Sparkを使用してStarRocksテーブルにデータをロードするためのサポートを行っています。基本的な原則は、データを蓄積し、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を介して一度にStarRocksにロードすることです。Sparkコネクタは、Spark DataSource V2に基づいて実装されています。DataSourceは、Spark DataFramesまたはSpark SQLを使用して作成できます。バッチおよび構造化ストリーミングモードの両方がサポートされています。

> **注意**
>
> StarRocksテーブルのSELECTおよびINSERT権限を持つユーザーのみがこのテーブルにデータをロードできます。ユーザーにこれらの権限を付与する手順については、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)で提供される手順に従ってください。

## バージョン要件

| Sparkコネクタ | Spark            | StarRocks     | Java | Scala |
| --------------- | ---------------- | ------------- | ---- | ----- |
| 1.1.1 | 3.2、3.3、または3.4 | 2.5以降 | 8 | 2.12 |
| 1.1.0           | 3.2、3.3、または3.4 | 2.5以降 | 8    | 2.12  |

> **注意**
>
> - Sparkコネクタのさまざまなバージョン間の動作の変更については、[Sparkコネクタのアップグレード](#spark-connectorのアップグレード)を参照してください。
> - Sparkコネクタはバージョン1.1.1以降、MySQL JDBCドライバを提供していません。そのため、ドライバをsparkのクラスパスに手動でインポートする必要があります。ドライバは[MySQLサイト](https://dev.mysql.com/downloads/connector/j/)または[Maven Central](https://repo1.maven.org/maven2/mysql/mysql-connector-java/)で入手できます。

## Sparkコネクタの取得

SparkコネクタのJARファイルは、次の方法で取得できます。

- コンパイル済みのSparkコネクタJARファイルを直接ダウンロードします。
- Mavenプロジェクトの依存関係としてSparkコネクタを追加し、JARファイルをダウンロードします。
- Sparkコネクタのソースコードを自分でコンパイルしてJARファイルにします。

SparkコネクタJARファイルの命名形式は、`starrocks-spark-connector-${spark_version}_${scala_version}-${connector_version}.jar`です。

たとえば、環境にSpark 3.2とScala 2.12をインストールし、Sparkコネクタ1.1.0を使用したい場合、`starrocks-spark-connector-3.2_2.12-1.1.0.jar`を使用します。

> **注意**
>
> 一般的に、Sparkコネクタの最新バージョンは、最新の3つのSparkバージョンとの互換性を維持します。

### コンパイル済みのJarファイルをダウンロードする

[Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks)から、対応するバージョンのSparkコネクタJARを直接ダウンロードします。

### Mavenの依存関係

1. Mavenプロジェクトの`pom.xml`ファイルに、以下の形式に従ってSparkコネクタを依存関係として追加します。`spark_version`、`scala_version`、`connector_version`をそれぞれのバージョンに置き換えてください。

    ```xml
    <dependency>
    <groupId>com.starrocks</groupId>
    <artifactId>starrocks-spark-connector-${spark_version}_${scala_version}</artifactId>
    <version>${connector_version}</version>
    </dependency>
    ```

2. たとえば、環境のSparkのバージョンが3.2でScalaのバージョンが2.12であり、Sparkコネクタ1.1.0を選択した場合、次の依存関係を追加する必要があります。

    ```xml
    <dependency>
    <groupId>com.starrocks</groupId>
    <artifactId>starrocks-spark-connector-3.2_2.12</artifactId>
    <version>1.1.0</version>
    </dependency>
    ```

### 自分でコンパイルする

1. [Sparkコネクタパッケージ](https://github.com/StarRocks/starrocks-connector-for-apache-spark)をダウンロードします。
2. 次のコマンドを実行して、SparkコネクタのソースコードをコンパイルしてJARファイルにします。`spark_version`は対応するSparkバージョンに置き換えてください。

      ```bash
      sh build.sh <spark_version>
      ```

   たとえば、環境のSparkバージョンが3.2の場合、次のコマンドを実行します。

      ```bash
      sh build.sh 3.2
      ```

3. `target/`ディレクトリに移動して、コンパイルされたSparkコネクタJARファイル（たとえば、`starrocks-spark-connector-3.2_2.12-1.1.0-SNAPSHOT.jar`）を見つけます。

> **注意**
>
> 正式にリリースされていないSparkコネクタの名前には、`SNAPSHOT`の接尾辞が含まれています。

## パラメータ

| パラメータ                                      | 必須 | デフォルト値 | 説明                                                  |
| ---------------------------------------------- | ---- | ------------- | ------------------------------------------------------------ |
| starrocks.fe.http.url                          | YES  | None          | StarRocksクラスタのFEのHTTP URLです。複数のURLを指定することができます。URLはカンマ（,）で区切る必要があります。形式：`<fe_host1>:<fe_http_port1>,<fe_host2>:<fe_http_port2>`。バージョン1.1.1以降では、URLに`http://`の接頭辞を追加することもできます。たとえば、`http://<fe_host1>:<fe_http_port1>,http://<fe_host2>:<fe_http_port2>`のようになります。|
| starrocks.fe.jdbc.url                          | YES  | None          | FEのMySQLサーバーに接続するために使用されるアドレスです。形式：`jdbc:mysql://<fe_host>:<fe_query_port>`。 |
| starrocks.table.identifier                     | YES  | None          | StarRocksテーブルの名前です。形式：`<database_name>.<table_name>`。 |
| starrocks.user                                 | YES  | None          | StarRocksクラスタのアカウントのユーザー名です。ユーザーはStarRocksテーブルの[SELECTおよびINSERT権限](../sql-reference/sql-statements/account-management/GRANT.md)が必要です。             |
| starrocks.password                             | YES  | None          | StarRocksクラスタのアカウントのパスワードです。              |
| starrocks.write.label.prefix                   | NO   | spark-        | Stream Loadで使用されるラベルのプレフィックスです。                        |
| starrocks.write.enable.transaction-stream-load | NO | TRUE | データをロードするために[Stream Loadトランザクションインターフェース](../loading/Stream_Load_transaction_interface.md)を使用するかどうかを指定します。StarRocks v2.5以降が必要です。この機能は、トランザクション内でより多くのデータをロードし、メモリ使用量を減らし、パフォーマンスを向上させることができます。<br/> **注意:** バージョン1.1.1以降、このパラメータは、`starrocks.write.max.retries`の値が非正の場合にのみ効果があります。なぜなら、Stream Loadトランザクションインターフェースはリトライをサポートしていないためです。 |
| starrocks.write.buffer.size                    | NO   | 104857600     | 一度にStarRocksに送信される前にメモリに蓄積できるデータの最大サイズです。このパラメータを大きな値に設定すると、ロードパフォーマンスが向上する可能性がありますが、ロードの待機時間が増加する可能性があります。 |
| starrocks.write.buffer.rows | NO | Integer.MAX_VALUE | メモリに蓄積される前に一度にStarRocksに送信できる行の最大数です。バージョン1.1.1以降でサポートされています。 |
| starrocks.write.flush.interval.ms              | NO   | 300000        | データがStarRocksに送信される間隔です。このパラメータは、ロードの待機時間を制御するために使用されます。 |
| starrocks.write.max.retries                    | NO   | 3             | ロードが失敗した場合に、コネクタが同じデータのバッチに対してStream Loadを実行し続ける回数です。<br/> **注意:** Stream Loadトランザクションインターフェースはリトライをサポートしていないため、このパラメータが正の値である場合、コネクタは常にStream Loadインターフェースを使用し、`starrocks.write.enable.transaction-stream-load`の値は無視されます。|
| starrocks.write.retry.interval.ms              | NO   | 10000         | 同じデータのバッチに対してロードが失敗した場合のリトライ間隔です。バージョン1.1.1以降でサポートされています。|
| starrocks.columns                              | NO   | None          | データをロードするStarRocksテーブルの列です。複数の列を指定することができます。列はカンマ（,）で区切る必要があります。たとえば、`"col0,col1,col2"`のように指定します。 |
| starrocks.column.types                         | NO   | None          | バージョン1.1.1以降でサポートされています。Sparkのデフォルトではなく、Sparkのためのカスタムカラムデータ型を指定します。パラメータの値は、Sparkの[StructType#toDDL](https://github.com/apache/spark/blob/master/sql/api/src/main/scala/org/apache/spark/sql/types/StructType.scala#L449)と同じ形式のDDL形式のスキーマです。たとえば、`col0 INT, col1 STRING, col2 BIGINT`のような形式です。カスタマイズが必要な列のみを指定する必要があることに注意してください。ビットマップ（[BITMAPタイプの列にデータをロードする](#ビットマップタイプの列にデータをロードする)）またはHLL（[HLLタイプの列にデータをロードする](#HLLタイプの列にデータをロードする)）タイプの列にデータをロードする場合など、1つの使用例です。|
| starrocks.write.properties.*                   | NO   | None          | Stream Loadの動作を制御するために使用されるパラメータです。たとえば、パラメータ`starrocks.write.properties.format`は、ロードするデータの形式（CSVまたはJSONなど）を指定します。サポートされているパラメータとその説明のリストについては、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。 |
| starrocks.write.properties.format              | NO   | CSV           | SparkコネクタがデータをStarRocksに送信する前に、各データバッチを変換する基になるファイル形式です。有効な値：CSVおよびJSON。 |
| starrocks.write.properties.row_delimiter       | NO   | \n            | CSV形式のデータの行区切り記号です。                    |
| starrocks.write.properties.column_separator    | NO   | \t            | CSV形式のデータの列区切り記号です。                 |
| starrocks.write.num.partitions                 | NO   | None          | Sparkが並列にデータを書き込むためのパーティションの数です。データ量が少ない場合は、パーティションの数を減らしてロードの並行性と頻度を下げることができます。このパラメータのデフォルト値はSparkによって決定されます。ただし、この方法はSpark Shuffleのコストを引き起こす可能性があります。 |
| starrocks.write.partition.columns              | NO   | None          | Sparkのパーティショニング列です。パラメータは、`starrocks.write.num.partitions`が指定されている場合にのみ効果があります。このパラメータが指定されていない場合、書き込まれるすべての列がパーティショニングに使用されます。 |
| starrocks.timezone | NO | JVMのデフォルトタイムゾーン | バージョン1.1.1以降でサポートされています。Sparkの`TimestampType`をStarRocksの`DATETIME`に変換するために使用されるタイムゾーンです。デフォルトは、`ZoneId#systemDefault()`によって返されるJVMのタイムゾーンです。形式は、`Asia/Shanghai`などのタイムゾーン名、または`+08:00`などのゾーンオフセットです。 |

## SparkとStarRocksのデータ型のマッピング

- デフォルトのデータ型のマッピングは次のとおりです。

  | Sparkのデータ型 | StarRocksのデータ型                                          |
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
  | ArrayType       | ARRAY <br /> **注意:** <br /> **バージョン1.1.1以降でサポートされています**。詳細な手順については、[ARRAYタイプの列にデータをロードする](#ARRAYタイプの列にデータをロードする)を参照してください。 |

- データ型のマッピングをカスタマイズすることもできます。

  たとえば、StarRocksテーブルにはBITMAPとHLLの列が含まれていますが、Sparkはこれらのデータ型をサポートしていません。Sparkで対応するデータ型をカスタマイズする必要があります。詳細な手順については、[BITMAP](#BITMAPタイプの列にデータをロードする)および[HLL](#HLLタイプの列にデータをロードする)の列にデータをロードするを参照してください。**BITMAPとHLLはバージョン1.1.1以降でサポートされています**。

## Sparkコネクタのアップグレード

### バージョン1.1.0から1.1.1へのアップグレード

- バージョン1.1.1以降、Sparkコネクタは公式のMySQLのJDBCドライバである`mysql-connector-java`を提供していません。これは、`mysql-connector-java`が使用しているGPLライセンスの制限によるものです。
  ただし、Sparkコネクタはまだテーブルメタデータに接続するためにMySQLのJDBCドライバが必要ですので、ドライバをSparkのクラスパスに手動で追加する必要があります。ドライバは[MySQLサイト](https://dev.mysql.com/downloads/connector/j/)または[Maven Central](https://repo1.maven.org/maven2/mysql/mysql-connector-java/)で入手できます。
- バージョン1.1.1以降、コネクタはデフォルトでStream Loadインターフェースを使用し、バージョン1.1.0のStream Loadトランザクションインターフェースを使用しません。Stream Loadトランザクションインターフェースを引き続き使用する場合は、オプション`starrocks.write.max.retries`を`0`に設定します。詳細については、`starrocks.write.enable.transaction-stream-load`および`starrocks.write.max.retries`の説明を参照してください。

## 例

以下の例では、Sparkコネクタを使用してSpark DataFramesまたはSpark SQLを使用してStarRocksテーブルにデータをロードする方法を示します。Spark DataFramesはバッチおよび構造化ストリーミングモードの両方をサポートしています。

より詳細な例については、[Sparkコネクタの例](https://github.com/StarRocks/starrocks-connector-for-apache-spark/tree/main/src/test/java/com/starrocks/connector/spark/examples)を参照してください。

### 準備

#### StarRocksテーブルの作成

StarRocksでデータベース`test`を作成し、Primary Keyテーブル`score_board`を作成します。

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

#### Spark環境のセットアップ

以下の例は、Spark 3.2.4で実行され、`spark-shell`、`pyspark`、`spark-sql`を使用しています。例を実行する前に、SparkコネクタJARファイルを`$SPARK_HOME/jars`ディレクトリに配置していることを確認してください。

### Spark DataFramesを使用してデータをロードする

以下の2つの例では、Spark DataFramesのバッチモードまたは構造化ストリーミングモードでデータをロードする方法を説明します。

#### バッチ

メモリ内のデータを構築し、データをStarRocksテーブルにロードします。

1. ScalaまたはPythonを使用してSparkアプリケーションを作成できます。

  Scalaの場合、`spark-shell`で次のコードスニペットを実行します。

  ```Scala
  // 1. シーケンスからDataFrameを作成します。
  val data = Seq((1, "starrocks", 100), (2, "spark", 100))
  val df = data.toDF("id", "name", "score")

  // 2. フォーマットを「starrocks」に設定し、次のオプションを設定してStarRocksに書き込みます。環境に合わせてオプションを変更してください。
  df.write.format("starrocks")
      .option("starrocks.fe.http.url", "127.0.0.1:8030")
      .option("starrocks.fe.jdbc.url", "jdbc:mysql://127.0.0.1:9030")
      .option("starrocks.table.identifier", "test.score_board")
      .option("starrocks.user", "root")
      .option("starrocks.password", "")
      .mode("append")
      .save()
  ```

  Pythonの場合、`pyspark`で次のコードスニペットを実行します。

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

    # 2. フォーマットを「starrocks」に設定し、次のオプションを設定してStarRocksに書き込みます。環境に合わせてオプションを変更してください。
    df.write.format("starrocks") \
        .option("starrocks.fe.http.url", "127.0.0.1:8030") \
        .option("starrocks.fe.jdbc.url", "jdbc:mysql://127.0.0.1:9030") \
        .option("starrocks.table.identifier", "test.score_board") \
        .option("starrocks.user", "root") \
        .option("starrocks.password", "") \
        .mode("append") \
        .save()
    ```

2. StarRocksテーブルでデータをクエリします。

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

CSVファイルからのストリーミングリードを構築し、データをStarRocksテーブルにロードします。

1. `csv-data`ディレクトリ内に、次のデータを含むCSVファイル`test.csv`を作成します。

    ```csv
    3,starrocks,100
    4,spark,100
    ```

2. ScalaまたはPythonを使用してSparkアプリケーションを作成できます。

  Scalaの場合、`spark-shell`で次のコードスニペットを実行します。

    ```Scala
    import org.apache.spark.sql.types.StructType

    // 1. CSVからDataFrameを作成します。
    val schema = (new StructType()
            .add("id", "integer")
            .add("name", "string")
            .add("score", "integer")
        )
    val df = (spark.readStream
            .option("sep", ",")
            .schema(schema)
            .format("csv") 
            // "csv-data"ディレクトリへのパスに置き換えてください。
            .load("/path/to/csv-data")
        )
    
    // 2. フォーマットを「starrocks」に設定し、次のオプションを設定してStarRocksに書き込みます。環境に合わせてオプションを変更してください。
    val query = (df.writeStream.format("starrocks")
            .option("starrocks.fe.http.url", "127.0.0.1:8030")
            .option("starrocks.fe.jdbc.url", "jdbc:mysql://127.0.0.1:9030")
            .option("starrocks.table.identifier", "test.score_board")
            .option("starrocks.user", "root")
            .option("starrocks.password", "")
            // チェックポイントディレクトリに置き換えてください
            .option("checkpointLocation", "/path/to/checkpoint")
            .outputMode("append")
            .start()
        )
    ```

  Pythonの場合、`pyspark`で次のコードスニペットを実行します。
   
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
    df = (
        spark.readStream
        .option("sep", ",")
        .schema(schema)
        .format("csv")
        # "csv-data"ディレクトリへのパスに置き換えてください。
        .load("/path/to/csv-data")
    )

    # 2. フォーマットを「starrocks」に設定し、次のオプションを設定してStarRocksに書き込みます。環境に合わせてオプションを変更してください。
    query = (
        df.writeStream.format("starrocks")
        .option("starrocks.fe.http.url", "127.0.0.1:8030")
        .option("starrocks.fe.jdbc.url", "jdbc:mysql://127.0.0.1:9030")
        .option("starrocks.table.identifier", "test.score_board")
        .option("starrocks.user", "root")
        .option("starrocks.password", "")
        # チェックポイントディレクトリに置き換えてください
        .option("checkpointLocation", "/path/to/checkpoint")
        .outputMode("append")
        .start()
    )
    ```

3. StarRocksテーブルでデータをクエリします。

    ```SQL
    MySQL [test]> select * from score_board;
    +------+-----------+-------+
    | id   | name      | score |
    +------+-----------+-------+
    |    4 | spark     |   100 |
    |    3 | starrocks |   100 |
    +------+-----------+-------+
    2 rows in set (0.67 sec)
    ```

### Spark SQLを使用してデータをロードする

次の例では、[Spark SQL CLI](https://spark.apache.org/docs/latest/sql-distributed-sql-engine-spark-sql-cli.html)の`INSERT INTO`ステートメントを使用してSpark SQLでデータをロードする方法を説明します。

1. `spark-sql`で次のSQLステートメントを実行します。

    ```SQL
    -- 1. データソースを「starrocks」に設定し、次のオプションを設定してテーブルを作成します。環境に合わせてオプションを変更してください。
    CREATE TABLE `score_board`
    USING starrocks
    OPTIONS(
    "starrocks.fe.http.url"="127.0.0.1:8030",
    "starrocks.fe.jdbc.url"="jdbc:mysql://127.0.0.1:9030",
    "starrocks.table.identifier"="test.score_board",
    "starrocks.user"="root",
    "starrocks.password"=""
    );

    -- 2. テーブルに2つの行を挿入します。
    INSERT INTO `score_board` VALUES (5, "starrocks", 100), (6, "spark", 100);
    ```

2. StarRocksテーブルでデータをクエリします。

    ```SQL
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

### Primary Keyテーブルへのデータのロード

このセクションでは、Primary Keyテーブルへのデータのロードを通じて部分更新と条件付き更新を実現する方法を示します。
これらの機能の詳細な説明については、[ロードによるデータの変更](../loading/Load_to_Primary_Key_tables.md)を参照してください。
これらの例では、Spark SQLを使用しています。

#### 準備

StarRocksでデータベース`test`を作成し、Primary Keyテーブル`score_board`を作成します。

```SQL
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

#### 部分更新

この例では、`name`列のデータのみをロードして更新する方法を示します。

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

   - オプション`starrocks.write.properties.partial_update`を`true`に設定し、コネクタに部分更新を行うように指示します。
   - オプション`starrocks.columns`を`"id,name"`に設定し、書き込む列をコネクタに指定します。

   ```SQL
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

3. Spark SQLクライアントでテーブルにデータを挿入し、`name`列のみを更新します。

   ```SQL
   INSERT INTO `score_board` VALUES (1, 'starrocks-update'), (2, 'spark-update');
   ```

4. MySQLクライアントでStarRocksテーブルをクエリします。

   `name`の値のみが変更され、`score`の値は変更されていません。

   ```SQL
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

この例では、列`score`の値に応じて条件付きで更新する方法を示します。新しい`score`の値が古い値以上の場合にのみ、`id`に対して更新が行われます。

1. MySQLクライアントでStarRocksテーブルに初期データを挿入します。

    ```SQL
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

    - オプション`starrocks.write.properties.merge_condition`を`score`に設定し、条件として列`score`を使用するようにコネクタに指示します。
    - コネクタがStream Loadインターフェースを使用してデータをロードするようにするため、Stream Loadトランザクションインターフェースではないことを確認してください。なぜなら、後者はこの機能をサポートしていないからです。

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

3. Spark SQLクライアントでデータをテーブルに挿入し、`id`が1の行のスコア値を小さく、`id`が2の行のスコア値を大きくします。

    ```SQL
    INSERT INTO `score_board` VALUES (1, 'starrocks-update', 99), (2, 'spark-update', 101);
    ```

4. MySQLクライアントでStarRocksテーブルをクエリします。

    `id`が2の行のみが変更され、`id`が1の行は変更されていません。

    ```SQL
    mysql> select * from score_board;
    +------+--------------+-------+
    | id   | name         | score |
    +------+--------------+-------+
    |    1 | starrocks    |   100 |
    |    2 | spark-update |   101 |
    +------+--------------+-------+
   2 行がセットされました (0.03 秒)
   ```

### BITMAP タイプの列にデータをロードする

[`BITMAP`](../sql-reference/sql-statements/data-types/BITMAP.md) は、UV のカウントなどの高速化によく使用されます。詳細は [Use Bitmap for exact Count Distinct](../using_starrocks/Using_bitmap.md) を参照してください。
ここでは、UV のカウントを例にして、`BITMAP` タイプの列にデータをロードする方法を示します。**`BITMAP` はバージョン 1.1.1 以降でサポートされています**。

1. StarRocks 集約テーブルを作成します。

   データベース `test` で、`visit_users` 列が `BITMAP` タイプであり、集約関数が `BITMAP_UNION` で構成されている `page_uv` という集約テーブルを作成します。

    ```SQL
    CREATE TABLE `test`.`page_uv` (
      `page_id` INT NOT NULL COMMENT 'ページ ID',
      `visit_date` datetime NOT NULL COMMENT 'アクセス時刻',
      `visit_users` BITMAP BITMAP_UNION NOT NULL COMMENT 'ユーザー ID'
    ) ENGINE=OLAP
    AGGREGATE KEY(`page_id`, `visit_date`)
    DISTRIBUTED BY HASH(`page_id`);
    ```

2. Spark テーブルを作成します。

    Spark テーブルのスキーマは StarRocks テーブルから推論されますが、Spark は `BITMAP` タイプをサポートしていません。そのため、Spark で対応する列のデータ型をカスタマイズする必要があります。たとえば、`BIGINT` として設定する場合は、オプション `"starrocks.column.types"="visit_users BIGINT"` を設定します。データをロードする際には、コネクタは [`to_bitmap`](../sql-reference/sql-functions/bitmap-functions/to_bitmap.md) 関数を使用して、`BIGINT` タイプのデータを `BITMAP` タイプに変換します。

    `spark-sql` で以下の DDL を実行します:

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

    `spark-sql` で以下の DML を実行します:

    ```SQL
    INSERT INTO `page_uv` VALUES
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 13),
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 23),
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 33),
       (1, CAST('2020-06-23 02:30:30' AS TIMESTAMP), 13),
       (2, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 23);
    ```

4. StarRocks テーブルからページの UV を計算します。

    ```SQL
    MySQL [test]> SELECT `page_id`, COUNT(DISTINCT `visit_users`) FROM `page_uv` GROUP BY `page_id`;
    +---------+-----------------------------+
    | page_id | count(DISTINCT visit_users) |
    +---------+-----------------------------+
    |       2 |                           1 |
    |       1 |                           3 |
    +---------+-----------------------------+
    2 行がセットされました (0.01 秒)
    ```

> **注意:**
>
> コネクタは、Spark の `TINYINT`、`SMALLINT`、`INTEGER`、`BIGINT` タイプのデータを StarRocks の `BITMAP` タイプに変換するために [`to_bitmap`](../sql-reference/sql-functions/bitmap-functions/to_bitmap.md) 関数を使用し、その他の Spark データ型に対しては [`bitmap_hash`](../sql-reference/sql-functions/bitmap-functions/bitmap_hash.md) 関数を使用します。

### HLL タイプの列にデータをロードする

[`HLL`](../sql-reference/sql-statements/data-types/HLL.md) は、近似的なカウントのために使用されます。詳細は [Use HLL for approximate count distinct](../using_starrocks/Using_HLL.md) を参照してください。
ここでは、UV のカウントを例にして、`HLL` タイプの列にデータをロードする方法を示します。**`HLL` はバージョン 1.1.1 以降でサポートされています**。

1. StarRocks 集約テーブルを作成します。

   データベース `test` で、`visit_users` 列が `HLL` タイプであり、集約関数が `HLL_UNION` で構成されている `hll_uv` という集約テーブルを作成します。

    ```SQL
    CREATE TABLE `hll_uv` (
    `page_id` INT NOT NULL COMMENT 'ページ ID',
    `visit_date` datetime NOT NULL COMMENT 'アクセス時刻',
    `visit_users` HLL HLL_UNION NOT NULL COMMENT 'ユーザー ID'
    ) ENGINE=OLAP
    AGGREGATE KEY(`page_id`, `visit_date`)
    DISTRIBUTED BY HASH(`page_id`);
    ```

2. Spark テーブルを作成します。

   Spark テーブルのスキーマは StarRocks テーブルから推論されますが、Spark は `HLL` タイプをサポートしていません。そのため、Spark で対応する列のデータ型をカスタマイズする必要があります。たとえば、`BIGINT` として設定する場合は、オプション `"starrocks.column.types"="visit_users BIGINT"` を設定します。データをロードする際には、コネクタは [`hll_hash`](../sql-reference/sql-functions/aggregate-functions/hll_hash.md) 関数を使用して、`BIGINT` タイプのデータを `HLL` タイプに変換します。

    `spark-sql` で以下の DDL を実行します:

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

    `spark-sql` で以下の DML を実行します:

    ```SQL
    INSERT INTO `hll_uv` VALUES
       (3, CAST('2023-07-24 12:00:00' AS TIMESTAMP), 78),
       (4, CAST('2023-07-24 13:20:10' AS TIMESTAMP), 2),
       (3, CAST('2023-07-24 12:30:00' AS TIMESTAMP), 674);
    ```

4. StarRocks テーブルからページの UV を計算します。

    ```SQL
    MySQL [test]> SELECT `page_id`, COUNT(DISTINCT `visit_users`) FROM `hll_uv` GROUP BY `page_id`;
    +---------+-----------------------------+
    | page_id | count(DISTINCT visit_users) |
    +---------+-----------------------------+
    |       4 |                           1 |
    |       3 |                           2 |
    +---------+-----------------------------+
    2 行がセットされました (0.01 秒)
    ```

### ARRAY タイプの列にデータをロードする

以下の例では、[`ARRAY`](../sql-reference/sql-statements/data-types/Array.md) タイプの列にデータをロードする方法を説明します。

1. StarRocks テーブルを作成します。

   データベース `test` で、`INT` 列と 2 つの `ARRAY` 列を含む主キー付きテーブル `array_tbl` を作成します。

   ```SQL
   CREATE TABLE `array_tbl` (
       `id` INT NOT NULL,
       `a0` ARRAY<STRING>,
       `a1` ARRAY<ARRAY<INT>>
    )
    ENGINE=OLAP
    PRIMARY KEY(`id`)
    DISTRIBUTED BY HASH(`id`)
    ;
    ```

2. StarRocks にデータを書き込みます。

   StarRocks の一部のバージョンでは、`ARRAY` 列のメタデータが提供されないため、コネクタはこの列の対応する Spark データ型を推論することができません。ただし、オプション `starrocks.column.types` で列の対応する Spark データ型を明示的に指定することができます。この例では、オプションを `a0 ARRAY<STRING>,a1 ARRAY<ARRAY<INT>>` と設定します。

   `spark-shell` で以下のコードを実行します:

   ```scala
    val data = Seq(
       |  (1, Seq("hello", "starrocks"), Seq(Seq(1, 2), Seq(3, 4))),
       |  (2, Seq("hello", "spark"), Seq(Seq(5, 6, 7), Seq(8, 9, 10)))
       | )
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

3. StarRocks テーブルでデータをクエリします。

   ```SQL
   MySQL [test]> SELECT * FROM `array_tbl`;
   +------+-----------------------+--------------------+
   | id   | a0                    | a1                 |
   +------+-----------------------+--------------------+
   |    1 | ["hello","starrocks"] | [[1,2],[3,4]]      |
   |    2 | ["hello","spark"]     | [[5,6,7],[8,9,10]] |
   +------+-----------------------+--------------------+
   2 行がセットされました (0.01 秒)
   ```
