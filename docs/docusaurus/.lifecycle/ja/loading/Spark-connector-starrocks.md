---
displayed_sidebar: "Japanese"
---

# Sparkコネクタを使用してデータをロードする（推奨）

StarRocksでは、Spark Connector for Apache Spark™（以下、Sparkコネクタ）という自社開発のコネクタを使用して、データを蓄積し、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を介してStarRocksテーブルに一括でロードすることで、Sparkを使用してデータをStarRocksテーブルにロードすることができます。SparkコネクタはSpark DataSource V2に基づいて実装されています。Spark DataFramesまたはSpark SQLを使用してDataSourceを作成することができます。バッチおよび構造化ストリーミングモードの両方がサポートされています。

> **注意**
>
> StarRocksテーブルにおいてSELECTおよびINSERT特権を持つユーザーのみが、このテーブルにデータをロードできます。ユーザーにこれらの特権を付与する方法については、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)で提供されている手順に従ってください。

## バージョン要件

| Sparkコネクタ | Spark            | StarRocks     | Java | Scala |
| --------------- | ---------------- | ------------- | ---- | ----- |
| 1.1.1 | 3.2, 3.3, または 3.4 | 2.5以降 | 8 | 2.12 |
| 1.1.0           | 3.2, 3.3, または 3.4 | 2.5以降 | 8    | 2.12  |

