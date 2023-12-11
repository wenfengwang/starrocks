---
displayed_sidebar: "Japanese"
---

# Spark connectorを使用してStarRocksからデータを読む

StarRocksは、Apache Spark™（以下、Spark connector）向けのStarRocks Connectorという自社開発のコネクタを提供し、これを使用してStarRocksのテーブルからデータを読むための支援を行っています。Sparkを使用して、StarRocksから読み込んだデータに対して複雑な処理や機械学習を行うことができます。

Spark connectorでは、Spark SQL、Spark DataFrame、Spark RDDの3つの読み込み方法がサポートされています。

Spark SQLを使用して、StarRocksのテーブルに一時ビューを作成し、その一時ビューを使用してStarRocksのテーブルから直接データを読むことができます。

また、StarRocksのテーブルをSpark DataFrameまたはSpark RDDにマップし、その後Spark DataFrameまたはSpark RDDからデータを読むことができます。Spark DataFrameの使用を推奨します。

> **注意**
>
> StarRocksのテーブルからデータを読むには、そのテーブルに対するSELECT権限があるユーザーのみが可能です。[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)で権限をユーザーに付与する手順に従ってください。

## 使用上の注意

- データを読む前にStarRocksでデータをフィルタリングすることで、転送されるデータ量を削減することができます。
- データの読み込みにかかるオーバーヘッドが大きい場合は、適切なテーブル設計とフィルタ条件を使用して、一度に過剰なデータをSparkが読み込むのを防ぐことができます。これにより、ディスクやネットワーク接続へのI/O圧力を軽減し、通常のクエリが正常に実行されることを保証できます。

## バージョン要件

| Spark connector | Spark         | StarRocks       | Java | Scala |
|---------------- | ------------- | --------------- | ---- | ----- |
| 1.1.1           | 3.2, 3.3, 3.4 | 2.5 以降       | 8    | 2.12  |
| 1.1.0           | 3.2, 3.3, 3.4 | 2.5 以降       | 8    | 2.12  |
| 1.0.0           | 3.x           | 1.18 以降      | 8    | 2.12  |
| 1.0.0           | 2.x           | 1.18 以降      | 8    | 2.11  |

