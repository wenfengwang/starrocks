---
displayed_sidebar: Chinese
---

# Spark Connector を使用してデータを読み取る

StarRocks は Apache Spark™ Connector（StarRocks Connector for Apache Spark™）を提供しており、Spark を通じて StarRocks に保存されているデータを読み取ることができます。読み取ったデータに対して複雑な処理や機械学習などを行うことができます。

Spark Connector は、Spark SQL、Spark DataFrame、Spark RDD の3種類のデータ読み取り方法をサポートしています。

Spark SQL を使用して StarRocks のテーブル上に一時ビューを作成し、そのビューを通じて StarRocks のテーブルデータを直接読み取ることができます。

また、StarRocks のテーブルを Spark DataFrame や Spark RDD としてマッピングし、それらからデータを読み取ることもできます。StarRocks に保存されているデータを読み取るには、Spark DataFrame の使用を推奨します。

## 使用説明

- StarRocks 側でデータフィルタリングを完了し、データ転送量を減少させることができます。

- データの読み取りコストが高い場合は、適切なテーブル設計とフィルタ条件の使用により、Spark が一度に多くのデータを読み取らないように制御し、ディスクやネットワークに過度の I/O 圧力をかけたり、通常のクエリ業務に影響を与えたりすることを避けることができます。

## バージョン要件

| Spark Connector | Spark         | StarRocks   | Java | Scala |
|---------------- | ------------- | ----------- | ---- | ----- |
| 1.1.1           | 3.2, 3.3, 3.4 | 2.5 以上     | 8    | 2.12  |
| 1.1.0           | 3.2, 3.3, 3.4 | 2.5 以上     | 8    | 2.12  |
| 1.0.0           | 3.x           | 1.18 以上    | 8    | 2.12  |
| 1.0.0           | 2.x           | 1.18 以上    | 8    | 2.11  |

