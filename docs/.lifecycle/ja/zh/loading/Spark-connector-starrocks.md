---
displayed_sidebar: Chinese
---

# Spark connector を使用してデータをインポートする（推奨）

StarRocks は Apache Spark™ 用のコネクタ（StarRocks Connector for Apache Spark™）を提供しており、Spark を通じて StarRocks にデータをインポートすることができます（推奨）。
基本的な原理は、データをバッチ処理した後、[Stream Load](./StreamLoad.md) を使用して StarRocks にバッチでインポートすることです。コネクタは Spark DataSource V2 をベースにデータインポートを実現し、
Spark DataFrame や Spark SQL を使用して DataSource を作成でき、Batch および Structured Streaming をサポートしています。

> **注意**
>
> StarRocks へのデータインポートには、対象テーブルの SELECT および INSERT 権限が必要です。お持ちのユーザーアカウントにこれらの権限がない場合は、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md) を参照してユーザーに権限を付与してください。

## バージョン要件

| コネクタ | Spark           | StarRocks | Java  | Scala |
|----------|-----------------|-----------|-------| ---- |
| 1.1.1    | 3.2, 3.3, 3.4   | 2.5 以上   | 8     | 2.12 |
| 1.1.0    | 3.2, 3.3, 3.4   | 2.5 以上   | 8     | 2.12 |