> **注意**
>
> - 異なるコネクタバージョン間の動作の変更については、[Spark connectorをアップグレードする](#upgrade-spark-connector)を参照してください。
> - バージョン1.1.1以降のコネクタではMySQL JDBCドライバが提供されず、ドライバをSparkのクラスパスに手動でインポートする必要があります。ドライバは[Maven Central](https://repo1.maven.org/maven2/mysql/mysql-connector-java/)で入手できます。
> - バージョン1.0.0では、Spark connectorはStarRocksからのデータ読み取りのみをサポートしていました。バージョン1.1.0以降、Spark connectorはStarRocksへのデータ読み取りおよび書き込みの両方をサポートしています。
> - バージョン1.0.0とバージョン1.1.0では、パラメータとデータ型のマッピングが異なります。[Spark connectorをアップグレードする](#upgrade-spark-connector)を参照してください。
> - 通常の場合、バージョン1.0.0には新しい機能が追加されることはありません。できるだけ早くSpark connectorをアップグレードすることをお勧めします。

## Spark connectorの取得方法

ビジネスニーズに合ったSpark connector **.jar**パッケージを取得するには、以下の方法のいずれかを使用できます：

- コンパイル済みのパッケージをダウンロードする。
- Mavenを使用してSpark connectorに必要な依存関係を追加する。（この方法は、Spark connector 1.1.0およびそれ以降に対応しています。）
- パッケージを手動でコンパイルする。

### Spark connector 1.1.0 およびそれ以降

Spark connector **.jar**パッケージの命名規則は以下の形式です：

`starrocks-spark-connector-${spark_version}_${scala_version}-${connector_version}.jar`

例えば、Spark connector 1.1.0をSpark 3.2およびScala 2.12で使用したい場合、`starrocks-spark-connector-3.2_2.12-1.1.0.jar`を選択できます。

> **注意**
>
> 通常の場合、最新のSpark connectorバージョンは最新の3つのSparkバージョンで使用できます。

#### コンパイル済みのパッケージをダウンロードする

さまざまなバージョンのSpark connector **.jar**パッケージを[Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks)で入手できます。

#### Mavenの依存関係を追加する

次のように、Spark connectorに必要な依存関係を構成します：

> **注意**
>
> 使用するSparkのバージョン、Scalaのバージョン、Spark connectorのバージョンに応じて、`spark_version`、`scala_version`、`connector_version`を置き換える必要があります。

```xml
<dependency>
  <groupId>com.starrocks</groupId>
  <artifactId>starrocks-spark-connector-${spark_version}_${scala_version}</artifactId>
  <version>${connector_version}</version>
</dependency>
```

例えば、Spark connector 1.1.0をSpark 3.2およびScala 2.12で使用したい場合、依存関係を次のように構成します：

```xml
<dependency>
  <groupId>com.starrocks</groupId>
  <artifactId>starrocks-spark-connector-3.2_2.12</artifactId>
  <version>1.1.0</version>
</dependency>
```

#### パッケージを手動でコンパイルする

1. [Spark connectorのコード](https://github.com/StarRocks/starrocks-connector-for-apache-spark)をダウンロードします。

2. 次のコマンドを使用して、Spark connectorをコンパイルします：

   > **注意**
   >
   > 使用するSparkのバージョンに応じて、`spark_version`を置き換える必要があります。

   ```shell
   sh build.sh <spark_version>
   ```

   例えば、Spark 3.2を使用したい場合、次のようにSpark connectorをコンパイルします：

   ```shell
   sh build.sh 3.2
   ```

3. コンパイル後に、`target/`パスに`starrocks-spark-connector-3.2_2.12-1.1.0-SNAPSHOT.jar`のようなSpark connector **.jar**パッケージが生成されます。

   > **注意**
   >
   > 公式にリリースされていないSpark connectorバージョンを使用している場合、生成されるSpark connector **.jar**パッケージの名前には`SNAPSHOT`がサフィックスとして含まれます。

### Spark connector 1.0.0

#### コンパイル済みのパッケージをダウンロードする

- [Spark 2.x](https://cdn-thirdparty.starrocks.com/spark/starrocks-spark2_2.11-1.0.0.jar)
- [Spark 3.x](https://cdn-thirdparty.starrocks.com/spark/starrocks-spark3_2.12-1.0.0.jar)

#### パッケージを手動でコンパイルする

1. [Spark connectorのコード](https://github.com/StarRocks/starrocks-connector-for-apache-spark/tree/spark-1.0)をダウンロードします。

   > **注意**
   >
   > `spark-1.0`に切り替える必要があります。

2. Spark connectorをコンパイルするために、次のいずれかの操作を実行します：

   - Spark 2.xを使用している場合、デフォルトでSpark 2.3.4に合わせてSpark connectorをコンパイルする次のコマンドを実行します：

     ```Plain
     sh build.sh 2
     ```

   - Spark 3.xを使用している場合、デフォルトでSpark 3.1.2に合わせてSpark connectorをコンパイルする次のコマンドを実行します：

     ```Plain
     sh build.sh 3
     ```

3. コンパイル後に、`output/`パスに`starrocks-spark2_2.11-1.0.0.jar`のようなファイルが生成されます。その後、ファイルをSparkのクラスパスにコピーします：

   - Sparkクラスターが`Local`モードで実行されている場合、ファイルを`jars/`パスに配置します。
   - Sparkクラスターが`Yarn`モードで実行されている場合、ファイルを事前配置パッケージに配置します。

ファイルを指定された場所に配置した後に、Spark connectorを使用してStarRocksからデータを読むことができます。

## パラメータ

このセクションでは、Spark connectorを使用してStarRocksからデータを読む際に構成する必要があるパラメータについて説明します。

### 共通パラメータ

次のパラメータは、Spark SQL、Spark DataFrame、Spark RDDのいずれの読み込み方法にも適用されます。

| パラメータ                            | デフォルト値      | 説明                                                    |
| ------------------------------------ | ----------------- | ------------------------------------------------------------ |
| starrocks.fenodes                    | なし             | StarRocksクラスター内のFEのHTTP URL。フォーマットは`<fe_host>:<fe_http_port>`です。複数のURLを指定することができますが、カンマ(,)で区切る必要があります。 |
| starrocks.table.identifier           | なし             | StarRocksテーブルの名前。フォーマット:`<database_name>.<table_name>` |
| starrocks.request.retries            | 3                | SparkがStarRocksに読み取り要求を送信する際の最大リトライ回数。 |
| starrocks.request.connect.timeout.ms | 30000            | StarRocksへの読み取り要求がタイムアウトするまでの最大時間。 |
| starrocks.request.read.timeout.ms    | 30000             | リクエストがStarRocksに送信されてからタイムアウトするまでの最大時間。 |
| starrocks.request.query.timeout.s    | 3600              | StarRocksからのデータクエリがタイムアウトするまでの最大時間。デフォルトのタイムアウト期間は1時間です。`-1`はタイムアウト期間が指定されていないことを意味します。 |
| starrocks.request.tablet.size        | Integer.MAX_VALUE | 各Spark RDDパーティションにグループ化されるStarRocksのタブレットの数。このパラメータの値が小さいと、より多くのSpark RDDパーティションが生成されます。Spark RDDパーティションの数が多いとSparkでの並列処理が高くなりますが、StarRocksに対する負荷が高くなります。 |
| starrocks.batch.size                 | 4096              | 一度にBEから読み取ることができる最大行数。このパラメータの値を増やすと、SparkとStarRocks間で確立される接続数が減少し、ネットワークの遅延による余分な時間オーバーヘッドが軽減されます。 |
| starrocks.exec.mem.limit             | 2147483648        | クエリごとに許可される最大メモリ量。単位：バイト。デフォルトのメモリ制限は2 GBです。 |
| starrocks.deserialize.arrow.async    | false             | ArrowメモリフォーマットをSparkコネクタのイテレーションに必要なRowBatchesに非同期で変換するかどうかを指定します。 |
| starrocks.deserialize.queue.size     | 64                | ArrowメモリフォーマットをRowBatchesに非同期で変換するタスクを保持する内部キューのサイズ。このパラメータは`starrocks.deserialize.arrow.async`が`true`に設定されている場合に有効です。 |
| starrocks.filter.query               | None              | StarRocks上でデータをフィルタリングする条件。複数のフィルタ条件を指定できますが、`and`で結合する必要があります。データがSparkによって読み取られる前に、StarRocksは指定されたフィルタ条件に基づいてStarRocksテーブルからデータをフィルタリングします。 |
| starrocks.timezone | JVMのデフォルトタイムゾーン | 1.1.1以降サポート。StarRocksの`DATETIME`をSparkの`TimestampType`に変換するために使用されるタイムゾーン。デフォルトは`ZoneId#systemDefault()`が返すJVMのタイムゾーンです。`Asia/Shanghai`などのタイムゾーン名、または`+08:00`などのゾーンオフセットが可能な形式です。 |

### Spark SQLとSpark DataFrameのパラメータ

以下のパラメータはSpark SQLとSpark DataFrameの読み取りメソッドにのみ適用されます。

| パラメータ                           | デフォルト値 | 説明                                                    |
| ----------------------------------- | ------------- | -------------------------------------------------------- |
| starrocks.fe.http.url               | None          | FEのHTTP IPアドレス。このパラメータはSparkコネクタ1.1.0以降でサポートされています。このパラメータは`starrocks.fenodes`と同等です。どちらか一方を構成するだけです。1.1.0以降のSparkコネクタでは、`starrocks.fe.http.url`の使用を推奨します。`starrocks.fenodes`は非推奨になる可能性があります。 |
| starrocks.fe.jdbc.url               | None          | FEのMySQLサーバーに接続するために使用されるアドレス。形式:`jdbc:mysql://<fe_host>:<fe_query_port>`。<br />**NOTICE**<br />Sparkコネクタ1.1.0以降では、このパラメータは必須です。   |
| user                                | None          | StarRocksクラスターアカウントのユーザー名。ユーザーはStarRocksテーブルの[SELECT権限](../sql-reference/sql-statements/account-management/GRANT.md)が必要です。   |
| starrocks.user                      | None          | StarRocksクラスターアカウントのユーザー名。このパラメータはSparkコネクタ1.1.0以降でサポートされています。このパラメータは`user`と同等です。どちらか一方を構成するだけです。1.1.0以降のSparkコネクタでは、`starrocks.user`の使用を推奨します。`user`は非推奨になる可能性があります。 |
| password                            | None          | StarRocksクラスターアカウントのパスワード。    |
| starrocks.password                  | None          | StarRocksクラスターアカウントのパスワード。このパラメータはSparkコネクタ1.1.0以降でサポートされています。このパラメータは`password`と同等です。どちらか一方を構成するだけです。1.1.0以降のSparkコネクタでは、`starrocks.password`の使用を推奨します。`password`は非推奨になる可能性があります。 |
| starrocks.filter.query.in.max.count | 100           | プレディケートプッシュダウン中のIN式でサポートされる値の最大数。IN式で指定された値の数がこの制限を超えると、IN式で指定されたフィルタ条件がSparkで処理されます。   |

### Spark RDDのパラメータ

以下のパラメータはSpark RDDの読み取りメソッドにのみ適用されます。

| パラメータ                       | デフォルト値 | 説明                                                    |
| ------------------------------- | ------------- | -------------------------------------------------------- |
| starrocks.request.auth.user     | None          | StarRocksクラスターアカウントのユーザー名。              |
| starrocks.request.auth.password | None          | StarRocksクラスターアカウントのパスワード。              |
| starrocks.read.field            | None          | データを読み取りたいStarRocksテーブルの列。複数の列を指定することができ、カンマ(,)で区切る必要があります。 |

## StarRocksとSpark間のデータ型マッピング

### Sparkコネクタ1.1.0以降

| StarRocksのデータ型 | Sparkのデータ型           |
|-------------------- |-------------------------- |
| BOOLEAN             | DataTypes.BooleanType     |
| TINYINT             | DataTypes.ByteType        |
| SMALLINT            | DataTypes.ShortType       |
| INT                 | DataTypes.IntegerType     |
| BIGINT              | DataTypes.LongType        |
| LARGEINT            | DataTypes.StringType      |
| FLOAT               | DataTypes.FloatType       |
| DOUBLE              | DataTypes.DoubleType      |
| DECIMAL             | DecimalType               |
| CHAR                | DataTypes.StringType      |
| VARCHAR             | DataTypes.StringType      |
| STRING              | DataTypes.StringType      |
| DATE                | DataTypes.DateType        |
| DATETIME            | DataTypes.TimestampType   |
| ARRAY               | サポートされていないデータ型      |
| HLL                 | サポートされていないデータ型      |
| BITMAP              | サポートされていないデータ型      |

### Sparkコネクタ1.0.0

| StarRocksのデータ型  | Sparkのデータ型        |
| -------------------- | ---------------------- |
| BOOLEAN              | DataTypes.BooleanType  |
| TINYINT              | DataTypes.ByteType     |
| SMALLINT             | DataTypes.ShortType    |
| INT                  | DataTypes.IntegerType  |
| BIGINT               | DataTypes.LongType     |
| LARGEINT             | DataTypes.StringType   |
| FLOAT                | DataTypes.FloatType    |
| DOUBLE               | DataTypes.DoubleType   |
| DECIMAL              | DecimalType            |
| CHAR                 | DataTypes.StringType   |
| VARCHAR              | DataTypes.StringType   |
| DATE                 | DataTypes.StringType   |
| DATETIME             | DataTypes.StringType   |
| ARRAY                | サポートされていないデータ型   |
| HLL                  | サポートされていないデータ型   |
| BITMAP               | サポートされていないデータ型   |

StarRocksによって使用される基礎ストレージエンジンの処理ロジックは、DATEやDATETIMEのデータ型が直接使用されると期待される時間範囲をカバーできません。そのため、SparkコネクタはStarRocksのDATEやDATETIMEのデータ型をSparkのSTRINGのデータ型にマッピングし、StarRocksから読み取られた日付と時刻のデータに一致する読みやすい文字列テキストを生成します。

## Sparkコネクタのアップグレード

### バージョン1.0.0からバージョン1.1.0へのアップグレード

- 1.1.1以降、SparkコネクタはGPLライセンスを使用している`mysql-connector-java`の制約により、MySQLの公式JDBCドライバである`mysql-connector-java`を提供しません。
  ただし、引き続きテーブルメタデータを取得するためにはドライバが必要です。そのため、Sparkのクラスパスにドライバを手動で追加する必要があります。ドライバは[MySQLサイト](https://dev.mysql.com/downloads/connector/j/)や[Maven Central](https://repo1.maven.org/maven2/mysql/mysql-connector-java/)から見つけることができます。
  
- バージョン1.1.0では、より詳細なテーブル情報を取得するためにSparkコネクタがJDBCを使用するようになりました。そのため、`starrocks.fe.jdbc.url`を構成する必要があります。

- バージョン1.1.0では、一部のパラメータがリネームされました。新旧のパラメータが現在も保持されています。対応するパラメータのペアごとに、いずれか一方を構成するだけで問題ありませんが、古いパラメータは非推奨になる可能性があるため、新しいパラメータの使用を推奨します。
  - `starrocks.fenodes`は`starrocks.fe.http.url`にリネームされました。
  - `user`は`starrocks.user`にリネームされました。
  - `password`は`starrocks.password`にリネームされました。

- バージョン1.1.0では、一部のデータ型のマッピングがSpark 3.xに基づいて調整されました。
  - StarRocksの`DATE`はSparkの`DataTypes.DateType`(元々`DataTypes.StringType`)にマッピングされています。
  - StarRocksの`DATETIME`はSparkの`DataTypes.TimestampType`(元々`DataTypes.StringType`)にマッピングされています。

## 例

以下の例では、StarRocksクラスタに`test`という名前のデータベースが作成されており、ユーザー`root`の権限があると仮定しています。例ではSparkコネクタ1.1.0のパラメータ設定を前提としています。

### データの例

以下の手順に従って、サンプルテーブルの準備を行ってください。

1. `test`データベースに移動し、`score_board`というテーブルを作成してください。

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

2. `score_board`テーブルにデータを挿入してください。

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

3. `score_board`テーブルからデータをクエリしてください。

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
   21 rows in set (0.01 sec)
   ```

### Spark SQLを使用してデータを読み取る

1. Spark SQLを起動するため、Sparkディレクトリで以下のコマンドを実行してください。

   ```Plain
   sh spark-sql
   ```

2. 以下のコマンドを実行し、`test`データベースに所属する`score_board`テーブルに`spark_starrocks`という一時ビューを作成してください。

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

3. 以下のコマンドを実行し、一時ビューからデータを読み取ってください。

   ```SQL
   spark-sql> SELECT * FROM spark_starrocks;
   ```

   上記のコマンドにより、Sparkは次のデータを返します。

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
   Time taken: 1.883 seconds, Fetched 21 row(s)
   22/08/09 15:29:36 INFO thriftserver.SparkSQLCLIDriver: Time taken: 1.883 seconds, Fetched 21 row(s)
   ```

### Spark DataFrameを使用してデータを読み取る

1. Sparkシェルを起動するため、Sparkディレクトリで以下のコマンドを実行してください。

   ```Plain
   sh spark-shell
   ```

2. 以下のコマンドを実行し、`test`データベースに所属する`score_board`テーブルに`starrocksSparkDF`というDataFrameを作成してください。

   ```Scala
   scala> val starrocksSparkDF = spark.read.format("starrocks")
              .option("starrocks.table.identifier", s"test.score_board")
              .option("starrocks.fe.http.url", s"<fe_host>:<fe_http_port>")
              .option("starrocks.fe.jdbc.url", s"jdbc:mysql://<fe_host>:<fe_query_port>")
              .option("starrocks.user", s"root")
              .option("starrocks.password", s"")
              .load()
   ```

3. DataFrameからデータを読み取ってください。例えば、最初の10行を読み取る場合は、以下のコマンドを実行してください。

   ```Scala
   scala> starrocksSparkDF.show(10)
   ```

   Sparkは次のデータを返します。

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
   only showing top 10 rows
   ```

   > **注意**
   >
   > データを読み取る行数を指定しない場合、Sparkはデフォルトで最初の20行を返します。

### Spark RDDを使用してデータを読み取る

1. Sparkシェルを起動するため、Sparkディレクトリで以下のコマンドを実行してください。

   ```Plain
   sh spark-shell
   ```

2. 以下のコマンドを実行し、`test`データベースに所属する`score_board`テーブルに`starrocksSparkRDD`というRDDを作成してください。

   ```Scala
   scala> import com.starrocks.connector.spark._
   scala> val starrocksSparkRDD = sc.starrocksRDD
              (
              tableIdentifier = Some("test.score_board"),
              cfg = Some(Map(
                  "starrocks.fenodes" -> "<fe_host>:<fe_http_port>",
                  "starrocks.request.auth.user" -> "root",
                  "starrocks.request.auth.password" -> ""
              ))
              )
   ```

3. RDDからデータを読み取ってください。例えば、最初の10要素を読み取る場合は、以下のコマンドを実行してください。

   ```Scala
   scala> starrocksSparkRDD.take(10)
   ```

   Sparkは次のデータを返します。

   ```Scala
   res0: Array[AnyRef] = Array([1, Bob, 21], [2, Stan, 21], [3, Sam, 22], [4, Tony, 22], [5, Alice, 22], [6, Lucy, 23], [7, Polly, 23], [8, Tom, 23], [9, Rose, 24], [10, Jerry, 24])
   ```

   全体のRDDを読み取るには、以下のコマンドを実行してください。

   ```Scala
   scala> starrocksSparkRDD.collect()
   ```

   Sparkは次のデータを返します。
```scala
res1: Array[AnyRef] = Array([1, Bob, 21], [2, Stan, 21], [3, Sam, 22], [4, Tony, 22], [5, Alice, 22], [6, Lucy, 23], [7, Polly, 23], [8, Tom, 23], [9, Rose, 24], [10, Jerry, 24], [11, Jason, 24], [12, Lily, 25], [13, Stephen, 25], [14, David, 25], [15, Eddie, 26], [16, Kate, 27], [17, Cathy, 27], [18, Judy, 27], [19, Julia, 28], [20, Robert, 28], [21, Jack, 29])
   ```

## Best practices

StarRocksのデータをSparkコネクタを使用して読み込む際、`starrocks.filter.query`パラメータを使用して、Sparkがパーティション、バケツ、およびプレフィックスインデックスの剪定を行い、データ取得のコストを削減するフィルタ条件を指定することができます。このセクションでは、Spark DataFrameを使用して、これをどのように達成するかを示します。

### 環境設定

| コンポーネント  | バージョン                                                  |
| -------------- | ---------------------------------------------------------- |
| Spark          | Spark 2.4.4 および Scala 2.11.12 (OpenJDK 64-Bit Server VM, Java 1.8.0_302) |
| StarRocks      | 2.2.0                                                      |
| Sparkコネクタ   | starrocks-spark2_2.11-1.0.0.jar                            |

### データの例

以下の手順に従って、サンプルテーブルを準備します:

1. `test`データベースに移動して、`mytable`というテーブルを作成します。

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

2. `mytable`にデータを挿入します。

   ```SQL
   MySQL [test]> INSERT INTO mytable
   VALUES
        (1, 11, '2022-01-02 08:00:00', 111),
        (2, 22, '2022-02-02 08:00:00', 222),
        (3, 33, '2022-03-02 08:00:00', 333);
   ```

3. `mytable`テーブルをクエリします。

   ```SQL
   MySQL [test]> select * from mytable;
   +------+------+---------------------+------+
   | k    | b    | dt                  | v    |
   +------+------+---------------------+------+
   |    1 |   11 | 2022-01-02 08:00:00 |  111 |
   |    2 |   22 | 2022-02-02 08:00:00 |  222 |
   |    3 |   33 | 2022-03-02 08:00:00 |  333 |
   +------+------+---------------------+------+
   3 rows in set (0.01 sec)
   ```

### フルテーブルスキャン

1. Sparkディレクトリで以下のコマンドを実行して、`test`データベースに属する`mytable`テーブルの`df`という名前のDataFrameを作成します:

   ```Scala
   scala>  val df = spark.read.format("starrocks")
           .option("starrocks.table.identifier", s"test.mytable")
           .option("starrocks.fenodes", s"<fe_host>:<fe_http_port>")
           .option("user", s"root")
           .option("password", s"")
           .load()
   ```

2. StarRocksクラスタのFEログファイル**fe.log**を表示し、データを読み込むために実行されたSQLステートメントを見つけます。例:

   ```SQL
   2022-08-09 18:57:38,091 INFO (nioEventLoopGroup-3-10|196) [TableQueryPlanAction.executeWithoutPassword():126] external service [ user ['root'@'%']] からデータベース [test] テーブル [mytable] の [select `k`,`b`,`dt`,`v` from `test`.`mytable`] ステートメントを受け取りました
   ```

3. `test`データベースで、`test`.`mytable`から`k`,`b`,`dt`,`v`を選択する実行計画を取得するためにEXPLAINを使用します:

   ```SQL
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
   26 rows in set (0.00 sec)
   ```

この例では、剪定が行われていません。そのため、Sparkはデータを保持する3つのパーティション（`partitions=3/3`で示されている）およびこれらの3つのパーティション内の9つのタブレット（`tabletRatio=9/9`で示されている）すべてをスキャンします。

### パーティションの剪定

1. 以下のコマンドを実行して、パーティション剪定のためのフィルタ条件 `dt='2022-01-02 08:00:00`を指定した`mytable`テーブルの`df`という名前のDataFrameを作成します:

   ```Scala
   scala> val df = spark.read.format("starrocks")
          .option("starrocks.table.identifier", s"test.mytable")
          .option("starrocks.fenodes", s"<fe_host>:<fe_http_port>")
          .option("user", s"root")
          .option("password", s"")
          .option("starrocks.filter.query", "dt='2022-01-02 08:00:00'")
          .load()
   ```

2. StarRocksクラスタのFEログファイル**fe.log**を表示し、データを読み込むために実行されたSQLステートメントを見つけます。例:

   ```SQL
   2022-08-09 19:02:31,253 INFO (nioEventLoopGroup-3-14|204) [TableQueryPlanAction.executeWithoutPassword():126] external service [ user ['root'@'%']] からデータベース [test] テーブル [mytable] の [select `k`,`b`,`dt`,`v` from `test`.`mytable` where dt='2022-01-02 08:00:00'] ステートメントを受け取りました
   ```

3. `test`データベースで、`test`.`mytable`から`dt='2022-01-02 08:00:00'`のデータを選択する実行計画を取得するためにEXPLAINを使用します:

   ```SQL
   MySQL [test]> EXPLAIN select `k`,`b`,`dt`,`v` from `test`.`mytable` where dt='2022-01-02 08:00:00';
   +------------------------------------------------+
   | Explain String                                 |
   +------------------------------------------------+
   | PLAN FRAGMENT 0                                |
   |  OUTPUT EXPRS:1: k | 2: b | 3: dt | 4: v       |
   |   PARTITION: UNPARTITIONED                     |
   |                                                |
   |   RESULT SINK                                  |

```
+   |                                                |
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
   27 行が設定されました（0.01 秒）

この例では、分割剪定のみが実行され、バケツ剪定は実行されていません。したがって、Spark は 3 つのパーティションの 1 つをスキャンし（`partitions=1/3` で示されるように）、そのパーティション内のすべてのタブレットをスキャンします（`tabletRatio=3/3` で示されるように）。

### バケツ剪定

1. 次のコマンドを実行し、`starrocks.filter.query` パラメータを使用してバケツ剪定のためのフィルタ条件 `k=1` を指定し、`test` データベースの `mytable` テーブルに `df` という名前の DataFrame を作成します。

   ```Scala
   scala> val df = spark.read.format("starrocks")
          .option("starrocks.table.identifier", s"test.mytable")
          .option("starrocks.fenodes", s"<fe_host>:<fe_http_port>")
          .option("user", s"root")
          .option("password", s"")
          .option("starrocks.filter.query", "k=1")
          .load()
   ```

2. StarRocks クラスタの FE ログファイル**fe.log**を表示し、データを読み取るために実行された SQL ステートメントを見つけます。例：

   ```SQL
   2022-08-09 19:04:44,479 INFO (nioEventLoopGroup-3-16|208) [TableQueryPlanAction.executeWithoutPassword():126] External service からデータベース [test] テーブル [mytable] のための SQL ステートメント [select `k`,`b`,`dt`,`v` from `test`.`mytable` where k=1] を受け取りました。ユーザー ['root'@'%']
   ```

3. `test` データベースで、`k=1` のフィルタ条件を満たす Hash 値を取得するために3 つのパーティション ( `partitions=3/3` で示されるように) とその3つのパーティション内の `k = 1` フィルタ条件を満たす Hash 値を取得するために全ての3タブレットをスキャンするように Spark に指示した `tabletRatio=3/9` であるSELECT `k`,`b`,`dt`,`v` の実行計画を取得するために EXPLAIN を使用します。

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
   27 行が設定されました（0.01 秒）

この例では、分割剪定のみが実行され、バケツ剪定は実行されていません。したがって、Spark はデータを保持する 3 つのパーティションをすべてスキャンし（`partitions=3/3` で示されるように）、バケツ内の `k = 1` フィルタ条件を満たす Hash 値を取得するために3つのタブレットをすべてスキャンします（`tabletRatio=3/9` で示されるように）。

### 分割剪定およびバケツ剪定

1. 次のコマンドを実行し、`starrocks.filter.query` パラメータを使用してバケツ剪定と分割剪定のためのフィルタ条件 `k=7` および `dt='2022-01-02 08:00:00'` を指定し、`test` データベースの `mytable` テーブルに DataFrame `df` を作成します。

   ```Scala
   scala> val df = spark.read.format("starrocks")
          .option("starrocks.table.identifier", s"test.mytable")
          .option("starrocks.fenodes", s"<fe_host>:<fe_http_port>")
          .option("user", s"")
          .option("password", s"")
          .option("starrocks.filter.query", "k=7 and dt='2022-01-02 08:00:00'")
          .load()
   ```

2. StarRocks クラスタの FE ログファイル**fe.log**を表示し、データを読み取るために実行された SQL ステートメントを見つけます。例：

   ```SQL
   2022-08-09 19:06:34,939 INFO (nioEventLoopGroup-3-18|212)  [TableQueryPlanAction.executeWithoutPassword():126] 外部サービスからデータベース [test] テーブル [mytable] のための SQL ステートメント [select `k`,`b`,`dt`,`v` from `test`.`mytable` where k=7 and dt='2022-01-02 08:00:00'] を受け取りました。ユーザー ['root'@'%']
   ```

3. `test` データベースで、`k = 7` および `dt='2022-01-02 08:00:00'` フィルタ条件を満たす Hash 値を取得するために`partitions=1/3` に従う1つのパーティションおよびそのパーティション内の`tabletRatio=1/3` のSELECT `k`,`b`,`dt`,`v` の実行計画を取得するために EXPLAIN を使用します。

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
   17 行が設定されました（0.00 秒）

この例では、分割剪定とバケツ剪定の両方が実行されています。したがって、Spark は 1 つのパーティションのみをスキャンし（`partitions=1/3` で示されるように）、そのパーティション内の 1 つのタブレットのみをスキャンします（`tabletRatio=1/3` で示されるように）。

### プレフィックスインデックスのフィルタリング

1. `test` データベースの `mytable` テーブルのパーティションに追加のデータレコードを挿入します。

   ```Scala
   MySQL [test]> INSERT INTO mytable
   VALUES
       (1, 11, "2022-01-02 08:00:00", 111), 
       (3, 33, "2022-01-02 08:00:00", 333), 
       (3, 33, "2022-01-02 08:00:00", 333), 
       (3, 33, "2022-01-02 08:00:00", 333);
   ```

2. `mytable` テーブルをクエリします。

   ```Scala
   MySQL [test]> SELECT * FROM mytable;
   +------+------+---------------------+------+
   | k    | b    | dt                  | v    |
   +------+------+---------------------+------+
   |    1 |   11 | 2022-01-02 08:00:00 |  111 |
```markdown
   |    1 |   11 | 2022-01-02 08:00:00 |  111 |
   |    3 |   33 | 2022-01-02 08:00:00 |  333 |
   |    3 |   33 | 2022-01-02 08:00:00 |  333 |
   |    3 |   33 | 2022-01-02 08:00:00 |  333 |
   |    2 |   22 | 2022-02-02 08:00:00 |  222 |
   |    3 |   33 | 2022-03-02 08:00:00 |  333 |
   +------+------+---------------------+------+
   7 rows in set (0.01 sec)
   ```

3. The following command is executed in the Spark directory to create a DataFrame named `df` on the `mytable` table which belongs to the `test` database, using the `starrocks.filter.query` parameter to specify a filter condition `k=1` for prefix index filtering:

   ```Scala
   scala> val df = spark.read.format("starrocks")
          .option("starrocks.table.identifier", s"test.mytable")
          .option("starrocks.fenodes", s"<fe_host>:<fe_http_port>")
          .option("user", s"root")
          .option("password", s"")
          .option("starrocks.filter.query", "k=1")
          .load()
   ```

4. In the `test` database, `is_report_success` is set to `true` to enable profile reporting:

   ```SQL
   MySQL [test]> SET is_report_success = true;
   Query OK, 0 rows affected (0.00 sec)
   ```

5. The profile of the `SELECT * FROM mytable where k=1` statement can be viewed by opening the `http://<fe_host>:<http_http_port>/query` page in a browser. Example:

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
          - __MIN_OF_UncompressedBytesRead: 0.00 
   ```

In this example, the filter condition `k = 1` can hit the prefix index. Therefore, Spark can filter out three rows (as suggested by `ShortKeyFilterRows: 3`).
```