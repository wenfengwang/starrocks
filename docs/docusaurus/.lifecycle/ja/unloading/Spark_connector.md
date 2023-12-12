---
displayed_sidebar: "Japanese"
---

# Sparkコネクタを使用してStarRocksからデータを読む

StarRocksは、Apache Spark™（以下、Sparkコネクタ）用にStarRocksコネクタという自社開発のコネクタを提供し、Sparkを使用してStarRocksテーブルからデータを読むために役立ちます。Sparkを使用すると、StarRocksから読んだデータを使って複雑な処理や機械学習を行うことができます。

Sparkコネクタは、Spark SQL、Spark DataFrame、およびSpark RDDの3つの読み取り方法をサポートしています。

Spark SQLを使用してStarRocksテーブルに一時ビューを作成し、その一時ビューを使用してStarRocksテーブルから直接データを読むことができます。

また、StarRocksテーブルをSpark DataFrameまたはSpark RDDにマップし、その後、Spark DataFrameまたはSpark RDDからデータを読むことができます。Spark DataFrameの使用を推奨します。

> **注意**
>
> StarRocksテーブルのSELECT権限を持つユーザーのみ、このテーブルからデータを読むことができます。[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)で権限をユーザーに付与する手順に従ってください。

## 使用上の注意

- データを読む前にStarRocksでデータをフィルタリングすることで、転送するデータ量を減らすことができます。
- データの読み取りコストが大きい場合は、適切なテーブル設計およびフィルタ条件を使用して、一度に過剰なデータを読み取ることを防ぐことができます。そのようにして、ディスクおよびネットワーク接続へのI/O圧力を軽減し、定期的なクエリを適切に実行できるようにします。

## バージョン要件

| Sparkコネクタ | Spark         | StarRocks       | Java | Scala |
|-------------- | ------------- | --------------- | ---- | ----- |
| 1.1.1         | 3.2, 3.3, 3.4 | 2.5 以降       | 8    | 2.12  |
| 1.1.0         | 3.2, 3.3, 3.4 | 2.5 以降       | 8    | 2.12  |
| 1.0.0         | 3.x           | 1.18 以降      | 8    | 2.12  |
| 1.0.0         | 2.x           | 1.18 以降      | 8    | 2.11  |