> **注意**
>
> - 異なるバージョンの Spark connector の挙動の変更点については、[Spark connector のアップグレード](#升级-spark-connector)をご覧ください。
> - バージョン 1.1.1 以降、Spark connector は MySQL JDBC ドライバーを提供しなくなりました。ドライバーは手動で Spark のクラスパスに配置する必要があります。ドライバーは [MySQL 公式サイト](https://dev.mysql.com/downloads/connector/j/)または [Maven Central Repository](https://repo1.maven.org/maven2/mysql/mysql-connector-java/) で入手できます。

## コネクタの取得

コネクタの jar パッケージは以下の方法で入手できます。

- コンパイル済みの jar を直接ダウンロードする
- Maven でコネクタの依存関係を追加する
- ソースコードから手動でコンパイルする

コネクタ jar パッケージの命名規則は以下の通りです。

`starrocks-spark-connector-${spark_version}_${scala_version}-${connector_version}.jar`

例えば、Spark 3.2 と Scala 2.12 でバージョン 1.1.0 のコネクタを使用する場合は、`starrocks-spark-connector-3.2_2.12-1.1.0.jar` を選択します。

> **注意**
>
> 一般的に、最新バージョンのコネクタは最新の 3 つの Spark バージョンのみをサポートしています。

### 直接ダウンロード

[Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks) から異なるバージョンのコネクタ jar を入手できます。

### Maven 依存関係

依存関係の設定は以下の形式で、`spark_version`、`scala_version`、`connector_version` を対応するバージョンに置き換えます。

```xml
<dependency>
  <groupId>com.starrocks</groupId>
  <artifactId>starrocks-spark-connector-${spark_version}_${scala_version}</artifactId>
  <version>${connector_version}</version>
</dependency>
```

例えば、Spark 3.2 と Scala 2.12 でバージョン 1.1.0 のコネクタを使用する場合は、以下の依存関係を追加します。

```xml
<dependency>
  <groupId>com.starrocks</groupId>
  <artifactId>starrocks-spark-connector-3.2_2.12</artifactId>
  <version>1.1.0</version>
</dependency>
```

### 手動でコンパイル

1. [Spark コネクタのソースコード](https://github.com/StarRocks/starrocks-connector-for-apache-spark) をダウンロードします。

2. 以下のコマンドでコンパイルを行い、`spark_version` を対応する Spark のバージョンに置き換えます。

      ```shell
      sh build.sh <spark_version>
      ```

   例えば、Spark 3.2 で使用する場合は、以下のコマンドを実行します。

      ```shell
      sh build.sh 3.2
      ```

3. コンパイルが完了すると、`target/` ディレクトリにコネクタの jar パッケージが生成されます。例：`starrocks-spark-connector-3.2_2.12-1.1-SNAPSHOT.jar`。

> **注意**
>
> 正式リリースされていないコネクタのバージョンには `SNAPSHOT` のサフィックスが付きます。

## パラメータ説明

| パラメータ                                           | 必須     | デフォルト値 | 説明                                                                                                                                                                                                                    |
|------------------------------------------------|-------- | ---- |-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| starrocks.fe.http.url                          | はい      | なし | FE の HTTP アドレス。複数の FE アドレスを入力する場合は、コンマ `,` で区切ります。形式：`<fe_host1>:<fe_http_port1>,<fe_host2>:<fe_http_port2>`。バージョン 1.1.1 からは、URL に `http://` プレフィックスを追加することもできます。例：`http://<fe_host1>:<fe_http_port1>,http://<fe_host2>:<fe_http_port2>`。|
| starrocks.fe.jdbc.url                          | はい      | なし | FE の MySQL Server 接続アドレス。形式：`jdbc:mysql://<fe_host>:<fe_query_port>`。                                                                                                                                                    |
| starrocks.table.identifier                     | はい      | なし | StarRocks の対象テーブル名。形式：`<database_name>.<table_name>`。                                                                                                                                                                    |
| starrocks.user                                 | はい      | なし | StarRocks クラスタのユーザー名。StarRocks へのデータインポートには対象テーブルの SELECT および INSERT 権限が必要です。権限がない場合は、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md) を参照してください。                                                                                                                                                                                                   |
| starrocks.password                             | はい      | なし | StarRocks クラスタのパスワード。                                                                                                                                                                                                  |
| starrocks.write.label.prefix                   | いいえ      | spark- | Stream Load で使用するラベルのプレフィックスを指定します。                                                                                                                                                                                             |
| starrocks.write.enable.transaction-stream-load | いいえ      | true | [Stream Load トランザクションインターフェース](../loading/Stream_Load_transaction_interface.md)を使用してデータをインポートするかどうか。StarRocks のバージョンが v2.5 以上が必要です。この機能により、一度のインポートトランザクションでより多くのデータをインポートし、メモリ使用量を減らし、パフォーマンスを向上させることができます。<br/>**注意：**<br/>バージョン 1.1.1 以降、`starrocks.write.max.retries` の値が正の数でない場合にのみこのパラメータが有効になります。Stream Load トランザクションインターフェースはリトライをサポートしていません。                                                               |
| starrocks.write.buffer.size                    | いいえ      | 104857600 | メモリに蓄積されるデータ量で、この閾値に達するとデータが一度に StarRocks に送信されます。単位 `k`, `m`, `g` がサポートされています。この値を大きくするとインポート性能が向上しますが、書き込み遅延が発生する可能性があります。                                                                                                                                                              |
| starrocks.write.buffer.rows                    | いいえ      | Integer.MAX_VALUE | バージョン 1.1.1 からサポートされています。メモリに蓄積されるデータ行数で、この閾値に達するとデータが一度に StarRocks に送信されます。                                                                                                                                                              |
| starrocks.write.flush.interval.ms              | いいえ      | 300000 | データをバッチ処理して送信する間隔で、StarRocks へのデータ書き込み遅延を制御するために使用されます。                                                                                                                                                                                       |
| starrocks.write.max.retries                    | いいえ       | 3             | バージョン 1.1.1 からサポートされています。一括データのインポートに失敗した場合、Spark connector がそのバッチのデータを再試行する最大回数。<br/>**注意：**Stream Load トランザクションインターフェースはリトライをサポートしていないため、このパラメータが正の数である場合、Spark connector は常に Stream Load インターフェースを使用し、`starrocks.write.enable.transaction-stream-load` の値は無視されます。|
| starrocks.write.retry.interval.ms              | いいえ       | 10000         | バージョン 1.1.1 からサポートされています。一括データのインポートに失敗した場合、Spark connector が次の試行を行うまでの間隔。|
| starrocks.columns                              | いいえ      | なし | StarRocks のテーブルに一部の列のみを書き込むことをサポートし、このパラメータで列名を指定します。複数の列名はコンマ `,` で区切ります。例：`c0,c1,c2`。                                                                                                                                                       |
| starrocks.write.properties.*                   | いいえ      | なし | Stream Load のパラメータを指定してインポート動作を制御します。例えば、`starrocks.write.properties.format` を使用してインポートデータの形式を CSV または JSON に指定します。詳細なパラメータと説明については、[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md) を参照してください。 |
| starrocks.write.properties.format              | いいえ      | CSV | インポートデータの形式を指定します。CSV または JSON が選択できます。コネクタは各バッチのデータを指定された形式に変換して StarRocks に送信します。                                                                                                                                             |
| starrocks.write.properties.row_delimiter       | いいえ      | \n | CSV 形式でインポートする際に行区切り文字を指定します。                                                                                                                                                                                                  |
| starrocks.write.properties.column_separator    | いいえ      | \t | CSV 形式でインポートする際に列区切り文字を指定します。                                                                                                                                                                                                  |
| starrocks.write.num.partitions                 | いいえ      | なし | 並列書き込みのために Spark が使用するパーティション数。データ量が少ない場合は、パーティション数を減らしてインポートの並行性と頻度を下げることができます。デフォルトのパーティション数は Spark によって決定されます。この機能を使用すると、Spark Shuffle のコストが発生する可能性があります。                                                                                                                                  |
| starrocks.write.partition.columns              | いいえ      | なし | Spark のパーティションに使用する列。`starrocks.write.num.partitions` を指定した場合のみ有効です。指定しない場合は、書き込みに使用されるすべての列でパーティションが行われます。                                                                                                                                               |
| starrocks.timezone                             | いいえ | JVM のデフォルトタイムゾーン | バージョン 1.1.1 からサポートされています。StarRocks のタイムゾーン。Spark の `TimestampType` 型の値を StarRocks の `DATETIME` 型の値に変換するために使用されます。デフォルトは JVM のタイムゾーンで、`ZoneId#systemDefault()` によって返されます。形式はタイムゾーン名（例：Asia/Shanghai）またはタイムゾーンオフセット（例：+08:00）で指定できます。|

## データ型マッピング

- デフォルトのデータ型マッピングは以下の通りです：

  | Spark データ型 | StarRocks データ型                                             |
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
  | ArrayType       | ARRAY <br /> **説明:** <br /> **バージョン 1.1.1 からサポートされています。** 詳細な手順については、[ARRAY 型の列へのインポート](#导入至-array-列)を参照してください。 |

- データ型のカスタムマッピングも可能です。


例えば、StarRocksのテーブルにはBITMAPとHLLの列が含まれていますが、Sparkはこれらのデータタイプをサポートしていません。そのため、Sparkでサポートされているデータタイプを設定し、データタイプのマッピング関係をカスタマイズする必要があります。詳細な手順については、[BITMAP](#导入至-bitmap-列)列と[HLL](#导入至-hll-列)列へのインポートを参照してください。バージョン1.1.1から、BITMAPとHLLの列へのインポートがサポートされています。

## Spark connectorのアップグレード

### 1.1.0から1.1.1へのアップグレード

- バージョン1.1.1から、Spark connectorはMySQL公式のJDBCドライバ`mysql-connector-java`を提供しなくなりました。これは、GPLライセンスを使用しているため、いくつかの制限があるためです。しかし、Spark connectorは依然としてStarRocksに接続してテーブルのメタデータを取得するためにMySQL JDBCドライバが必要です。そのため、ドライバを手動でSparkのクラスパスに追加する必要があります。このドライバは[MySQL公式サイト](https://dev.mysql.com/downloads/connector/j/)または[Maven中央リポジトリ](https://repo1.maven.org/maven2/mysql/mysql-connector-java/)で見つけることができます。
- バージョン1.1.1から、Spark connectorはデフォルトでStream Loadインターフェースを使用し、バージョン1.1.0のStream Loadトランザクションインターフェースではなくなりました。Stream Loadトランザクションインターフェースを引き続き使用したい場合は、オプション`starrocks.write.max.retries`を`0`に設定できます。詳細は、`starrocks.write.enable.transaction-stream-load`および`starrocks.write.max.retries`の説明を参照してください。

## 使用例

StarRocksテーブルにデータを書き込む方法を例を通して説明します。これには、Spark DataFrameとSpark SQLの使用が含まれ、DataFrameにはBatchとStructured Streamingの2つのモードがあります。

より多くの例については、[Spark Connector Examples](https://github.com/StarRocks/starrocks-connector-for-apache-spark/tree/main/src/test/java/com/starrocks/connector/spark/examples)を参照してください。今後、さらに多くの例が追加される予定です。

### 準備

#### StarRocksテーブルの作成

`test`データベースを作成し、その中に`score_board`というプライマリキーテーブルを作成します。

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
DISTRIBUTED BY HASH(`id`)
;
```

#### Spark環境

この例はSpark 3.2.4に基づいており、`spark-shell`、`pyspark`、`spark-sql`を使用してデモンストレーションを行います。実行する前に、connector jarを`$SPARK_HOME/jars`ディレクトリに配置してください。

### Spark DataFrameを使用したデータの書き込み

BatchとStructured Streamingでのデータの書き込み方法を以下に説明します。

#### Batch

この例では、メモリ内でデータを構築し、StarRocksテーブルに書き込む方法を示しています。

1. ScalaまたはPython言語を使用してSparkアプリケーションを書くことができます。

   Scala言語を使用する場合は、`spark-shell`で以下のコードを実行できます：

      ```scala
      // 1. CSVからDataFrameを作成します。
      val data = Seq((1, "starrocks", 100), (2, "spark", 100))
      val df = data.toDF("id", "name", "score")

      // 2. "starrocks"フォーマットを指定し、以下のオプションを設定してStarRocksに書き込みます。
      // 環境に合わせてオプションを変更する必要があります。
      df.write.format("starrocks")
         .option("starrocks.fe.http.url", "127.0.0.1:8030")
         .option("starrocks.fe.jdbc.url", "jdbc:mysql://127.0.0.1:9030")
         .option("starrocks.table.identifier", "test.score_board")
         .option("starrocks.user", "root")
         .option("starrocks.password", "")
         .mode("append")
         .save()
      ```

   Python言語を使用する場合は、`pyspark`で以下のコードを実行できます：

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
    # 環境に合わせてオプションを変更する必要があります。
    df.write.format("starrocks") \
        .option("starrocks.fe.http.url", "127.0.0.1:8030") \
        .option("starrocks.fe.jdbc.url", "jdbc:mysql://127.0.0.1:9030") \
        .option("starrocks.table.identifier", "test.score_board") \
        .option("starrocks.user", "root") \
        .option("starrocks.password", "") \
        .mode("append") \
        .save()
    ```

2. StarRocksで結果をクエリします。

```SQL
MySQL [test]> SELECT * FROM `score_board`;
+------+-----------+-------+
| id   | name      | score |
+------+-----------+-------+
|    1 | starrocks |   100 |
|    2 | spark     |   100 |
+------+-----------+-------+
2行セット (0.00秒)
```

#### Structured Streaming

この例では、CSVファイルからストリームデータを読み込み、StarRocksテーブルに書き込む方法を示しています。

1. `csv-data`ディレクトリに`test.csv`というCSVファイルを作成し、以下のようなデータを入れます。

   ```csv
   3,starrocks,100
   4,spark,100
   ```

2. ScalaまたはPython言語を使用してSparkアプリケーションを書くことができます。

   Scala言語を使用する場合は、`spark-shell`で以下のコードを実行できます：

   ```scala
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
         // ディレクトリ"csv-data"へのパスに置き換えてください。
         .load("/path/to/csv-data")
      )

   // 2. "starrocks"フォーマットを指定し、自分のオプションに置き換えてStarRocksに書き込みます。
   val query = (df.writeStream.format("starrocks")
         .option("starrocks.fe.http.url", "127.0.0.1:8030")
         .option("starrocks.fe.jdbc.url", "jdbc:mysql://127.0.0.1:9030")
         .option("starrocks.table.identifier", "test.score_board")
         .option("starrocks.user", "root")
         .option("starrocks.password", "")
         // チェックポイントディレクトリに置き換えてください。
         .option("checkpointLocation", "/path/to/checkpoint")
         .outputMode("append")
         .start()
      )
   ```

   Python言語を使用する場合は、`pyspark`で以下のコードを実行できます：

   ```python
   from pyspark.sql import SparkSession
   from pyspark.sql.types import IntegerType, StringType, StructType, StructField

   spark = SparkSession \
        .builder \
        .appName("StarRocks SS Example") \
        .getOrCreate()

    # 1. CSVからDataFrameを作成します。
    schema = StructType([ \
            StructField("id", IntegerType()), \
            StructField("name", StringType()), \
            StructField("score", IntegerType()) \
        ])
    df = spark.readStream \
            .option("sep", ",") \
            .schema(schema) \
            .format("csv") \
            // "csv-data"ディレクトリへのパスに置き換えてください。
            .load("/path/to/csv-data")

    # 2. "starrocks"フォーマットを指定し、以下のオプションを設定してStarRocksに書き込みます。
    # 環境に合わせてオプションを変更する必要があります。
    query = df.writeStream.format("starrocks") \
            .option("starrocks.fe.http.url", "127.0.0.1:8030") \
            .option("starrocks.fe.jdbc.url", "jdbc:mysql://127.0.0.1:9030") \
            .option("starrocks.table.identifier", "test.score_board") \
            .option("starrocks.user", "root") \
            .option("starrocks.password", "") \
            // チェックポイントディレクトリに置き換えてください。
            .option("checkpointLocation", "/path/to/checkpoint") \
            .outputMode("append") \
            .start()
        )
    ```

3. StarRocksで結果をクエリします。

```SQL
MySQL [test]> select * from score_board;
+------+-----------+-------+
| id   | name      | score |
+------+-----------+-------+
|    4 | spark     |   100 |
|    3 | starrocks |   100 |
+------+-----------+-------+
2行セット (0.67秒)
```

### Spark SQLを使用したデータの書き込み

この例では、`INSERT INTO`を使用してデータを書き込む方法を示しています。この例は[Spark SQL CLI](https://spark.apache.org/docs/latest/sql-distributed-sql-engine-spark-sql-cli.html)を通じて実行できます。

1. `spark-sql`で以下の例を実行します。

   ```SQL
   -- 1. `starrocks`データソースを指定し、以下のオプションを設定してテーブルを作成します。
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

2. StarRocksで結果をクエリします。

```SQL
MySQL [test]> select * from score_board;
+------+-----------+-------+
| id   | name      | score |
+------+-----------+-------+
|    6 | spark     |   100 |
|    5 | starrocks |   100 |
+------+-----------+-------+
2行セット (0.00秒)
```

## ベストプラクティス

### 主キーモデルテーブルへのインポート

このセクションでは、部分更新と条件更新を実現するために、データをStarRocksの主キーモデルテーブルにインポートする方法を示します。部分更新と条件更新の詳細については、[インポートによるデータ変更](./Load_to_Primary_Key_tables.md)を参照してください。

以下の例はSpark SQLを使用しています。

#### 準備

StarRocksに`test`という名前のデータベースを作成し、その中に`score_board`という主キーモデルテーブルを作成します。

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

この例では、StarRocksテーブルの`name`列の値のみを更新するためにデータをインポートする方法を示しています。

1. MySQLクライアントでStarRocksテーブル`score_board`に2行のデータを挿入します。

   ```SQL
   mysql> INSERT INTO `score_board` VALUES (1, 'starrocks', 100), (2, 'spark', 100);

   mysql> select * from score_board;
   +------+-----------+-------+
   | id   | name      | score |
   +------+-----------+-------+
   |    1 | starrocks |   100 |
   |    2 | spark     |   100 |
   +------+-----------+-------+
   2行セット (0.02秒)
   ```

2. Spark SQLクライアントで`score_board`テーブルを作成します。
   - オプション`starrocks.write.properties.partial_update`を`true`に設定して、Spark connectorに部分更新を行うように要求します。
   - 更新する必要のある列をSpark connectorに伝えるために、オプション`starrocks.columns`を`id,name`に設定します。

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

3. Spark SQLクライアントで、2行のデータをテーブルに挿入します。データ行の主キーはStarRocksテーブルのデータ行の主キーと同じですが、`name`列の値が変更されています。

   ```SQL
   INSERT INTO `score_board` VALUES (1, 'starrocks-update'), (2, 'spark-update');
   ```

4. MySQLクライアントでStarRocksテーブルを照会します。

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

   `name`列の値のみが変更され、`score`列の値は変更されていないことがわかります。

#### 条件付き更新

この例では、`score`列の値に基づいて条件付き更新を行う方法を示しています。インポートされたデータ行の`score`列の値がStarRocksテーブルの現在の値以上の場合にのみ、そのデータ行が更新されます。

1. MySQLクライアントでStarRocksテーブルに2行のデータを挿入します。

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

2. Spark SQLクライアントで以下のように`score_board`テーブルを作成します：

   - オプション`starrocks.write.properties.merge_condition`を`score`に設定し、Spark connectorが更新条件として`score`列を使用するように要求します。
   - Spark connectorがデータをインポートする際にStream Loadインターフェースを使用し、Stream Loadトランザクションインターフェースを使用しないことを確認します。なぜなら、Stream Loadトランザクションインターフェースは条件付き更新をサポートしていないからです。

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

3. Spark SQLクライアントでテーブルに2行のデータを挿入します。データ行の主キーはStarRocksテーブル内の行と同じです。最初の行のデータでは`score`列の値が小さく、2行目のデータでは`score`列の値が大きくなっています。

   ```SQL
   INSERT INTO `score_board` VALUES (1, 'starrocks-update', 99), (2, 'spark-update', 101);
   ```

4. MySQLクライアントでStarRocksテーブルを照会します。

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

   2行目のデータのみが変更され、1行目のデータは変更されていないことに注意してください。

### BITMAP列へのインポート

`BITMAP`は、例えばユニークビジター数(UV)の計算など、正確な重複排除カウントを加速するためによく使用されます。詳細については、[Bitmapを使用した正確な重複排除](../using_starrocks/Using_bitmap.md)を参照してください。

この例では、ユニークビジター数(UV)の計算を例に、StarRocksテーブルの`BITMAP`列にデータをインポートする方法を示しています。**バージョン1.1.1から`BITMAP`列へのインポートがサポートされています**。

1. MySQLクライアントでStarRocks集約テーブルを作成します。

   `test`データベースに、`visit_users`列が`BITMAP`型で定義され、集約関数`BITMAP_UNION`が設定された集約テーブル`page_uv`を作成します。

    ```SQL
    CREATE TABLE `test`.`page_uv` (
      `page_id` INT NOT NULL COMMENT 'ページID',
      `visit_date` datetime NOT NULL COMMENT 'アクセス時間',
      `visit_users` BITMAP BITMAP_UNION NOT NULL COMMENT 'ユーザーID'
    ) ENGINE=OLAP
    AGGREGATE KEY(`page_id`, `visit_date`)
    DISTRIBUTED BY HASH(`page_id`);
    ```

2. Spark SQLクライアントでテーブルを作成します。

   SparkテーブルのスキーマはStarRocksテーブルから推測されますが、SparkはBITMAP型をサポートしていません。したがって、Sparkで対応する列のデータ型をカスタマイズする必要があります。例えば、オプション`"starrocks.column.types"="visit_users BIGINT"`を設定して、BIGINT型として設定します。Stream Loadを使用してデータをインポートする際に、Spark connectorは`to_bitmap`関数を使用してBIGINT型のデータをBITMAP型に変換します。

   `spark-sql`で以下のDDL文を実行します：

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

3. Spark SQLクライアントでテーブルにデータを挿入します。

   `spark-sql`で以下のDML文を実行します：

    ```SQL
    INSERT INTO `page_uv` VALUES
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 13),
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 23),
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 33),
       (1, CAST('2020-06-23 02:30:30' AS TIMESTAMP), 13),
       (2, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 23);
    ```

4. MySQLクライアントでStarRocksテーブルを照会し、ページのUV数を計算します。

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

> **注意**
>
> Sparkの該当列のデータ型がTINYINT、SMALLINT、INTEGER、またはBIGINTの場合、Spark connectorは[`to_bitmap`](../sql-reference/sql-functions/bitmap-functions/to_bitmap.md)関数を使用してStarRocksのBITMAP型に変換します。Sparkの該当列が他のデータ型の場合、Spark connectorは[`bitmap_hash`](../sql-reference/sql-functions/bitmap-functions/bitmap_hash.md)関数を使用して変換します。

### HLL列へのインポート

`HLL`は近似的な重複排除カウントに使用されます。詳細については、[HLLを使用した近似的な重複排除](../using_starrocks/Using_HLL.md)を参照してください。

この例では、ユニークビジター数(UV)の計算を例に、StarRocksテーブルの`HLL`列にデータをインポートする方法を示しています。**バージョン1.1.1から`HLL`列へのインポートがサポートされています**。

1. MySQLクライアントでStarRocks集約テーブルを作成します。

   `test`データベースに、`visit_users`列が`HLL`型で定義され、集約関数`HLL_UNION`が設定された集約テーブル`hll_uv`を作成します。

   ```SQL
   CREATE TABLE `hll_uv` (
   `page_id` INT NOT NULL COMMENT 'ページID',
   `visit_date` datetime NOT NULL COMMENT 'アクセス時間',
   `visit_users` HLL HLL_UNION NOT NULL COMMENT 'ユーザーID'
   ) ENGINE=OLAP
   AGGREGATE KEY(`page_id`, `visit_date`)
   DISTRIBUTED BY HASH(`page_id`);
   ```

2. Spark SQLクライアントでテーブルを作成します。

   SparkテーブルのスキーマはStarRocksテーブルから推測されますが、SparkはHLL型をサポートしていません。したがって、Sparkで対応する列のデータ型をカスタマイズする必要があります。例えば、オプション`"starrocks.column.types"="visit_users BIGINT"`を設定して、BIGINT型として設定します。Stream Loadを使用してデータをインポートする際に、Spark connectorは[hll_hash](../sql-reference/sql-functions/aggregate-functions/hll_hash.md)関数を使用してBIGINT型のデータをHLL型に変換します。

   `spark-sql`で以下のDDL文を実行します：

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

3. Spark SQLクライアントでテーブルにデータを挿入します。

   `spark-sql`で以下のDML文を実行します：

    ```SQL
    INSERT INTO `hll_uv` VALUES
       (3, CAST('2023-07-24 12:00:00' AS TIMESTAMP), 78),
       (4, CAST('2023-07-24 13:20:10' AS TIMESTAMP), 2),
       (3, CAST('2023-07-24 12:30:00' AS TIMESTAMP), 674);
    ```

4. MySQLクライアントでStarRocksテーブルを照会し、ページのUV数を計算します。

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

### ARRAY列へのインポート

以下の例は、ARRAY型の列にデータをインポートする方法を説明しています。

1. StarRocksテーブルを作成する

   `test`データベースに、`INT`型の列と2つの`ARRAY`型の列を含む主キーテーブル`array_tbl`を作成します。

   ```sql
   CREATE TABLE `array_tbl` (
     `id` INT NOT NULL,
     `a0` ARRAY<STRING>,
     `a1` ARRAY<ARRAY<INT>>
   ) ENGINE=OLAP
   PRIMARY KEY(`id`)
   DISTRIBUTED BY HASH(`id`);
   ```

2. StarRocksテーブルにデータを書き込む。

   StarRocksの一部のバージョンではARRAY列のメタデータが提供されていないため、Spark connectorは該当列のSparkデータ型を推測できません。しかし、オプション`starrocks.column.types`で列の対応するSparkデータ型を明示的に指定することができます。この例では、オプションを`a0 ARRAY<STRING>,a1 ARRAY<ARRAY<INT>>`として設定します。

   Scala言語を使用している場合は、`spark-shell`で以下のコードを実行できます：

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

3. StarRocksテーブルを照会します。

   ```SQL
   MySQL [test]> SELECT * FROM `array_tbl`;
   +------+-----------------------+--------------------+
   | id   | a0                    | a1                 |
   +------+-----------------------+--------------------+
   |    1 | ["hello","starrocks"] | [[1,2],[3,4]]      |
   |    2 | ["hello","spark"]     | [[5,6,7],[8,9,10]] |
   +------+-----------------------+--------------------+
   2 rows in set (0.01 sec)
   ```