> **注意**
>
> - 異なるバージョンの Spark Connector の挙動の変更については、[Spark Connector のアップグレード](#100-升级至-110)を参照してください。
> - バージョン 1.1.1 以降、Spark Connector は MySQL JDBC ドライバーを提供しなくなりました。ドライバーを手動で Spark のクラスパスに配置する必要があります。ドライバーは [MySQL 公式サイト](https://dev.mysql.com/downloads/connector/j/)または [Maven Central Repository](https://repo1.maven.org/maven2/mysql/mysql-connector-java/)で見つけることができます。
> - バージョン 1.0.0 は StarRocks の読み取りのみをサポートしており、バージョン 1.1.0 からは読み書きをサポートしています。
> - バージョン 1.0.0 と 1.1.0 ではパラメータと型のマッピングに違いがあります。詳細は [Spark Connector のアップグレード](#100-升级至-110)を参照してください。
> - バージョン 1.0.0 は通常、新機能を追加することはありません。条件が許せば、できるだけ早く Spark Connector をアップグレードしてください。

## Spark Connector の取得

以下の方法で Spark Connector の Jar パッケージを取得できます：

- すでにコンパイルされた Jar パッケージを直接ダウンロードします。
- Maven を使用して Spark Connector の依存関係を追加します（バージョン 1.1.0 以上のみサポート）。
- ソースコードから手動でコンパイルします。

### Spark Connector バージョン 1.1.0 以上

Spark Connector Jar パッケージの命名フォーマットは以下の通りです：

`starrocks-spark-connector-${spark_version}_${scala_version}-${connector_version}.jar`

例えば、Spark 3.2 と Scala 2.12 を使用してバージョン 1.1.0 の Spark Connector を使用したい場合は、`starrocks-spark-connector-3.2_2.12-1.1.0.jar` を選択できます。

> **注意**
>
> 通常、最新バージョンの Spark Connector は最新の3つのバージョンの Spark のみをメンテナンスします。

#### 直接ダウンロード

[Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks) で異なるバージョンの Spark Connector Jar パッケージを取得できます。

#### Maven 依存関係の追加

依存関係の設定フォーマットは以下の通りです：

> **注意**
>
> `spark_version`、`scala_version`、`connector_version` を対応するバージョンに置き換える必要があります。

```xml
<dependency>
  <groupId>com.starrocks</groupId>
  <artifactId>starrocks-spark-connector-${spark_version}_${scala_version}</artifactId>
  <version>${connector_version}</version>
</dependency>
```

例えば、Spark 3.2 と Scala 2.12 を使用してバージョン 1.1.0 の Spark Connector を使用したい場合は、以下の依存関係を追加できます：

```xml
<dependency>
  <groupId>com.starrocks</groupId>
  <artifactId>starrocks-spark-connector-3.2_2.12</artifactId>
  <version>1.1.0</version>
</dependency>
```

#### 手動でコンパイル

1. [Spark Connector のコード](https://github.com/StarRocks/starrocks-connector-for-apache-spark)をダウンロードします。

2. 以下のコマンドでコンパイルを実行します：

   > **注意**
   >
   > `spark_version` を対応する Spark のバージョンに置き換える必要があります。

   ```shell
   sh build.sh <spark_version>
   ```

   例えば、Spark 3.2 を使用して Spark Connector を使用したい場合は、以下のコマンドでコンパイルを実行できます：

   ```shell
   sh build.sh 3.2
   ```

3. コンパイルが完了したら、`target/` ディレクトリに移動して確認します。ディレクトリには Spark Connector の Jar パッケージが生成されます。例えば `starrocks-spark-connector-3.2_2.12-1.1.0-SNAPSHOT.jar`。

   > **注意**
   >
   > 非公式リリースの Spark Connector バージョンを使用している場合、生成された Spark Connector Jar パッケージ名には `SNAPSHOT` 接尾辞が付きます。

### Spark Connector バージョン 1.0.0

#### 直接ダウンロード

- [Spark 2.x 用](https://cdn-thirdparty.starrocks.com/spark/starrocks-spark2_2.11-1.0.0.jar)
- [Spark 3.x 用](https://cdn-thirdparty.starrocks.com/spark/starrocks-spark3_2.12-1.0.0.jar)

#### 手動でコンパイル

1. [Spark Connector のコード](https://github.com/StarRocks/starrocks-connector-for-apache-spark/tree/spark-1.0)をダウンロードします。

   > **注意**
   >
   > `spark-1.0` ブランチに切り替える必要があります。

2. 以下のコマンドで Spark Connector をコンパイルします：

   - Spark のバージョンが 2.x の場合は、以下のコマンドを実行します。デフォルトでは Spark 2.3.4 に対応する Spark Connector がコンパイルされます：

     ```Plain
     sh build.sh 2
     ```

   - Spark のバージョンが 3.x の場合は、以下のコマンドを実行します。デフォルトでは Spark 3.1.2 に対応する Spark Connector がコンパイルされます：

     ```Plain
     sh build.sh 3
     ```

3. コンパイルが完了したら、`output/` ディレクトリに移動して確認します。ディレクトリには `starrocks-spark2_2.11-1.0.0.jar` ファイルが生成されます。このファイルを Spark のクラスパス（Classpath）にコピーします：

   - Spark を `Local` モードで実行している場合は、`jars/` ディレクトリにこのファイルを配置する必要があります。
   - Spark を `Yarn` モードで実行している場合は、プリデプロイメントパッケージ（Pre-deployment Package）にこのファイルを含める必要があります。

ファイルを指定の場所に配置した後、Spark Connector を使用してデータを読み取り始めることができます。

## パラメータ説明

このセクションでは、Spark Connector を使用してデータを読み取る際に設定する必要があるパラメータについて説明します。

### 共通パラメータ

以下のパラメータは、Spark SQL、Spark DataFrame、Spark RDD の3つの読み取り方法に適用されます。

| パラメータ名                             | デフォルト値            | 説明                                                         |
| ------------------------------------ | ----------------- | ------------------------------------------------------------ |
| starrocks.fenodes                    | なし                | StarRocks クラスタ内の FE の HTTP アドレス。形式は `<fe_host>:<fe_http_port>`。複数のアドレスを入力する場合は、カンマ (,) で区切ります。 |
| starrocks.table.identifier           | なし                | StarRocks テーブルの名前。形式は `<database_name>.<table_name>`。 |
| starrocks.request.retries            | 3                 | Spark Connector が StarRocks に送信する読み取りリクエストのリトライ回数。 |
| starrocks.request.connect.timeout.ms | 30000             | 読み取りリクエストの接続確立のタイムアウト時間。 |
| starrocks.request.read.timeout.ms    | 30000             | StarRocks データの読み取りリクエストのタイムアウト時間。 |
| starrocks.request.query.timeout.s    | 3600              | StarRocks からデータをクエリするタイムアウト時間。デフォルトは1時間です。 |
| starrocks.request.tablet.size        | Integer.MAX_VALUE | 1つの Spark RDD パーティションに対応する StarRocks Tablet の数。パラメータを小さく設定するほど、生成されるパーティションが多くなり、Spark 側の並列度が高まりますが、同時に StarRocks 側に大きな負荷をかけることになります。 |
| starrocks.batch.size                 | 4096              | BE から一度に読み取る最大行数。パラメータの値を大きくすると、Spark と StarRocks 間で接続を確立する回数を減らし、ネットワーク遅延による追加の時間コストを軽減できます。StarRocks 2.2 以降のバージョンでは、サポートされる最小バッチサイズは 4096 です。この値より小さい設定を行った場合は、4096 として処理されます。 |
| starrocks.exec.mem.limit             | 2147483648        | 単一クエリのメモリ制限。単位はバイト。デフォルトのメモリ制限は 2 GB です。 |
| starrocks.deserialize.arrow.async    | false             | Arrow 形式を非同期で Spark Connector が必要とする RowBatch に変換するかどうか。 |
| starrocks.deserialize.queue.size     | 64                | Arrow 形式を非同期で変換する際の内部処理キューのサイズ。`starrocks.deserialize.arrow.async` が `true` の場合に有効です。 |
| starrocks.filter.query               | なし                | フィルタ条件を指定します。複数のフィルタ条件は `and` で接続します。StarRocks は指定されたフィルタ条件に基づいて読み取りデータのフィルタリングを行います。 |
| starrocks.timezone | JVM のデフォルトタイムゾーン | バージョン 1.1.1 以降でサポートされています。StarRocks のタイムゾーン。StarRocks の `DATETIME` 型の値を Spark の `TimestampType` 型の値に変換するために使用されます。デフォルトは JVM のデフォルトタイムゾーンである `ZoneId#systemDefault()` です。形式はタイムゾーン名（例：Asia/Shanghai）またはタイムゾーンオフセット（例：+08:00）です。 |

### Spark SQL および Spark DataFrame 専用パラメータ

以下のパラメータは、Spark SQL および Spark DataFrame の読み取り方法にのみ適用されます。

| パラメータ名                             | デフォルト値  | 説明                                                         |
| ----------------------------------- | ------ | ------------------------------------------------------------ |
| starrocks.fe.http.url               | なし     | FE の HTTP アドレス。Spark Connector バージョン 1.1.0 以降でサポートされており、`starrocks.fenodes` と同等です。どちらか一方を記入すれば十分です。Spark Connector バージョン 1.1.0 以降では、このパラメータの使用を推奨し、`starrocks.fenodes` は将来的に廃止される可能性があります。 |

| starrocks.fe.jdbc.url               | なし     | FEのMySQL Server接続アドレス。形式は `jdbc:mysql://<fe_host>:<fe_query_port>`。<br />**注意**<br />Spark Connector 1.1.0以降のバージョンでは、このパラメータは必須です。    |
| user                                | なし     | StarRocksクラスタのアカウントユーザー名。  |
| starrocks.user                      | なし     | StarRocksクラスタのアカウントユーザー名。Spark Connector 1.1.0バージョンからサポートされ、`user`と同等です。どちらか一方を記入すればよいです。Spark Connector 1.1.0以降のバージョンでは、このパラメータの使用を推奨し、`user`は将来的に廃止される可能性があります。   |
| password                            | なし     | StarRocksクラスタのアカウントパスワード。  |
| starrocks.password                  | なし     | StarRocksクラスタのアカウントパスワード。Spark Connector 1.1.0バージョンからサポートされ、`password`と同等です。どちらか一方を記入すればよいです。Spark Connector 1.1.0以降のバージョンでは、このパラメータの使用を推奨し、`password`は将来的に廃止される可能性があります。   |
| starrocks.filter.query.in.max.count | 100    | 谓词下推中、IN式がサポートする値の最大数。IN式に指定された値の数がこの上限を超える場合、IN式に指定された条件のフィルタリングはSpark側で処理されます。  |

### Spark RDD 専用パラメータ

以下のパラメータはSpark RDD読み取り方式にのみ適用されます。

| パラメータ名                        | デフォルト値 | 説明                                                         |
| ------------------------------- | ------ | ------------------------------------------------------------ |
| starrocks.request.auth.user     | なし     | StarRocksクラスタのアカウントユーザー名。                                 |
| starrocks.request.auth.password | なし     | StarRocksクラスタのアカウントパスワード。                               |
| starrocks.read.field            | なし     | StarRocksテーブルから読み取る列を指定します。複数の列名はコンマ(,)で区切ります。 |

## データ型マッピング関係

### Spark Connector 1.1.0 以上のバージョン

| StarRocksデータ型 | Sparkデータ型           |
|----------------- |-------------------------|
| BOOLEAN          | DataTypes.BooleanType   |
| TINYINT          | DataTypes.ByteType      |
| SMALLINT         | DataTypes.ShortType     |
| INT              | DataTypes.IntegerType   |
| BIGINT           | DataTypes.LongType      |
| LARGEINT         | DataTypes.StringType    |
| FLOAT            | DataTypes.FloatType     |
| DOUBLE           | DataTypes.DoubleType    |
| DECIMAL          | DecimalType             |
| CHAR             | DataTypes.StringType    |
| VARCHAR          | DataTypes.StringType    |
| STRING           | DataTypes.StringType    |
| DATE             | DataTypes.DateType      |
| DATETIME         | DataTypes.TimestampType |
| ARRAY            | Unsupported datatype    |
| HLL              | Unsupported datatype    |
| BITMAP           | Unsupported datatype    |

### Spark Connector 1.0.0 バージョン

| StarRocksデータ型  | Sparkデータ型          |
| ------------------ | --------------------- |
| BOOLEAN            | DataTypes.BooleanType |
| TINYINT            | DataTypes.ByteType    |
| SMALLINT           | DataTypes.ShortType   |
| INT                | DataTypes.IntegerType |
| BIGINT             | DataTypes.LongType    |
| LARGEINT           | DataTypes.StringType  |
| FLOAT              | DataTypes.FloatType   |
| DOUBLE             | DataTypes.DoubleType  |
| DECIMAL            | DecimalType           |
| CHAR               | DataTypes.StringType  |
| VARCHAR            | DataTypes.StringType  |
| DATE               | DataTypes.StringType  |
| DATETIME           | DataTypes.StringType  |
| ARRAY              | Unsupported datatype  |
| HLL                | Unsupported datatype  |
| BITMAP             | Unsupported datatype  |

Spark Connectorでは、DATEとDATETIMEデータ型をSTRINGデータ型にマッピングします。これは、StarRocksの基盤となるストレージエンジンの処理ロジックにより、直接DATEとDATETIMEデータ型を使用した場合、カバーする時間範囲が要件を満たさないためです。そのため、STRINGデータ型を使用して対応する時間の読み取り可能なテキストを直接返します。

## Spark Connector アップグレード

### 1.0.0から1.1.0へのアップグレード

- 1.1.1バージョンから、Spark ConnectorはMySQL公式JDBCドライバ`mysql-connector-java`を提供しなくなりました。このドライバはGPLライセンスを使用しており、いくつかの制限があります。しかし、Spark Connectorは依然としてMySQL JDBCドライバが必要です。StarRocksに接続してテーブルのメタデータを取得するため、手動でドライバをSparkクラスパスに追加する必要があります。このドライバは[MySQL公式サイト](https://dev.mysql.com/downloads/connector/j/)または[Maven中央リポジトリ](https://repo1.maven.org/maven2/mysql/mysql-connector-java/)で見つけることができます。

- 1.1.0バージョンでは、JDBCを介してStarRocksにアクセスし、より詳細なテーブル情報を取得する必要があるため、`starrocks.fe.jdbc.url`を設定する必要があります。

- 1.1.0バージョンでは、いくつかのパラメータ名が変更されました。現在、変更前後のパラメータが両方とも保持されていますが、一方を設定するだけでよく、新しいパラメータの使用を推奨します。古いパラメータは将来的に廃止される可能性があります：
  - `starrocks.fenodes`は`starrocks.fe.http.url`に変更されました。
  - `user`は`starrocks.user`に変更されました。
  - `password`は`starrocks.password`に変更されました。

- 1.1.0バージョンはSpark 3.xに基づいていくつかの型マッピングを調整しました：
  - StarRocksの`DATE`はSparkの`DataTypes.DateType`に、以前は`DataTypes.StringType`でした。
  - StarRocksの`DATETIME`はSparkの`DataTypes.TimestampType`に、以前は`DataTypes.StringType`でした。

## 使用例

StarRocksクラスタに`test`データベースが既に作成されており、`root`アカウントの権限を持っていると仮定します。例のパラメータ設定はSpark Connector 1.1.0バージョンに基づいています。

### データサンプル

以下の手順でデータサンプルを準備します：

1. `test`データベースに入り、`score_board`という名前のテーブルを作成します。

   ```SQL
   MySQL [test]> CREATE TABLE `score_board`
   (
       `id` int(11) NOT NULL COMMENT "",
       `name` varchar(65533) NULL DEFAULT "" COMMENT "",
       `score` int(11) NOT NULL DEFAULT "0" COMMENT ""
   )
   ENGINE=OLAP
   PRIMARY KEY(`id`)
   COMMENT "OLAP"
   DISTRIBUTED BY HASH(`id`)
   PROPERTIES (
       "replication_num" = "3"
   );
   ```

2. `score_board`テーブルにデータを挿入します。

   ```SQL
   MySQL [test]> INSERT INTO score_board
   VALUES
       (1, 'Bob', 21),
       (2, 'Stan', 21),
       (3, 'Sam', 22),
       (4, 'Tony', 22),
       (5, 'Alice', 22),
       (6, 'Lucy', 23),
       (7, 'Polly', 23),
       (8, 'Tom', 23),
       (9, 'Rose', 24),
       (10, 'Jerry', 24),
       (11, 'Jason', 24),
       (12, 'Lily', 25),
       (13, 'Stephen', 25),
       (14, 'David', 25),
       (15, 'Eddie', 26),
       (16, 'Kate', 27),
       (17, 'Cathy', 27),
       (18, 'Judy', 27),
       (19, 'Julia', 28),
       (20, 'Robert', 28),
       (21, 'Jack', 29);
   ```

3. `score_board`テーブルのデータを照会します。

   ```SQL
   MySQL [test]> SELECT * FROM score_board;
   +------+---------+-------+
   | id   | name    | score |
   +------+---------+-------+
   |    1 | Bob     |    21 |
   |    2 | Stan    |    21 |
   |    3 | Sam     |    22 |
   |    4 | Tony    |    22 |
   |    5 | Alice   |    22 |
   |    6 | Lucy    |    23 |
   |    7 | Polly   |    23 |
   |    8 | Tom     |    23 |
   |    9 | Rose    |    24 |
   |   10 | Jerry   |    24 |
   |   11 | Jason   |    24 |
   |   12 | Lily    |    25 |
   |   13 | Stephen |    25 |
   |   14 | David   |    25 |
   |   15 | Eddie   |    26 |
   |   16 | Kate    |    27 |
   |   17 | Cathy   |    27 |
   |   18 | Judy    |    27 |
   |   19 | Julia   |    28 |
   |   20 | Robert  |    28 |
   |   21 | Jack    |    29 |
   +------+---------+-------+
   21行セット (0.01秒)
   ```

### Spark SQLを使用してデータを読み取る

1. Sparkの実行可能プログラムディレクトリに入り、以下のコマンドを実行します：

   ```Plain
   sh spark-sql
   ```

2. 以下のコマンドを実行し、データベース`test`のテーブル`score_board`に`spark_starrocks`という名前の一時ビューを作成します：

   ```SQL
   spark-sql> CREATE TEMPORARY VIEW spark_starrocks
              USING starrocks
              OPTIONS
              (
                  "starrocks.table.identifier" = "test.score_board",
                  "starrocks.fe.http.url" = "<fe_host>:<fe_http_port>",
                  "starrocks.fe.jdbc.url" = "jdbc:mysql://<fe_host>:<fe_query_port>",
                  "starrocks.user" = "root",
                  "starrocks.password" = ""
              );
   ```

3. 以下のコマンドを実行し、一時ビューからデータを読み取ります：

   ```SQL
   spark-sql> SELECT * FROM spark_starrocks;
   ```

   戻りデータは以下の通りです：

   ```SQL
   1        Bob        21
   2        Stan        21
   3        Sam        22
   4        Tony        22
   5        Alice        22
   6        Lucy        23
   7        Polly        23
   8        Tom        23
   9        Rose        24
   10        Jerry        24
   11        Jason        24
   12        Lily        25
   13        Stephen        25
   14        David        25
   15        Eddie        26
   16        Kate        27
   17        Cathy        27
   18        Judy        27
   19        Julia        28
   20        Robert        28
   21        Jack        29
   処理時間: 1.883秒, 取得行数: 21行
   22/08/09 15:29:36 INFO thriftserver.SparkSQLCLIDriver: 処理時間: 1.883秒, 取得行数: 21行
   ```

### Spark DataFrameを使用してデータを読み取る


1. Sparkの実行可能プログラムディレクトリに移動し、以下のコマンドを実行します：

   ```Plain
   sh spark-shell
   ```

2. 以下のコマンドを実行し、StarRocksデータベース`test`のテーブル`score_board`を基に、`starrocksSparkDF`という名前のDataFrameをSparkで作成します：

   ```Scala
   scala> val starrocksSparkDF = spark.read.format("starrocks")
              .option("starrocks.table.identifier", "test.score_board")
              .option("starrocks.fe.http.url", "<fe_host>:<fe_http_port>")
              .option("starrocks.fe.jdbc.url", "jdbc:mysql://<fe_host>:<fe_query_port>")
              .option("starrocks.user", "root")
              .option("starrocks.password", "")
              .load()
   ```

3. DataFrameからデータを読み取ります。例えば、最初の10行のデータを読み取るには、以下のコマンドを実行します：

   ```Scala
   scala> starrocksSparkDF.show(10)
   ```

   返されるデータは以下の通りです：

   ```Scala
   +---+-----+-----+
   | id| name|score|
   +---+-----+-----+
   |  1|  Bob|   21|
   |  2| Stan|   21|
   |  3|  Sam|   22|
   |  4| Tony|   22|
   |  5|Alice|   22|
   |  6| Lucy|   23|
   |  7|Polly|   23|
   |  8|  Tom|   23|
   |  9| Rose|   24|
   | 10|Jerry|   24|
   +---+-----+-----+
   上位10行のみ表示
   ```

  > **説明**
  >
  > 読み取る行数を指定しない場合、デフォルトで最初の20行のデータが読み取られます。

### Spark RDDを使用してデータを読み取る

1. Sparkの実行可能プログラムディレクトリに移動し、以下のコマンドを実行します：

   ```Plain
   sh spark-shell
   ```

2. 以下のコマンドを実行し、StarRocksデータベース`test`のテーブル`score_board`を基に、`starrocksSparkRDD`という名前のRDDをSparkで作成します：

   ```Scala
   scala> import com.starrocks.connector.spark._
   scala> val starrocksSparkRDD = sc.starrocksRDD(
              tableIdentifier = Some("test.score_board"),
              cfg = Some(Map(
                  "starrocks.fenodes" -> "<fe_host>:<fe_http_port>",
                  "starrocks.request.auth.user" -> "root",
                  "starrocks.request.auth.password" -> ""
              ))
              )
   ```

3. RDDからデータを読み取ります。例えば、最初の10個の要素(Element)のデータを読み取るには、以下のコマンドを実行します：

   ```Scala
   scala> starrocksSparkRDD.take(10)
   ```

   返されるデータは以下の通りです：

   ```Scala
   res0: Array[AnyRef] = Array([1, Bob, 21], [2, Stan, 21], [3, Sam, 22], [4, Tony, 22], [5, Alice, 22], [6, Lucy, 23], [7, Polly, 23], [8, Tom, 23], [9, Rose, 24], [10, Jerry, 24])
   ```

   RDD全体を読み取るには、以下のコマンドを実行します：

   ```Scala
   scala> starrocksSparkRDD.collect()
   ```

   返されるデータは以下の通りです：

   ```Scala
   res1: Array[AnyRef] = Array([1, Bob, 21], [2, Stan, 21], [3, Sam, 22], [4, Tony, 22], [5, Alice, 22], [6, Lucy, 23], [7, Polly, 23], [8, Tom, 23], [9, Rose, 24], [10, Jerry, 24], [11, Jason, 24], [12, Lily, 25], [13, Stephen, 25], [14, David, 25], [15, Eddie, 26], [16, Kate, 27], [17, Cathy, 27], [18, Judy, 27], [19, Julia, 28], [20, Robert, 28], [21, Jack, 29])
   ```

## ベストプラクティス

StarRocksからデータを読み取る際にSpark Connectorを使用すると、`starrocks.filter.query`パラメータを指定してフィルタ条件を設定し、合理的なパーティション、バケット、プレフィックスインデックスのトリミングを行い、データの取得コストを削減できます。ここではSpark DataFrameを例にして説明し、実行計画を確認して実際のデータトリミング効果を検証します。

### 環境設定

| コンポーネント    | バージョン                                                   |
| --------------- | ------------------------------------------------------------ |
| Spark           | Spark 2.4.4 と Scala 2.11.12 (OpenJDK 64-Bit Server VM, Java 1.8.0_302) |
| StarRocks       | 2.2.0                                                       |
| Spark Connector | starrocks-spark2_2.11-1.0.0.jar                              |

### データサンプル

以下の手順でデータサンプルを準備します：

1. `test`データベースに入り、`mytable`という名前のテーブルを作成します。

   ```SQL
   MySQL [test]> CREATE TABLE `mytable`
   (
       `k` int(11) NULL COMMENT "bucket",
       `b` int(11) NULL COMMENT "",
       `dt` datetime NULL COMMENT "",
       `v` int(11) NULL COMMENT ""
   )
   ENGINE=OLAP
   DUPLICATE KEY(`k`,`b`, `dt`)
   COMMENT "OLAP"
   PARTITION BY RANGE(`dt`)
   (
       PARTITION p202201 VALUES [('2022-01-01 00:00:00'), ('2022-02-01 00:00:00')),
       PARTITION p202202 VALUES [('2022-02-01 00:00:00'), ('2022-03-01 00:00:00')),
       PARTITION p202203 VALUES [('2022-03-01 00:00:00'), ('2022-04-01 00:00:00'))
   )
   DISTRIBUTED BY HASH(`k`)
   PROPERTIES (
       "replication_num" = "3"
   );
   ```

2. `mytable`テーブルにデータを挿入します。

   ```SQL
   MySQL [test]> INSERT INTO mytable
   VALUES
        (1, 11, '2022-01-02 08:00:00', 111),
        (2, 22, '2022-02-02 08:00:00', 222),
        (3, 33, '2022-03-02 08:00:00', 333);
   ```

3. `mytable`テーブルのデータを照会します。

   ```SQL
   MySQL [test]> select * from mytable;
   +------+------+---------------------+------+
   | k    | b    | dt                  | v    |
   +------+------+---------------------+------+
   |    1 |   11 | 2022-01-02 08:00:00 |  111 |
   |    2 |   22 | 2022-02-02 08:00:00 |  222 |
   |    3 |   33 | 2022-03-02 08:00:00 |  333 |
   +------+------+---------------------+------+
   3行セット (0.01秒)
   ```

### フルテーブルスキャン

1. Spark実行可能プログラムディレクトリで、以下のコマンドを実行し、データベース`test`のテーブル`mytable`を基に`df`という名前のDataFrameを作成します：

   ```Scala
   scala> val df = spark.read.format("starrocks")
           .option("starrocks.table.identifier", "test.mytable")
           .option("starrocks.fenodes", "<fe_host>:<fe_http_port>")
           .option("user", "root")
           .option("password", "")
           .load()
   ```

2. StarRocksのFEログファイル**fe.log**を確認し、Sparkがデータを読み取るために使用したSQL文を見つけます。以下のように表示されます：

   ```SQL
   2022-08-09 18:57:38,091 INFO (nioEventLoopGroup-3-10|196) [TableQueryPlanAction.executeWithoutPassword():126] receive SQL statement [select `k`,`b`,`dt`,`v` from `test`.`mytable`] from external service [ user ['root'@'%']] for database [test] table [mytable]
   ```

3. データベース`test`で、SELECT `k`,`b`,`dt`,`v` from `test`.`mytable`の実行計画を取得するためにEXPLAINを使用します。以下のように表示されます：

   ```Scala
   MySQL [test]> EXPLAIN select `k`,`b`,`dt`,`v` from `test`.`mytable`;
   +-----------------------------------------------------------------------+
   | Explain String                                                        |
   +-----------------------------------------------------------------------+
   | PLAN FRAGMENT 0                                                       |
   |  OUTPUT EXPRS:1: k | 2: b | 3: dt | 4: v                              |
   |   PARTITION: UNPARTITIONED                                            |
   |                                                                       |
   |   RESULT SINK                                                         |
   |                                                                       |
   |   1:EXCHANGE                                                          |
   |                                                                       |
   | PLAN FRAGMENT 1                                                       |
   |  OUTPUT EXPRS:                                                        |
   |   PARTITION: RANDOM                                                   |
   |                                                                       |
   |   STREAM DATA SINK                                                    |
   |     EXCHANGE ID: 01                                                   |
   |     UNPARTITIONED                                                     |
   |                                                                       |
   |   0:OlapScanNode                                                      |
   |      TABLE: mytable                                                   |
   |      PREAGGREGATION: ON                                               |
   |      partitions=3/3                                                   |
   |      rollup: mytable                                                  |
   |      tabletRatio=9/9                                                  |
   |      tabletList=41297,41299,41301,41303,41305,41307,41309,41311,41313 |
   |      cardinality=3                                                    |
   |      avgRowSize=4.0                                                   |
   |      numNodes=0                                                       |
   +-----------------------------------------------------------------------+
   26行セット (0.00秒)
   ```

ここではトリミングが行われていません。したがって、データを含む3つのパーティション(`partitions=3/3`)と、これらのパーティション内の全9つのタブレット(`tabletRatio=9/9`)がスキャンされます。

### パーティショントリミング

1. Spark実行可能プログラムディレクトリで、以下のコマンドを実行し、データベース`test`のテーブル`mytable`を基に`df`という名前のDataFrameを作成します。このコマンドでは`starrocks.filter.query`パラメータを使用してフィルタ条件`dt='2022-01-02 08:00:00'`を指定し、パーティショントリミングを行います：

   ```Scala
   scala> val df = spark.read.format("starrocks")
          .option("starrocks.table.identifier", "test.mytable")
          .option("starrocks.fenodes", "<fe_host>:<fe_http_port>")
          .option("user", "root")
          .option("password", "")
          .option("starrocks.filter.query", "dt='2022-01-02 08:00:00'")
          .load()
   ```

2. StarRocksのFEログファイル**fe.log**を確認し、Sparkがデータを読み取るために使用したSQL文を見つけます。以下のように表示されます：

   ```SQL
   2022-08-09 19:02:31,253 INFO (nioEventLoopGroup-3-14|204) [TableQueryPlanAction.executeWithoutPassword():126] receive SQL statement [select `k`,`b`,`dt`,`v` from `test`.`mytable` where dt='2022-01-02 08:00:00'] from external service [ user ['root'@'%']] for database [test] table [mytable]
   ```

3. データベース`test`で、SELECT `k`,`b`,`dt`,`v` from `test`.`mytable` where dt='2022-01-02 08:00:00'の実行計画を取得するためにEXPLAINを使用します。以下のように表示されます：

   ```Scala
   MySQL [test]> EXPLAIN select `k`,`b`,`dt`,`v` from `test`.`mytable` where dt='2022-01-02 08:00:00';
   +------------------------------------------------+
   | Explain String                                 |
   +------------------------------------------------+
   | PLAN FRAGMENT 0                                |
   |  OUTPUT EXPRS:1: k | 2: b | 3: dt | 4: v       |
   |   PARTITION: UNPARTITIONED                     |
   |                                                |
   |   RESULT SINK                                  |
   |                                                |
   |   1:EXCHANGE                                   |
   |                                                |
   | PLAN FRAGMENT 1                                |
   |  OUTPUT EXPRS:                                 |
   |   PARTITION: RANDOM                            |
   |                                                |
   |   STREAM DATA SINK                             |
   |     EXCHANGE ID: 01                            |
   |     UNPARTITIONED                              |
   |                                                |
   |   0:OlapScanNode                               |
   |      TABLE: mytable                            |
   |      PREAGGREGATION: ON                        |
   |      PREDICATES: 3: dt = '2022-01-02 08:00:00' |
   |      partitions=1/3                            |
   |      rollup: mytable                           |
   |      tabletRatio=3/3                           |
   |      tabletList=41297,41299,41301              |
   |      cardinality=1                             |
   |      avgRowSize=20.0                           |
   |      numNodes=0                                |
   +------------------------------------------------+
   27 rows in set (0.01 sec)
   ```

ここではパーティションのみのプルーニングが行われ、バケットプルーニングは行われていません。そのため、3つのパーティションのうち1つのパーティション(`partitions=1/3`)と、そのパーティションに含まれるすべてのタブレット(`tabletRatio=3/3`)がスキャンされます。

### バケットプルーニング

1. Spark 実行可能プログラムのディレクトリで、以下のコマンドを使用して、`test`データベースの`mytable`テーブルに基づいて`df`という名前のDataFrameを作成します。コマンドでは`starrocks.filter.query`パラメータを使用してフィルタ条件`k=1`を指定し、バケットプルーニングを行います：

   ```Scala
   scala> val df = spark.read.format("starrocks")
          .option("starrocks.table.identifier", "test.mytable")
          .option("starrocks.fenodes", "<fe_host>:<fe_http_port>")
          .option("user", "root")
          .option("password", "")
          .option("starrocks.filter.query", "k=1")
          .load()
   ```

2. StarRocksのFEログファイル **fe.log** を確認し、Sparkがデータを読み込むために使用したSQL文を見つけます。以下のように表示されます：

   ```SQL
   2022-08-09 19:04:44,479 INFO (nioEventLoopGroup-3-16|208) [TableQueryPlanAction.executeWithoutPassword():126] receive SQL statement [select `k`,`b`,`dt`,`v` from `test`.`mytable` where k=1] from external service [ user ['root'@'%']] for database [test] table [mytable]
   ```

3. `test`データベースでEXPLAINを使用して、SELECT `k`,`b`,`dt`,`v` from `test`.`mytable` where k=1の実行計画を取得します。以下のように表示されます：

   ```Scala
   MySQL [test]> EXPLAIN select `k`,`b`,`dt`,`v` from `test`.`mytable` where k=1;
   +------------------------------------------+
   | Explain String                           |
   +------------------------------------------+
   | PLAN FRAGMENT 0                          |
   |  OUTPUT EXPRS:1: k | 2: b | 3: dt | 4: v |
   |   PARTITION: UNPARTITIONED               |
   |                                          |
   |   RESULT SINK                            |
   |                                          |
   |   1:EXCHANGE                             |
   |                                          |
   | PLAN FRAGMENT 1                          |
   |  OUTPUT EXPRS:                           |
   |   PARTITION: RANDOM                      |
   |                                          |
   |   STREAM DATA SINK                       |
   |     EXCHANGE ID: 01                      |
   |     UNPARTITIONED                        |
   |                                          |
   |   0:OlapScanNode                         |
   |      TABLE: mytable                      |
   |      PREAGGREGATION: ON                  |
   |      PREDICATES: 1: k = 1                |
   |      partitions=3/3                      |
   |      rollup: mytable                     |
   |      tabletRatio=3/9                     |
   |      tabletList=41299,41305,41311        |
   |      cardinality=1                       |
   |      avgRowSize=20.0                     |
   |      numNodes=0                          |
   +------------------------------------------+
   27 rows in set (0.01 sec)
   ```

ここではパーティションプルーニングは行われず、バケットプルーニングのみが行われました。そのため、データを含むすべての3つのパーティション(`partitions=3/3`)と、これら3つのパーティションの中で`k = 1`のハッシュ値に一致するすべての3つのタブレット(`tabletRatio=3/9`)がスキャンされます。

### パーティションとバケットのプルーニング

1. Spark 実行可能プログラムのディレクトリで、以下のコマンドを使用して、`test`データベースの`mytable`テーブルに基づいて`df`という名前のDataFrameを作成します。コマンドでは`starrocks.filter.query`パラメータを使用してフィルタ条件`k=7`と`dt='2022-01-02 08:00:00'`を指定し、パーティションとバケットのプルーニングを行います：

   ```Scala
   scala> val df = spark.read.format("starrocks")
          .option("starrocks.table.identifier", "test.mytable")
          .option("starrocks.fenodes", "<fe_host>:<fe_http_port>")
          .option("user", "")
          .option("password", "")
          .option("starrocks.filter.query", "k=7 and dt='2022-01-02 08:00:00'")
          .load()
   ```

2. StarRocksのFEログファイル **fe.log** を確認し、Sparkがデータを読み込むために使用したSQL文を見つけます。以下のように表示されます：

   ```SQL
   2022-08-09 19:06:34,939 INFO (nioEventLoopGroup-3-18|212) [TableQueryPlanAction.executeWithoutPassword():126] receive SQL statement [select `k`,`b`,`dt`,`v` from `test`.`mytable` where k=7 and dt='2022-01-02 08:00:00'] from external service [ user ['root'@'%']] for database [test] table [mytable]
   ```

3. `test`データベースでEXPLAINを使用して、SELECT `k`,`b`,`dt`,`v` from `test`.`mytable` where k=7 and dt='2022-01-02 08:00:00'の実行計画を取得します。以下のように表示されます：

   ```Scala
   MySQL [test]> EXPLAIN select `k`,`b`,`dt`,`v` from `test`.`mytable` where k=7 and dt='2022-01-02 08:00:00';
   +----------------------------------------------------------+
   | Explain String                                           |
   +----------------------------------------------------------+
   | PLAN FRAGMENT 0                                          |
   |  OUTPUT EXPRS:1: k | 2: b | 3: dt | 4: v                 |
   |   PARTITION: RANDOM                                      |
   |                                                          |
   |   RESULT SINK                                            |
   |                                                          |
   |   0:OlapScanNode                                         |
   |      TABLE: mytable                                      |
   |      PREAGGREGATION: ON                                  |
   |      PREDICATES: 1: k = 7, 3: dt = '2022-01-02 08:00:00' |
   |      partitions=1/3                                      |
   |      rollup: mytable                                     |
   |      tabletRatio=1/3                                     |
   |      tabletList=41301                                    |
   |      cardinality=1                                       |
   |      avgRowSize=20.0                                     |
   |      numNodes=0                                          |
   +----------------------------------------------------------+
   17 rows in set (0.00 sec)
   ```

ここではパーティションプルーニングとバケットプルーニングが同時に行われました。そのため、3つのパーティションのうち1つのパーティション(`partitions=1/3`)と、そのパーティションに含まれる1つのタブレット(`tabletRatio=1/3`)のみがスキャンされます。

### プレフィックスインデックスフィルタリング

1. 1つのパーティションにさらに多くのデータを挿入します。以下のようになります：

   ```Scala
   MySQL [test]> INSERT INTO mytable
   VALUES
       (1, 11, "2022-01-02 08:00:00", 111), 
       (3, 33, "2022-01-02 08:00:00", 333), 
       (3, 33, "2022-01-02 08:00:00", 333), 
       (3, 33, "2022-01-02 08:00:00", 333);
   ```

2. `mytable`テーブルのデータをクエリします。以下のようになります：

   ```Scala
   MySQL [test]> SELECT * FROM mytable;
   +------+------+---------------------+------+
   | k    | b    | dt                  | v    |
   +------+------+---------------------+------+
   |    1 |   11 | 2022-01-02 08:00:00 |  111 |
   |    1 |   11 | 2022-01-02 08:00:00 |  111 |
   |    3 |   33 | 2022-01-02 08:00:00 |  333 |
   |    3 |   33 | 2022-01-02 08:00:00 |  333 |
   |    3 |   33 | 2022-01-02 08:00:00 |  333 |
   |    2 |   22 | 2022-02-02 08:00:00 |  222 |
   |    3 |   33 | 2022-03-02 08:00:00 |  333 |
   +------+------+---------------------+------+
   7 rows in set (0.01 sec)
   ```

3. Spark 実行可能プログラムのディレクトリで、以下のコマンドを使用して、`test`データベースの`mytable`テーブルに基づいて`df`という名前のDataFrameを作成します。コマンドでは`starrocks.filter.query`パラメータを使用してフィルタ条件`k=1`を指定し、プレフィックスインデックスフィルタリングを行います：

   ```Scala
   scala> val df = spark.read.format("starrocks")
          .option("starrocks.table.identifier", "test.mytable")
          .option("starrocks.fenodes", "<fe_host>:<fe_http_port>")
          .option("user", "root")
          .option("password", "")
          .option("starrocks.filter.query", "k=1")
          .load()
   ```

4. `test`データベースでプロファイルレポートを有効にします：

   ```SQL
   MySQL [test]> SET enable_profile = true;
   Query OK, 0 rows affected (0.00 sec)
   ```

5. ブラウザで `http://<fe_host>:<http_http_port>/query` ページを開き、SELECT * FROM mytable where k=1 のプロファイルを確認します。以下のようになります：

   ```SQL
   OLAP_SCAN (plan_node_id=0):
     CommonMetrics:
        - CloseTime: 1.255ms
        - OperatorTotalTime: 1.404ms
        - PeakMemoryUsage: 0.00 
        - PullChunkNum: 8
        - PullRowNum: 2
          - __MAX_OF_PullRowNum: 2
          - __MIN_OF_PullRowNum: 0
        - PullTotalTime: 148.60us
        - PushChunkNum: 0
        - PushRowNum: 0
        - PushTotalTime: 0ns
        - SetFinishedTime: 136ns
        - SetFinishingTime: 129ns
     UniqueMetrics:
        - Predicates: 1: k = 1
        - Rollup: mytable
        - Table: mytable
        - BytesRead: 88.00 B
          - __MAX_OF_BytesRead: 88.00 B
          - __MIN_OF_BytesRead: 0.00 
        - CachedPagesNum: 0
        - CompressedBytesRead: 844.00 B
          - __MAX_OF_CompressedBytesRead: 844.00 B
          - __MIN_OF_CompressedBytesRead: 0.00 
        - CreateSegmentIter: 18.582us
        - IOTime: 4.425us
        - LateMaterialize: 17.385us
        - PushdownPredicates: 3
        - RawRowsRead: 2
          - __MAX_OF_RawRowsRead: 2
          - __MIN_OF_RawRowsRead: 0
        - ReadPagesNum: 12
          - __MAX_OF_ReadPagesNum: 12
          - __MIN_OF_ReadPagesNum: 0
        - RowsRead: 2
          - __MAX_OF_RowsRead: 2
          - __MIN_OF_RowsRead: 0
        - ScanTime: 154.367us
        - SegmentInit: 95.903us
          - BitmapIndexFilter: 0ns
          - BitmapIndexFilterRows: 0
          - BloomFilterFilterRows: 0
          - ShortKeyFilterRows: 3
            - __MAX_OF_ShortKeyFilterRows: 3
            - __MIN_OF_ShortKeyFilterRows: 0
          - ZoneMapIndexFilterRows: 0
        - SegmentRead: 2.559us
          - BlockFetch: 2.187us
          - BlockFetchCount: 2
            - __MAX_OF_BlockFetchCount: 2
            - __MIN_OF_BlockFetchCount: 0
          - BlockSeek: 7.789us
          - BlockSeekCount: 2
            - __MAX_OF_BlockSeekCount: 2
            - __MIN_OF_BlockSeekCount: 0
          - ChunkCopy: 25ns
          - DecompressT: 0ns
          - DelVecFilterRows: 0
          - IndexLoad: 0ns
          - PredFilter: 353ns
          - PredFilterRows: 0
          - RowsetsReadCount: 7
          - SegmentsReadCount: 3
            - __MAX_OF_SegmentsReadCount: 2
            - __MIN_OF_SegmentsReadCount: 0
          - TotalColumnsDataPageCount: 8
            - __MAX_OF_TotalColumnsDataPageCount: 8
            - __MIN_OF_TotalColumnsDataPageCount: 0
        - UncompressedBytesRead: 508.00 B
          - __MAX_OF_UncompressedBytesRead: 508.00 B
          - __MIN_OF_UncompressedBytesRead: 0.00 B
   ```

`k = 1` はプレフィックスインデックスをヒットするため、データ読み取りプロセス中に 3 行をフィルタリングします (`ShortKeyFilterRows: 3`)。