> **注意**
>
> - 異なるコネクタのバージョン間の動作変更については、[Sparkコネクタのアップグレード](#spark-connectorのアップグレード)を参照してください。
> - バージョン1.1.1以降、コネクタはMySQL JDBCドライバを提供していません。そのため、ドライバをSparkのクラスパスに手動でインポートする必要があります。ドライバは[Maven Central](https://repo1.maven.org/maven2/mysql/mysql-connector-java/)で入手できます。
> - バージョン1.0.0では、SparkコネクタはStarRocksからのデータ読み取りのみをサポートしています。バージョン1.1.0以降、SparkコネクタはStarRocksへのデータ読み取りおよび書き込みの両方をサポートしています。
> - バージョン1.0.0とバージョン1.1.0とでは、パラメータとデータ型のマッピングに違いがあります。[Sparkコネクタのアップグレード](#spark-connectorのアップグレード)を参照してください。
> - 一般的な場合、バージョン1.0.0には新機能が追加されません。できるだけ早くSparkコネクタをアップグレードすることをお勧めします。

## Sparkコネクタの取得

ビジネスニーズに適したSparkコネクタの**.jar**パッケージを入手するために、次のいずれかの方法を使用できます。

- コンパイルされたパッケージをダウンロードします。
- Mavenを使用してSparkコネクタに必要な依存関係を追加します（この方法は、Sparkコネクタ1.1.0以降にのみ対応しています）。
- パッケージを手動でコンパイルします。

### Sparkコネクタ1.1.0以降

Sparkコネクタの**.jar**パッケージは、次の形式で命名されています。

`starrocks-spark-connector-${spark_version}_${scala_version}-${connector_version}.jar`

たとえば、Spark 3.2およびScala 2.12と一緒にSparkコネクタ1.1.0を使用したい場合、`starrocks-spark-connector-3.2_2.12-1.1.0.jar`を選択できます。

> **注意**
>
> 通常の場合、最新のSparkコネクタのバージョンは、最近の3つのSparkバージョンで使用できます。

#### コンパイルされたパッケージをダウンロードする

さまざまなバージョンのSparkコネクタ**.jar**パッケージを[Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks)で入手できます。

#### Maven依存関係を追加する

Sparkコネクタに必要な依存関係を次のように構成します。

> **注意**
>
> 使用するSparkバージョン、Scalaバージョン、およびSparkコネクタバージョンに応じて、`spark_version`、`scala_version`、および`connector_version`を置き換える必要があります。

```xml
<dependency>
  <groupId>com.starrocks</groupId>
  <artifactId>starrocks-spark-connector-${spark_version}_${scala_version}</artifactId>
  <version>${connector_version}</version>
</dependency>
```

たとえば、Spark 3.2およびScala 2.12でSparkコネクタ1.1.0を使用したい場合、次のように依存関係を構成します。

```xml
<dependency>
  <groupId>com.starrocks</groupId>
  <artifactId>starrocks-spark-connector-3.2_2.12</artifactId>
  <version>1.1.0</version>
</dependency>
```

#### パッケージを手動でコンパイルする

1. [Sparkコネクタのコード](https://github.com/StarRocks/starrocks-connector-for-apache-spark)をダウンロードします。

2. 次のコマンドを使用してSparkコネクタをコンパイルします。

   > **注意**
   >
   > 使用するSparkバージョンを`spark_version`で置き換える必要があります。

   ```shell
   sh build.sh <spark_version>
   ```

   たとえば、Spark 3.2でSparkコネクタを使用したい場合、次のようにSparkコネクタをコンパイルします。

   ```shell
   sh build.sh 3.2
   ```

3. `target/`パスに移動し、Sparkコネクタ**.jar**パッケージ（`starrocks-spark-connector-3.2_2.12-1.1.0-SNAPSHOT.jar`など）が生成されます。

   > **注意**
   >
   > 公式にリリースされていないSparkコネクタのバージョンを使用している場合、生成されたSparkコネクタ**.jar**パッケージの名前には`SNAPSHOT`が含まれます。

### Sparkコネクタ1.0.0

#### コンパイルされたパッケージをダウンロードする

- [Spark 2.x](https://cdn-thirdparty.starrocks.com/spark/starrocks-spark2_2.11-1.0.0.jar)
- [Spark 3.x](https://cdn-thirdparty.starrocks.com/spark/starrocks-spark3_2.12-1.0.0.jar)

#### パッケージを手動でコンパイルする

1. [Sparkコネクタのコード](https://github.com/StarRocks/starrocks-connector-for-apache-spark/tree/spark-1.0)をダウンロードします。

   > **注意**
   >
   > `spark-1.0`に切り替える必要があります。

2. 次のいずれかのアクションを実行して、Sparkコネクタをコンパイルします。

   - Spark 2.xを使用している場合、デフォルトでSpark 2.3.4に適したSparkコネクタをコンパイルする次のコマンドを実行します。

     ```Plain
     sh build.sh 2
     ```

   - Spark 3.xを使用している場合、デフォルトでSpark 3.1.2に適したSparkコネクタをコンパイルする次のコマンドを実行します。

     ```Plain
     sh build.sh 3
     ```

3. `output/`パスに移動し、`starrocks-spark2_2.11-1.0.0.jar`ファイルなどが生成されます。その後、ファイルをSparkのクラスパスにコピーします。

   - Sparkクラスターが`Local`モードで実行されている場合、ファイルを`jars/`パスに配置します。
   - Sparkクラスターが`Yarn`モードで実行されている場合、ファイルを事前デプロイメントパッケージに配置します。

ファイルを指定された場所に配置した後、Sparkコネクタを使用してStarRocksからデータを読むことができます。

## パラメータ

このセクションでは、StarRocksからデータを読む際に構成する必要があるパラメータについて説明します。

### 一般的なパラメータ

次のパラメータは、Spark SQL、Spark DataFrame、およびSpark RDDのすべての読み取り方法に適用されます。

| パラメータ                               | デフォルト値       | 説明                                                   |
| ------------------------------------ | ----------------- | ------------------------------------------------------------ |
| starrocks.fenodes                    | なし              | StarRocksクラスター内のFEのHTTP URL。形式は `<fe_host>:<fe_http_port>` です。複数のURLを指定することができますが、URL間はカンマ(,)で区切る必要があります。 |
| starrocks.table.identifier           | なし              | StarRocksテーブルの名前。形式は `<database_name>.<table_name>` です。 |
| starrocks.request.retries            | 3                 | SparkがStarRocksに読み取りリクエストを送信しようとする回数の最大値。 |
| starrocks.request.connect.timeout.ms | 30000             | StarRocksへの読み取りリクエストのタイムアウト時間の最大値。 |
| starrocks.request.read.timeout.ms    | 30000             | リクエストをStarRocksに送信してからの読み取りがタイムアウトするまでの最大時間。 |
| starrocks.request.query.timeout.s    | 3600              | StarRocksからのデータクエリがタイムアウトするまでの最大時間。デフォルトのタイムアウト期間は1時間です。`-1` はタイムアウト期間が指定されていないことを意味します。 |
| starrocks.request.tablet.size        | Integer.MAX_VALUE | 各Spark RDDパーティションにグループ化されるStarRocksタブレットの数。このパラメータの値が小さいほど、より多くのSpark RDDパーティションが生成されます。より多くのSpark RDDパーティションはSpark上の並列処理を高めますが、StarRocksに大きな負荷をかけます。 |
| starrocks.batch.size                 | 4096              | 一度にBEから読み取れる最大行数。このパラメータの値を増やすと、SparkとStarRocksの間で確立される接続数が減少し、ネットワークの遅延による余分な時間オーバーヘッドが緩和されます。 |
| starrocks.exec.mem.limit             | 2147483648        | クエリごとに許可される最大メモリ量。単位：バイト。デフォルトのメモリ制限は2GBです。 |
| starrocks.deserialize.arrow.async    | false             | Arrowメモリ形式をSparkコネクタの反復処理に必要なRowBatchesに非同期に変換するサポートを行うかどうかを指定します。 |
| starrocks.deserialize.queue.size     | 64                | Arrowメモリ形式を非同期にRowBatchesに変換するタスクを保持する内部キューのサイズ。このパラメータは、`starrocks.deserialize.arrow.async` が `true` に設定されている場合に有効です。 |
| starrocks.filter.query               | None              | StarRocks上でデータをフィルタリングしたい条件。`and` で結合された複数のフィルタ条件を指定できます。Sparkによってデータが読み込まれる前に、StarRocksが指定されたフィルタ条件に基づいてStarRocksテーブルからデータをフィルタリングします。 |
| starrocks.timezone | JVMのデフォルトタイムゾーン | 1.1.1以降でサポート。StarRocksの `DATETIME` をSparkの `TimestampType` に変換するために使用されるタイムゾーン。デフォルトは `ZoneId#systemDefault()` によって返されるJVMのタイムゾーンです。タイムゾーン名（`Asia/Shanghai` など）やオフセット（`+08:00` など）などのフォーマットが可能です。 |

### Spark SQLおよびSpark DataFrameのパラメータ

以下のパラメータは、Spark SQLおよびSpark DataFrameの読み取りメソッドにのみ適用されます。

| パラメータ                           | デフォルト値 | 説明                                                |
| ----------------------------------- | ------------- | ---------------------------------------------------- |
| starrocks.fe.http.url               | None          | FEのHTTP IPアドレス。このパラメータは、Sparkコネクタの1.1.0以降でサポートされています。このパラメータは `starrocks.fenodes` と等価です。どちらか一方だけを構成すれば良いです。Sparkコネクタ1.1.0以降では、`starrocks.fe.http.url` を使用することをお勧めします。`starrocks.fenodes` は非推奨になる可能性があります。 |
| starrocks.fe.jdbc.url               | None          | FEのMySQLサーバに接続するために使用されるアドレス。フォーマット： `jdbc:mysql://<fe_host>:<fe_query_port>`。<br />**注意**<br />Sparkコネクタ1.1.0以降では、このパラメータは必須です。   |
| user                                | None          | StarRocksクラスタアカウントのユーザー名。ユーザーはStarRocksテーブルでの [SELECT権限](../sql-reference/sql-statements/account-management/GRANT.md) が必要です。   |
| starrocks.user                      | None          | StarRocksクラスタアカウントのユーザー名。このパラメータは、Sparkコネクタ1.1.0以降でサポートされています。このパラメータは `user` と等価です。どちらか一方だけを構成すれば良いです。Sparkコネクタ1.1.0以降では、`starrocks.user` を使用することをお勧めします。`user` は非推奨になる可能性があります。   |
| password                            | None          | StarRocksクラスタアカウントのパスワード。    |
| starrocks.password                  | None          | StarRocksクラスタアカウントのパスワード。このパラメータは、Sparkコネクタ1.1.0以降でサポートされています。このパラメータは `password` と等価です。どちらか一方だけを構成すれば良いです。Sparkコネクタ1.1.0以降では、`starrocks.password` を使用することをお勧めします。`password` は非推奨になる可能性があります。   |
| starrocks.filter.query.in.max.count | 100           | プレディケートプッシュダウン中にIN式でサポートされる最大値の数。IN式で指定された値の数がこの制限を超えると、IN式で指定されたフィルタ条件はSpark上で処理されます。   |

### Spark RDDのパラメータ

以下のパラメータは、Spark RDDの読み取りメソッドにのみ適用されます。

| パラメータ                       | デフォルト値 | 説明                                                |
| ------------------------------- | ------------- | ---------------------------------------------------- |
| starrocks.request.auth.user     | None          | StarRocksクラスタアカウントのユーザー名。              |
| starrocks.request.auth.password | None          | StarRocksクラスタアカウントのパスワード。              |
| starrocks.read.field            | None          | データを読み取りたいStarRocksテーブルのカラム。複数のカラムを指定することができます。カンマ(,)で区切る必要があります。 |

## StarRocksとSparkのデータ型のマッピング

### Sparkコネクタ1.1.0以降

| StarRocksのデータ型 | Sparkのデータ型        |
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

DATEおよびDATETIMEデータ型が直接使用される場合、StarRocksの使用する基礎ストレージエンジンの処理ロジックは、期待される時間範囲をカバーすることができません。そのため、SparkコネクタはStarRocksのDATEおよびDATETIMEデータ型をSparkのSTRINGデータ型にマッピングし、StarRocksから読み取られる日付および時間データに合致する可読性のある文字列テキストを生成します。

## Sparkコネクタのアップグレード

### バージョン1.0.0からバージョン1.1.0へのアップグレード

- 1.1.1から、GPLライセンスを使用している`mysql-connector-java`の制約により、SparkコネクタはMySQLの公式JDBCドライバである `mysql-connector-java` を提供しません。ただし、Sparkコネクタはテーブルメタデータを取得するために `mysql-connector-java` を必要とするため、ドライバをSparkのクラスパスに手動で追加する必要があります。ドライバは[MySQLサイト](https://dev.mysql.com/downloads/connector/j/)や[Mavenセントラル](https://repo1.maven.org/maven2/mysql/mysql-connector-java/)で見つけることができます。

- バージョン1.1.0では、Sparkコネクタはより詳細なテーブル情報を取得するためにJDBCを使用します。そのため、`starrocks.fe.jdbc.url`を設定する必要があります。

- バージョン1.1.0では、一部のパラメータが名前が変更されました。古いパラメータと新しいパラメータの両方が現在も保持されています。それぞれの等価なパラメータのペアについては、どちらか一方だけを構成すれば良いです。ただし、古いパラメータが非推奨になる可能性があるため、新しいパラメータを使用することをお勧めします。
   - `starrocks.fenodes` は `starrocks.fe.http.url` に名前が変更されました。
   - `user` は `starrocks.user` に名前が変更されました。
   - `password` は `starrocks.password` に名前が変更されました。

- バージョン1.1.0では、一部のデータ型のマッピングがSpark 3.xを基に調整されました。
   - StarRocksでの `DATE` は、Sparkの `DataTypes.DateType` （元々 `DataTypes.StringType` ）にマッピングされます。
   - StarRocksでの `DATETIME` は、Sparkの `DataTypes.TimestampType` （元々 `DataTypes.StringType` ）にマッピングされます。

## 例

以下の例では、StarRocksクラスタで `test` という名前のデータベースを作成し、ユーザー `root` の権限があると仮定しています。例のパラメータ設定は、Spark Connector 1.1.0に基づいています。

### データ例
次の手順に従ってサンプルテーブルを準備してください。

1. `test` データベースに移動し、`score_board` という名前のテーブルを作成します。

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

2. `score_board` テーブルにデータを挿入します。

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

3. `score_board` テーブルをクエリします。

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

### Spark SQL を使用してデータを読み取る

1. Spark SQL を起動するには、Spark ディレクトリで次のコマンドを実行します。

   ```Plain
   sh spark-sql
   ```

2. 次のコマンドを実行して、`test` データベースに属する `score_board` テーブルに `spark_starrocks` という名前の一時ビューを作成します。

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

3. 次のコマンドを実行して、一時ビューからデータを読み取ります。

   ```SQL
   spark-sql> SELECT * FROM spark_starrocks;
   ```

   Spark は次のデータを返します。

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

### Spark DataFrame を使用してデータを読み取る

1. Spark シェルを起動するには、Spark ディレクトリで次のコマンドを実行します。

   ```Plain
   sh spark-shell
   ```

2. 次のコマンドを実行して、`test` データベースに属する `score_board` テーブルに `starrocksSparkDF` という名前の DataFrame を作成します。

   ```Scala
   scala> val starrocksSparkDF = spark.read.format("starrocks")
              .option("starrocks.table.identifier", s"test.score_board")
              .option("starrocks.fe.http.url", s"<fe_host>:<fe_http_port>")
              .option("starrocks.fe.jdbc.url", s"jdbc:mysql://<fe_host>:<fe_query_port>")
              .option("starrocks.user", s"root")
              .option("starrocks.password", s"")
              .load()
   ```

3. DataFrame からデータを読み取ります。たとえば、最初の 10 行を読み取りたい場合は、次のように実行します。

   ```Scala
   scala> starrocksSparkDF.show(10)
   ```

   Spark は次のデータを返します。

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
   > 指定しない場合、デフォルトで Spark は最初の 20 行を返します。

### Spark RDD を使用してデータを読み取る

1. Spark シェルを起動するには、Spark ディレクトリで次のコマンドを実行します。

   ```Plain
   sh spark-shell
   ```

2. 次のコマンドを実行して、`test` データベースに属する `score_board` テーブルに `starrocksSparkRDD` という名前の RDD を作成します。

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

3. RDD からデータを読み取ります。たとえば、最初の 10 要素を読み取りたい場合は、次のように実行します。

   ```Scala
   scala> starrocksSparkRDD.take(10)
   ```

   Spark は次のデータを返します。

   ```Scala
   res0: Array[AnyRef] = Array([1, Bob, 21], [2, Stan, 21], [3, Sam, 22], [4, Tony, 22], [5, Alice, 22], [6, Lucy, 23], [7, Polly, 23], [8, Tom, 23], [9, Rose, 24], [10, Jerry, 24])
   ```

   全体の RDD を読み取るには、次のように実行します。

   ```Scala
   scala> starrocksSparkRDD.collect()
   ```

   Spark は次のデータを返します。

   ```Scala```
```java
    res1: Array[AnyRef] = Array([1, Bob, 21], [2, Stan, 21], [3, Sam, 22], [4, Tony, 22], [5, Alice, 22], [6, Lucy, 23], [7, Polly, 23], [8, Tom, 23], [9, Rose, 24], [10, Jerry, 24], [11, Jason, 24], [12, Lily, 25], [13, Stephen, 25], [14, David, 25], [15, Eddie, 26], [16, Kate, 27], [17, Cathy, 27], [18, Judy, 27], [19, Julia, 28], [20, Robert, 28], [21, Jack, 29])
   ```

## ベストプラクティス

Sparkコネクターを使用してStarRocksからデータを読み込む際、`starrocks.filter.query`パラメーターを使用して、Sparkがパーティション、バケツ、およびプレフィックスインデックスを刈ることに基づいたフィルタ条件を指定することで、データの取得コストを削減することができます。このセクションでは、Spark DataFrameを使用して、この方法がどのように実現されるかを示します。

### 環境の設定

| コンポーネント  | バージョン                                                  |
| --------------- | ------------------------------------------------------------ |
| Spark           | Spark 2.4.4およびScala 2.11.12（OpenJDK 64ビットサーバVM、Java 1.8.0_302） |
| StarRocks       | 2.2.0                                                        |
| Sparkコネクター | starrocks-spark2_2.11-1.0.0.jar                              |

### データの例

次の手順でサンプルテーブルを準備します。

1. `test`データベースに移動し、`mytable`という名前のテーブルを作成します。

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

1. 「Spark」ディレクトリで以下のコマンドを実行して、`test`データベースに属する`mytable`テーブル上に`df`という名前のDataFrameを作成します。

   ```Scala
   scala>  val df = spark.read.format("starrocks")
           .option("starrocks.table.identifier", s"test.mytable")
           .option("starrocks.fenodes", s"<fe_host>:<fe_http_port>")
           .option("user", s"root")
           .option("password", s"")
           .load()
   ```

2. StarRocksクラスターのFEログファイル**fe.log**を表示し、データを読み込むために実行されたSQLステートメントを見つけます。例:

   ```SQL
   2022-08-09 18:57:38,091 INFO (nioEventLoopGroup-3-10|196) [TableQueryPlanAction.executeWithoutPassword():126] receive SQL statement [select `k`,`b`,`dt`,`v` from `test`.`mytable`] from external service [ user ['root'@'%']] for database [test] table [mytable]
   ```

3. `test`データベースで、`test`.`mytable`からのSELECT `k`,`b`,`dt`,`v`の実行計画を取得するためにEXPLAINを使用します。

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

この例では、フィルタリングが行われていません。そのため、Sparkはデータを保持している3つのパーティション（`partitions=3/3`との指摘による）およびそれらの3つのパーティションに含まれるすべての9つのタブレット（`tabletRatio=9/9`との指摘による）をすべてスキャンします。

### パーティションの刈り込み

1. 以下のコマンドを実行し、パーティションの刈り取りのためにフィルタ条件`dt='2022-01-02 08:00:00`を指定するように`starrocks.filter.query`パラメーターを使用して、`test`データベースに属する`mytable`テーブル上に`df`という名前のDataFrameを作成します。

   ```Scala
   scala> val df = spark.read.format("starrocks")
          .option("starrocks.table.identifier", s"test.mytable")
          .option("starrocks.fenodes", s"<fe_host>:<fe_http_port>")
          .option("user", s"root")
          .option("password", s"")
          .option("starrocks.filter.query", "dt='2022-01-02 08:00:00'")
          .load()
   ```

2. StarRocksクラスターのFEログファイル**fe.log**を表示し、データを読み込むために実行されたSQLステートメントを見つけます。例:

   ```SQL
   2022-08-09 19:02:31,253 INFO (nioEventLoopGroup-3-14|204) [TableQueryPlanAction.executeWithoutPassword():126] receive SQL statement [select `k`,`b`,`dt`,`v` from `test`.`mytable` where dt='2022-01-02 08:00:00'] from external service [ user ['root'@'%']] for database [test] table [mytable]
   ```

3. `test`データベースで、`test`.`mytable`からのSELECT `k`,`b`,`dt`,`v` where dt='2022-01-02 08:00:00'の実行計画を取得するためにEXPLAINを使用します。

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
```+ {R}
  + {R}
+ {R}
  + {R}
```
   |    1 |   11 | 2022-01-02 08:00:00 |  111 |
   |    3 |   33 | 2022-01-02 08:00:00 |  333 |
   |    3 |   33 | 2022-01-02 08:00:00 |  333 |
   |    3 |   33 | 2022-01-02 08:00:00 |  333 |
   |    2 |   22 | 2022-02-02 08:00:00 |  222 |
   |    3 |   33 | 2022-03-02 08:00:00 |  333 |
   +------+------+---------------------+------+
   7 rows in set (0.01 sec)
   ```

3. Run the following command, in which you use the `starrocks.filter.query` parameter to specify a filter condition `k=1` for prefix index filtering, in the Spark directory to create a DataFrame named `df` on the `mytable` table which belongs to the `test` database:

   ```Scala
   scala> val df = spark.read.format("starrocks")
          .option("starrocks.table.identifier", s"test.mytable")
          .option("starrocks.fenodes", s"<fe_host>:<fe_http_port>")
          .option("user", s"root")
          .option("password", s"")
          .option("starrocks.filter.query", "k=1")
          .load()
   ```

4. In the `test` database, set `is_report_success` to `true` to enable profile reporting:

   ```SQL
   MySQL [test]> SET is_report_success = true;
   Query OK, 0 rows affected (0.00 sec)
   ```

5. Use a browser to open the `http://<fe_host>:<http_http_port>/query` page, and view the profile of the SELECT * FROM mytable where k=1 statement. Example:

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