> **注意**
>
> - 異なるバージョンのSparkコネクタ間の動作変更については、[Sparkコネクタのアップグレード](#upgrade-spark-connector)を参照してください。
> - Sparkコネクタは、バージョン1.1.1以降ではMySQL JDBCドライバを提供しておらず、ドライバをSparkのクラスパスに手動でインポートする必要があります。ドライバは[MySQLサイト](https://dev.mysql.com/downloads/connector/j/)または[Maven Central](https://repo1.maven.org/maven2/mysql/mysql-connector-java/)で取得できます。

## Sparkコネクタの取得

以下の方法でSparkコネクタのJARファイルを取得できます。

- コンパイル済みのSparkコネクタJARファイルを直接ダウンロードします。
- MavenプロジェクトにSparkコネクタを依存関係として追加し、JARファイルをダウンロードします。
- Sparkコネクタのソースコードを自分でコンパイルしてJARファイルを作成します。

SparkコネクタのJARファイルの命名形式は`starrocks-spark-connector-${spark_version}_${scala_version}-${connector_version}.jar`です。

例えば、環境にSpark 3.2およびScala 2.12がインストールされており、Sparkコネクタ1.1.0を使用したい場合は、`starrocks-spark-connector-3.2_2.12-1.1.0.jar`を使用できます。

> **注意**
>
> 一般的に、Sparkコネクタの最新バージョンは、最新の3つのSparkバージョンとの互換性を維持します。

### コンパイル済みのJARファイルのダウンロード

[Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks)から、対応するSparkコネクタのJARファイルを直接ダウンロードします。

### Maven依存関係

1. Mavenプロジェクトの`pom.xml`ファイルに、以下の形式に従ってSparkコネクタを依存関係として追加します。`spark_version`、`scala_version`、`connector_version`をそれぞれのバージョンに置き換えてください。

    ```xml
    <dependency>
    <groupId>com.starrocks</groupId>
    <artifactId>starrocks-spark-connector-${spark_version}_${scala_version}</artifactId>
    <version>${connector_version}</version>
    </dependency>
    ```

2. 例えば、環境のSparkのバージョンが3.2でScalaのバージョンが2.12で、Sparkコネクタ1.1.0を選択した場合、以下の依存関係を追加する必要があります。

    ```xml
    <dependency>
    <groupId>com.starrocks</groupId>
    <artifactId>starrocks-spark-connector-3.2_2.12</artifactId>
    <version>1.1.0</version>
    </dependency>
    ```

### 自分でコンパイルする

1. [Sparkコネクタパッケージ](https://github.com/StarRocks/starrocks-connector-for-apache-spark)をダウンロードします。
2. SparkコネクタのソースコードをJARファイルにコンパイルするために、以下のコマンドを実行します。`spark_version`は対応するSparkのバージョンに置き換えてください。

      ```bash
      sh build.sh <spark_version>
      ```

   例えば、環境のSparkのバージョンが3.2の場合、以下のコマンドを実行する必要があります。

      ```bash
      sh build.sh 3.2
      ```

3. `target/`ディレクトリに移動して、コンパイル後に生成された`starrocks-spark-connector-3.2_2.12-1.1.0-SNAPSHOT.jar`などのSparkコネクタのJARファイルを見つけることができます。

> **注意**
>
> 正式にリリースされていないSparkコネクタの名前には、`SNAPSHOT`が含まれています。

## パラメータ

| パラメータ                                      | 必須 | デフォルト値 | 説明                                                  |
| ---------------------------------------------- | ---- | ------------- | ------------------------------------------------------------ |
| starrocks.fe.http.url                          | YES  | なし          | StarRocksクラスタ内のFEのHTTP URLです。複数のURLを指定することができますが、これらはカンマ（,）で区切られている必要があります。形式：`<fe_host1>:<fe_http_port1>,<fe_host2>:<fe_http_port2>`。バージョン1.1.1以降では、URLに`http://`を追加することもできます。例：`http://<fe_host1>:<fe_http_port1>,http://<fe_host2>:<fe_http_port2>`。|
| starrocks.fe.jdbc.url                          | YES  | なし          | FEのMySQLサーバーに接続するためのアドレスです。形式：`jdbc:mysql://<fe_host>:<fe_query_port>`。 |
| starrocks.table.identifier                     | YES  | なし          | StarRocksテーブルの名前です。形式：`<database_name>.<table_name>`。 |
| starrocks.user                                 | YES  | なし          | StarRocksクラスタのアカウントのユーザー名です。ユーザーにはStarRocksテーブルにおける[SELECTおよびINSERT特権](../sql-reference/sql-statements/account-management/GRANT.md)が必要です。             |
| starrocks.password                             | YES  | なし          | StarRocksクラスタのアカウントのパスワードです。              |
| starrocks.write.label.prefix                   | NO   | spark-        | Stream Loadで使用されるラベルのプレフィックスです。                        |
| starrocks.write.enable.transaction-stream-load | NO   | TRUE | データをロードする際に[Stream Loadトランザクションインターフェース](../loading/Stream_Load_transaction_interface.md)を使用するかどうかを指定します。StarRocks v2.5以降が必要です。この機能を使用すると、トランザクション内でより多くのデータをロードし、メモリの使用量を減らし、パフォーマンスを向上させることができます。<br/>**注意:** 1.1.1以降、このパラメータは、`starrocks.write.max.retries`の値が非正である場合にのみ有効となります。なぜなら、Stream Loadトランザクションインターフェースはリトライをサポートしていないためです。 |
| starrocks.write.buffer.size                    | NO   | 104857600     | 一度にStarRocksに送信される前にメモリに蓄積できるデータの最大サイズです。このパラメータを大きな値に設定すると、ロードのパフォーマンスが向上する可能性がありますが、ロードの遅延が増加する可能性があります。 |
| starrocks.write.buffer.rows | NO | Integer.MAX_VALUE | バージョン1.1.1以降でサポートされています。一度にStarRocksに送信される前にメモリに蓄積できる行の最大数です。 |
| starrocks.write.flush.interval.ms              | NO   | 300000        | データをStarRocksに送信する間隔です。このパラメータは、ロードの遅延を制御するために使用されます。 |
| starrocks.write.max.retries                    | NO   | 3             | バージョン1.1.1以降でサポートされています。同じデータのバッチのロードに失敗した場合、コネクタがリトライする回数です。<br/>**注意:** Stream Loadトランザクションインターフェースはリトライをサポートしていないため、このパラメータが正の値の場合、コネクタは常にStream Loadインターフェースを使用し、`starrocks.write.enable.transaction-stream-load`の値を無視します。|
| starrocks.write.retry.interval.ms              | NO   | 10000         | バージョン1.1.1以降でサポートされています。同じデータのバッチのロードが失敗した場合のリトライ間隔です。|
| starrocks.columns                              | NO   | なし          | データをロードしたいStarRocksテーブルの列です。複数の列を指定することができますが、これらはカンマ（,）で区切られている必要があります。 例：`"col0,col1,col2"`。 |
| starrocks.column.types                         | NO       | None          | バージョン1.1.1以降でサポートされています。Sparkの既定値から推論されたStarRocksテーブルおよび[デフォルトマッピング](#data-type-mapping-between-spark-and-starrocks)の代わりに、Spark用の列データ型をカスタマイズします。パラメータの値は、Sparkの[StructType#toDDL](https://github.com/apache/spark/blob/master/sql/api/src/main/scala/org/apache/spark/sql/types/StructType.scala#L449)の出力と同じDDL形式のスキーマです。例えば `col0 INT, col1 STRING, col2 BIGINT`です。カスタマイズが必要な列のみを指定する必要があります。ビットマップタイプまたはHLLタイプの列にデータをロードする場合の1つのユースケースです。|
| starrocks.write.properties.*                   | NO       | None          | ストリームロードの動作を制御するために使用されるパラメータです。  たとえば、パラメータ `starrocks.write.properties.format` は、CSVやJSONなどデータをロードするフォーマットを指定します。サポートされているパラメータとその説明のリストについては、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。 |
| starrocks.write.properties.format              | NO       | CSV           | SparkコネクタがStarRocksにデータを送信する前に、各データバッチを変換するファイルフォーマットです。有効な値は、CSVとJSONです。 |
| starrocks.write.properties.row_delimiter       | NO       | \n            | CSV形式のデータの行区切り記号です。                    |
| starrocks.write.properties.column_separator    | NO       | \t            | CSV形式のデータの列区切り記号です。                 |
| starrocks.write.num.partitions                 | NO       | None          | Sparkがデータを並列書き込みするパーティション数です。データ量が少ない場合、パーティション数を減らしてロードの並列度と頻度を下げることができます。このパラメータの既定値はSparkによって決定されます。ただし、この方法ではSparkのシャッフルコストが発生する可能性があります。 |
| starrocks.write.partition.columns              | NO       | None          | Sparkのパーティション列です。パラメータは、`starrocks.write.num.partitions`が指定された場合のみ効果があります。このパラメータが指定されていない場合、書き込まれるすべての列がパーティション化に使用されます。 |
| starrocks.timezone | NO | JVMのデフォルトタイムゾーン | 1.1.1以降でサポートされています。Sparkの`TimestampType`をStarRocksの`DATETIME`に変換するために使用されるタイムゾーンです。デフォルトは`ZoneId#systemDefault()`によって返されるJVMのタイムゾーンです。形式は、`Asia/Shanghai`などのタイムゾーン名、または`+08:00`などのゾーンオフセットです。 |

## SparkとStarRocksのデータ型マッピング

- デフォルトのデータ型マッピングは次のとおりです：

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
  | ArrayType       | ARRAY <br /> **NOTE:** <br /> バージョン1.1.1以降でサポートされています。詳細な手順については、「ビットマップタイプの列にデータをロードする」(#load-data-into-columns-of-array-type)を参照してください。 |

- データ型マッピングをカスタマイズすることもできます。

  たとえば、StarRocksテーブルにはBITMAPおよびHLL列が含まれていますが、Sparkはこれら2つのデータ型をサポートしていません。したがって、Sparkで対応するデータ型をカスタマイズする必要があります。詳細な手順については、「ビットマップ」(#load-data-into-columns-of-bitmap-type)および「HLL」(#load-data-into-columns-of-hll-type)の列にデータをロードするを参照してください。 **BITMAPおよびHLLはバージョン1.1.1以降でサポートされています**。

## Sparkコネクタのアップグレード

### バージョン1.1.0から1.1.1へのアップグレード

- バージョン1.1.1以降、SparkコネクタはMySQLの公式JDBCドライバである `mysql-connector-java` を提供していません。これは、 `mysql-connector-java` が使用しているGPLライセンスの制限によるものです。
  ただし、まだテーブルメタデータを取得するためにSparkコネクタがMySQL JDBCドライバを必要としているため、MySQL JDBCドライバをSparkのクラスパスに手動で追加する必要があります。ドライバは[MySQLサイト](https://dev.mysql.com/downloads/connector/j/)または[Maven Central](https://repo1.maven.org/maven2/mysql/mysql-connector-java/)から見つけることができます。
- バージョン1.1.1以降、ストリームロードインターフェースを使用するデフォルトがStream Loadトランザクションインタフェースではなくなりました。まだStream Loadトランザクションインタフェースを使用したい場合は、オプション `starrocks.write.max.retries` を `0` に設定することができます。詳細については、 `starrocks.write.enable.transaction-stream-load` および `starrocks.write.max.retries` の説明を参照してください。

## 例

以下の例では、Sparkコネクタを使用してSpark DataFramesまたはSpark SQLを使用してStarRocksテーブルにデータをロードする方法を示しています。Spark DataFramesは、バッチモードとストリーミングモードの両方をサポートしています。

その他の例については、[Spark Connector Examples](https://github.com/StarRocks/starrocks-connector-for-apache-spark/tree/main/src/test/java/com/starrocks/connector/spark/examples)を参照してください。

### 準備

#### StarRocksテーブルの作成

データベース `test` を作成し、プライマリキーのテーブル `score_board` を作成します。

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

以下の例は、Spark 3.2.4で実行され、`spark-shell`、`pyspark`、`spark-sql`が使用されます。例を実行する前に、SparkコネクタのJARファイルを `$SPARK_HOME/jars` ディレクトリに配置してください。

### Spark DataFramesを使用してデータをロードする

次の2つの例では、Spark DataFramesのバッチモードまたはストリーミングモードでデータをロードする方法について説明します。

#### バッチ

メモリ内でデータを構築し、そのデータをStarRocksテーブルにロードします。

1. ScalaまたはPythonを使用して、次のコードスニペットを実行してSparkアプリケーションを作成できます。

  Scalaの場合は、 `spark-shell` で次のコードスニペットを実行します：

  ```Scala
  // 1. シーケンスからDataFrameを作成します。
  val data = Seq((1, "starrocks", 100), (2, "spark", 100))
  val df = data.toDF("id", "name", "score")

  // 2. フォーマットを "starrocks" に設定し、次のオプションを構成してStarRocksに書き込みます。 
  // 環境に応じてオプションを修正する必要があります。
  df.write.format("starrocks")
      .option("starrocks.fe.http.url", "127.0.0.1:8030")
      .option("starrocks.fe.jdbc.url", "jdbc:mysql://127.0.0.1:9030")
      .option("starrocks.table.identifier", "test.score_board")
      .option("starrocks.user", "root")
      .option("starrocks.password", "")
      .mode("append")
      .save()
  ```

  Pythonの場合は、 `pyspark` で次のコードスニペットを実行します：

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

    # 2. フォーマットを "starrocks" に設定し、次のオプションを構成してStarRocksに書き込みます。 
    # 環境に応じてオプションを修正する必要があります。
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
    2 行が設定されました (0.00 秒)

#### 構造化ストリーミング

CSVファイルからのデータのストリーミング読み取りを構築し、データをStarRocksテーブルにロードします。

1. `csv-data`ディレクトリ内に、以下のデータを持つCSVファイル`test.csv`を作成してください。

    ```csv
    3,starrocks,100
    4,spark,100
    ```

2. ScalaまたはPythonを使用してSparkアプリケーションを記述できます。

  Scalaの場合、`spark-shell`で次のコードスニペットを実行してください：

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
            // ディレクトリ "csv-data"へのパスに置き換えてください。
            .load("/path/to/csv-data")
        )
    
    // 2. テーブルを"starrocks"として構成し、次のオプションを使用してStarRocksに書き込みます。
    // 環境に合わせてオプションを変更する必要があります。
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

  Pythonの場合、`pyspark`で次のコードスニペットを実行してください：

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
        # ディレクトリ "csv-data"へのパスに置き換えてください。
        .load("/path/to/csv-data")
    )

    # 2. テーブルを"starrocks"として構成し、次のオプションを使用してStarRocksに書き込みます。
    # 環境に合わせてオプションを変更する必要があります。
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
    2 行が設定されました (0.67 秒)
    ```

### Spark SQLを使用してデータを読み込む

以下の例では、[Spark SQL CLI](https://spark.apache.org/docs/latest/sql-distributed-sql-engine-spark-sql-cli.html)で`INSERT INTO`ステートメントを使用してSpark SQLでデータを読み込む方法について説明します。

1. `spark-sql`で次のSQLステートメントを実行してください：

    ```SQL
    -- 1. データソースを「starrocks」として構成し、次のオプションを使用してテーブルを作成します。
    -- 環境に合わせてオプションを変更する必要があります。
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

2. StarRocksテーブルでデータをクエリします。

    ```SQL
    MySQL [test]> select * from score_board;
    +------+-----------+-------+
    | id   | name      | score |
    +------+-----------+-------+
    |    6 | spark     |   100 |
    |    5 | starrocks |   100 |
    +------+-----------+-------+
    2 行が設定されました (0.00 秒)
    ```

## ベストプラクティス

### 主キーテーブルへのデータのロード

このセクションでは、部分更新や条件付き更新を実現するためにStarRocksの主キーテーブルにデータをロードする方法を示します。
これらの機能の詳細は、 [Change data through loading](../loading/Load_to_Primary_Key_tables.md) をご覧ください。
これらの例ではSpark SQLが使用されています。

#### 準備

StarRocksで`test`データベースを作成し、Primary Keyテーブル`score_board`を作成してください。

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

この例では、ロードを介して列 `name` のデータのみを更新する方法を示します。

1. MySQLクライアントで初期データをStarRocksテーブルに挿入してください。

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

2. Spark SQLクライアントでSparkテーブル `score_board` を作成してください。

   - オプション `starrocks.write.properties.partial_update` を `true` に設定し、部分更新を行うようにコネクタに指示します。
   - オプション `starrocks.columns` を `"id,name"` に設定して、書き込む列をコネクタに伝えます。

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

3. Spark SQLクライアントでテーブルにデータを挿入し、列 `name` のみを更新してください。

   ```SQL
   INSERT INTO `score_board` VALUES (1, 'starrocks-update'), (2, 'spark-update');
   ```

4. MySQLクライアントでStarRocksテーブルをクエリしてください。

   `name`の値のみが変更され、`score`の値は変更されないことが分かります。

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

この例では、列 `score` の値に応じて条件付き更新を行う方法を示します。 `id`の更新は、新しい値が古い値より大きいか等しい場合にのみ有効になります。
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

2. Sparkテーブル `score_board` を以下の方法で作成します。

   - オプション`starrocks.write.properties.merge_condition`を`score`に設定し、コネクタが列`score`を条件として使用するよう指示します。
   - Sparkコネクタがこの機能をサポートしていないため、スパークコネクタがデータを読み込む際にStream Loadインターフェースを使用し、Stream Loadトランザクションインターフェースは使用しないでください。

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

3. Spark SQLクライアントでテーブルにデータを挿入し、`id`が1の行のスコア値を小さく、`id`が2の行のスコア値を大きく更新します。

   ```SQL
   INSERT INTO `score_board` VALUES (1, 'starrocks-update', 99), (2, 'spark-update', 101);
   ```

4. MySQLクライアントでStarRocksテーブルをクエリします。

   `id`が2の行のみが変更され、`id`が1の行は変更されていないことがわかります。

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

### BITMAP型の列にデータをロードする

[`BITMAP`](../sql-reference/sql-statements/data-types/BITMAP.md)は、UVの数を高速にカウントするためによく使用され、[Use Bitmap for exact Count Distinct](../using_starrocks/Using_bitmap.md)を参照してください。
ここでは、`BITMAP`型のカラムにデータをロードする方法をUVのカウントの例として示します。**`BITMAP`はバージョン1.1.1からサポートされています**。

1. StarRocks集約テーブルを作成します。

   `test`データベースで、カラム`visit_users`が`BITMAP`型として定義され、集約関数`BITMAP_UNION`で構成された集約テーブル`page_uv`を作成します。

   ```SQL
   CREATE TABLE `test`.`page_uv` (
     `page_id` INT NOT NULL COMMENT 'ページID',
     `visit_date` datetime NOT NULL COMMENT 'アクセス時間',
     `visit_users` BITMAP BITMAP_UNION NOT NULL COMMENT 'ユーザーID'
   ) ENGINE=OLAP
   AGGREGATE KEY(`page_id`, `visit_date`)
   DISTRIBUTED BY HASH(`page_id`);
   ```

2. Sparkテーブルを作成します。

   SparkのスキーマはStarRocksテーブルから推論されますが、Sparkは`BITMAP`型をサポートしていませんので、Spark内で対応するカラムのデータタイプを`BIGINT`などにカスタマイズする必要があります。例えば、オプション`"starrocks.column.types"="visit_users BIGINT"`を設定します。Stream Loadを使用してデータを取り込む際、コネクタは`BIGINT`型のデータを`BITMAP`型に変換するために[`to_bitmap`](../sql-reference/sql-functions/bitmap-functions/to_bitmap.md)関数を使用します。

    以下のDDLを`spark-sql`で実行します。

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

3. StarRocksテーブルにデータをロードします。

    以下のDMLを`spark-sql`で実行します。

    ```SQL
    INSERT INTO `page_uv` VALUES
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 13),
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 23),
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 33),
       (1, CAST('2020-06-23 02:30:30' AS TIMESTAMP), 13),
       (2, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 23);
    ```

4. StarRocksテーブルからページUVを計算します。

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

> **注意:**
>
> コネクタは、Sparkの`TINYINT`、`SMALLINT`、`INTEGER`、`BIGINT`型のデータをStarRocksの`BITMAP`型に変換するために[`to_bitmap`](../sql-reference/sql-functions/bitmap-functions/to_bitmap.md)関数を使用し、その他のSparkデータ型に対しては[`bitmap_hash`](../sql-reference/sql-functions/bitmap-functions/bitmap_hash.md)関数を使用します。

### HLL型の列にデータをロードする

[`HLL`](../sql-reference/sql-statements/data-types/HLL.md)は、近似カウントのために使用され、[Use HLL for approximate count distinct](../using_starrocks/Using_HLL.md)を参照してください。

ここでは、`HLL`型の列にデータをロードする方法をUVのカウントの例として示します。**`HLL`はバージョン1.1.1からサポートされています**。

1. StarRocks集約テーブルを作成します。

   `test`データベースで、カラム`visit_users`が`HLL`型として定義され、集約関数`HLL_UNION`で構成された集約テーブル`hll_uv`を作成します。

    ```SQL
    CREATE TABLE `hll_uv` (
    `page_id` INT NOT NULL COMMENT 'ページID',
    `visit_date` datetime NOT NULL COMMENT 'アクセス時間',
    `visit_users` HLL HLL_UNION NOT NULL COMMENT 'ユーザーID'
    ) ENGINE=OLAP
    AGGREGATE KEY(`page_id`, `visit_date`)
    DISTRIBUTED BY HASH(`page_id`);
    ```

2. Sparkテーブルを作成します。

   SparkのスキーマはStarRocksテーブルから推論されますが、Sparkは`HLL`型をサポートしていないため、Spark内で対応するカラムのデータタイプを`BIGINT`などにカスタマイズする必要があります。例えば、オプション`"starrocks.column.types"="visit_users BIGINT"`を設定します。Stream Loadを使用してデータを取り込む際、コネクタは`BIGINT`型のデータを`HLL`型に変換するために[`hll_hash`](../sql-reference/sql-functions/aggregate-functions/hll_hash.md)関数を使用します。

    以下のDDLを`spark-sql`で実行します。

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

3. StarRocksテーブルにデータをロードします。

    以下のDMLを`spark-sql`で実行します。

    ```SQL
    INSERT INTO `hll_uv` VALUES
       (3, CAST('2023-07-24 12:00:00' AS TIMESTAMP), 78),
       (4, CAST('2023-07-24 13:20:10' AS TIMESTAMP), 2),
       (3, CAST('2023-07-24 12:30:00' AS TIMESTAMP), 674);
    ```

4. StarRocksテーブルからページのUVを計算します。

    ```SQL
    MySQL [test]> SELECT `page_id`, COUNT(DISTINCT `visit_users`) FROM `hll_uv` GROUP BY `page_id`;
    +---------+-----------------------------+
    | page_id | count(DISTINCT visit_users) |
    +---------+-----------------------------+
    |       4 |                           1 |
    |       3 |                           2 |
    +---------+-----------------------------+
    2 行が返されました (0.01 秒)
    ```

### ARRAYタイプのカラムにデータをロードする

次の例では、[`ARRAY`](../sql-reference/sql-statements/data-types/Array.md) タイプのカラムにデータをロードする方法を説明します。

1. StarRocksテーブルを作成します。

   `test`データベースに、1つの`INT`カラムと2つの`ARRAY`カラムを含むPrimary Keyテーブル`array_tbl`を作成します。

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

2. StarRocksにデータを書き込みます。

   StarRocksの一部のバージョンは`ARRAY`カラムのメタデータを提供しないため、コネクタはこのカラムの対応するSparkデータ型を推測できません。ただし、オプション`starrocks.column.types`でカラムの対応するSparkデータ型を明示的に指定できます。この例では、オプションを`a0 ARRAY<STRING>,a1 ARRAY<ARRAY<INT>>`として構成できます。

   以下のコードを`spark-shell`で実行します:

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

3. StarRocksテーブルでデータをクエリします。

   ```SQL
   MySQL [test]> SELECT * FROM `array_tbl`;
   +------+-----------------------+--------------------+
   | id   | a0                    | a1                 |
   +------+-----------------------+--------------------+
   |    1 | ["hello","starrocks"] | [[1,2],[3,4]]      |
   |    2 | ["hello","spark"]     | [[5,6,7],[8,9,10]] |
   +------+-----------------------+--------------------+
   2 行が返されました (0.01 秒)
   ```