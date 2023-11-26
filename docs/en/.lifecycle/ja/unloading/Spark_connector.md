---
displayed_sidebar: "Japanese"
---

# StarRocks Sparkコネクタを使用してStarRocksからデータを読み取る

StarRocksは、Apache Spark™（以下、Sparkコネクタ）用のStarRocksコネクタという自社開発のコネクタを提供しており、これを使用してSparkを使用してStarRocksテーブルからデータを読み取ることができます。Sparkを使用して、StarRocksから読み取ったデータに対して複雑な処理や機械学習を行うことができます。

Sparkコネクタは、Spark SQL、Spark DataFrame、Spark RDDの3つの読み取り方法をサポートしています。

Spark SQLを使用して、StarRocksテーブルに一時ビューを作成し、その一時ビューを使用してStarRocksテーブルから直接データを読み取ることができます。

また、StarRocksテーブルをSpark DataFrameまたはSpark RDDにマッピングし、Spark DataFrameまたはSpark RDDからデータを読み取ることもできます。Spark DataFrameの使用をお勧めします。

> **注意**
>
> StarRocksテーブルのSELECT権限を持つユーザーのみがこのテーブルからデータを読み取ることができます。ユーザーに権限を付与するための手順については、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)で提供される手順に従ってください。

## 使用上の注意事項

- データを読み取る前に、StarRocksでデータをフィルタリングすることで、転送するデータ量を減らすことができます。
- データの読み取りのオーバーヘッドが大きい場合は、適切なテーブル設計とフィルタ条件を使用して、Sparkが一度に過剰なデータを読み取るのを防ぐことができます。これにより、ディスクとネットワーク接続のI/O負荷を軽減し、通常のクエリが正常に実行できるようにします。

## バージョン要件

| Sparkコネクタ | Spark         | StarRocks       | Java | Scala |
|---------------- | ------------- | --------------- | ---- | ----- |
| 1.1.1           | 3.2, 3.3, 3.4 | 2.5以降   | 8    | 2.12  |
| 1.1.0           | 3.2, 3.3, 3.4 | 2.5以降   | 8    | 2.12  |
| 1.0.0           | 3.x           | 1.18以降  | 8    | 2.12  |
| 1.0.0           | 2.x           | 1.18以降  | 8    | 2.11  |

> **注意**
>
> - 異なるコネクタバージョン間の動作の変更については、[Sparkコネクタのアップグレード](#upgrade-spark-connector)を参照してください。
> - バージョン1.1.1以降、コネクタはMySQL JDBCドライバを提供していません。そのため、ドライバをSparkのクラスパスに手動でインポートする必要があります。ドライバは[Maven Central](https://repo1.maven.org/maven2/mysql/mysql-connector-java/)で入手できます。
> - バージョン1.0.0では、SparkコネクタはStarRocksからのデータの読み取りのみをサポートしています。バージョン1.1.0以降、SparkコネクタはStarRocksからのデータの読み取りと書き込みの両方をサポートしています。
> - バージョン1.0.0と1.1.0では、パラメータとデータ型のマッピングが異なります。[Sparkコネクタのアップグレード](#upgrade-spark-connector)を参照してください。
> - 一般的な場合、バージョン1.0.0には新機能は追加されません。できるだけ早くSparkコネクタをアップグレードすることをお勧めします。

## Sparkコネクタの取得

ビジネスニーズに合ったSparkコネクタ**.jar**パッケージを取得するには、次のいずれかの方法を使用します。

- コンパイル済みパッケージをダウンロードします。
- Mavenを使用してSparkコネクタに必要な依存関係を追加します（この方法はSparkコネクタ1.1.0以降のみサポートされています）。
- パッケージを手動でコンパイルします。

### Sparkコネクタ1.1.0以降

Sparkコネクタ**.jar**パッケージの名前は、次の形式で命名されます。

`starrocks-spark-connector-${spark_version}_${scala_version}-${connector_version}.jar`

たとえば、Sparkコネクタ1.1.0をSpark 3.2とScala 2.12で使用する場合、`starrocks-spark-connector-3.2_2.12-1.1.0.jar`を選択できます。

> **注意**
>
> 通常の場合、最新のSparkコネクタバージョンは、最新の3つのSparkバージョンと一緒に使用できます。

#### コンパイル済みパッケージをダウンロードする

さまざまなバージョンのSparkコネクタ**.jar**パッケージを[Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks)で入手できます。

#### Mavenの依存関係を追加する

次のように、Sparkコネクタに必要な依存関係を設定します。

> **注意**
>
> `spark_version`、`scala_version`、`connector_version`を使用して、使用するSparkバージョン、Scalaバージョン、およびSparkコネクタバージョンに置き換える必要があります。

```xml
<dependency>
  <groupId>com.starrocks</groupId>
  <artifactId>starrocks-spark-connector-${spark_version}_${scala_version}</artifactId>
  <version>${connector_version}</version>
</dependency>
```

たとえば、Sparkコネクタ1.1.0をSpark 3.2とScala 2.12で使用する場合、次のように依存関係を設定します。

```xml
<dependency>
  <groupId>com.starrocks</groupId>
  <artifactId>starrocks-spark-connector-3.2_2.12</artifactId>
  <version>1.1.0</version>
</dependency>
```

#### パッケージを手動でコンパイルする

1. [Sparkコネクタのコード](https://github.com/StarRocks/starrocks-connector-for-apache-spark)をダウンロードします。

2. 次のコマンドを使用して、Sparkコネクタをコンパイルします。

   > **注意**
   >
   > `spark_version`を使用して、使用するSparkバージョンに置き換える必要があります。

   ```shell
   sh build.sh <spark_version>
   ```

   たとえば、Spark 3.2でSparkコネクタを使用する場合、次のようにSparkコネクタをコンパイルします。

   ```shell
   sh build.sh 3.2
   ```

3. `target/`パスに移動し、コンパイル時に生成された`starrocks-spark-connector-3.2_2.12-1.1.0-SNAPSHOT.jar`のようなSparkコネクタ**.jar**パッケージが生成されます。

   > **注意**
   >
   > 公式にリリースされていないSparkコネクタバージョンを使用している場合、生成されたSparkコネクタ**.jar**パッケージの名前には、サフィックスとして`SNAPSHOT`が含まれます。

### Sparkコネクタ1.0.0

#### コンパイル済みパッケージをダウンロードする

- [Spark 2.x](https://cdn-thirdparty.starrocks.com/spark/starrocks-spark2_2.11-1.0.0.jar)
- [Spark 3.x](https://cdn-thirdparty.starrocks.com/spark/starrocks-spark3_2.12-1.0.0.jar)

#### パッケージを手動でコンパイルする

1. [Sparkコネクタのコード](https://github.com/StarRocks/starrocks-connector-for-apache-spark/tree/spark-1.0)をダウンロードします。

   > **注意**
   >
   > `spark-1.0`に切り替える必要があります。

2. Sparkコネクタをコンパイルするには、次のいずれかの操作を実行します。

   - Spark 2.xを使用している場合は、次のコマンドを実行します。このコマンドは、デフォルトでSpark 2.3.4に合わせてSparkコネクタをコンパイルします。

     ```Plain
     sh build.sh 2
     ```

   - Spark 3.xを使用している場合は、次のコマンドを実行します。このコマンドは、デフォルトでSpark 3.1.2に合わせてSparkコネクタをコンパイルします。

     ```Plain
     sh build.sh 3
     ```

3. `output/`パスに移動し、コンパイル時に`starrocks-spark2_2.11-1.0.0.jar`ファイルが生成されます。次に、ファイルをSparkのクラスパスにコピーします。

   - Sparkクラスタが`Local`モードで実行されている場合は、ファイルを`jars/`パスに配置します。
   - Sparkクラスタが`Yarn`モードで実行されている場合は、ファイルを事前デプロイメントパッケージに配置します。

指定された場所にファイルを配置した後、Sparkコネクタを使用してStarRocksからデータを読み取ることができます。

## パラメータ

このセクションでは、Sparkコネクタを使用してStarRocksからデータを読み取る際に設定する必要のあるパラメータについて説明します。

### 共通パラメータ

次のパラメータは、Spark SQL、Spark DataFrame、Spark RDDのすべての読み取り方法に適用されます。

| パラメータ                            | デフォルト値     | 説明                                                    |
| ------------------------------------ | ----------------- | ------------------------------------------------------------ |
| starrocks.fenodes                    | None              | StarRocksクラスタのFEのHTTP URL。フォーマットは`<fe_host>:<fe_http_port>`です。複数のURLを指定できます。URLはカンマ（,）で区切る必要があります。 |
| starrocks.table.identifier           | None              | StarRocksテーブルの名前。フォーマットは`<database_name>.<table_name>`です。 |
| starrocks.request.retries            | 3                 | SparkがStarRocksに読み取りリクエストを送信しようとする最大試行回数。 |
| starrocks.request.connect.timeout.ms | 30000             | StarRocksに送信された読み取りリクエストがタイムアウトするまでの最大時間。 |
| starrocks.request.read.timeout.ms    | 30000             | StarRocksに送信されたリクエストの読み取りがタイムアウトするまでの最大時間。 |
| starrocks.request.query.timeout.s    | 3600              | StarRocksからのデータクエリがタイムアウトするまでの最大時間。デフォルトのタイムアウト期間は1時間です。`-1`はタイムアウト期間が指定されていないことを意味します。 |
| starrocks.request.tablet.size        | Integer.MAX_VALUE | Spark RDDパーティションごとにグループ化されるStarRocksタブレットの数。このパラメータの値が小さいほど、より多くのSpark RDDパーティションが生成されます。Spark RDDパーティションの数が多いほど、Spark上の並列処理が高くなりますが、StarRocksに対する圧力も高くなります。 |
| starrocks.batch.size                 | 4096              | 1回のリクエストでBEから読み取ることができる最大行数。このパラメータの値を増やすと、SparkとStarRocksの間で確立される接続数が減少し、ネットワークの遅延による余分な時間オーバーヘッドを軽減することができます。 |
| starrocks.exec.mem.limit             | 2147483648        | クエリごとに許可される最大メモリ量。単位：バイト。デフォルトのメモリ制限は2 GBです。 |
| starrocks.deserialize.arrow.async    | false             | Arrowメモリ形式をSparkコネクタの反復に必要なRowBatchesに非同期に変換するかどうかを指定します。 |
| starrocks.deserialize.queue.size     | 64                | Arrowメモリ形式をRowBatchesに非同期に変換するためのタスクを保持する内部キューのサイズ。このパラメータは、`starrocks.deserialize.arrow.async`が`true`に設定されている場合に有効です。 |
| starrocks.filter.query               | None              | StarRocksでデータをフィルタリングするための条件。複数のフィルタ条件を指定できますが、`and`で結合する必要があります。Sparkがデータを読み取る前に、指定したフィルタ条件に基づいてStarRocksテーブルからデータをフィルタリングします。 |
| starrocks.timezone | JVMのデフォルトタイムゾーン | 1.1.1以降でサポートされています。StarRocksの`DATETIME`をSparkの`TimestampType`に変換するために使用されるタイムゾーン。デフォルトは`ZoneId#systemDefault()`によって返されるJVMのタイムゾーンです。フォーマットは`Asia/Shanghai`などのタイムゾーン名、または`+08:00`などのゾーンオフセットです。 |

### Spark SQLおよびSpark DataFrame用のパラメータ

次のパラメータは、Spark SQLおよびSpark DataFrameの読み取り方法にのみ適用されます。

| パラメータ                           | デフォルト値 | 説明                                                    |
| ----------------------------------- | ------------- | ------------------------------------------------------------ |
| starrocks.fe.http.url               | None          | FEのHTTP IPアドレス。このパラメータはSparkコネクタ1.1.0以降でサポートされています。このパラメータは`starrocks.fenodes`と同等です。どちらか一方のみを設定する必要があります。Sparkコネクタ1.1.0以降では、`starrocks.fenodes`は非推奨になる可能性があるため、`starrocks.fe.http.url`を使用することをお勧めします。 |
| starrocks.fe.jdbc.url               | None          | FEのMySQLサーバーに接続するために使用されるアドレス。フォーマット：`jdbc:mysql://<fe_host>:<fe_query_port>`。<br />**注意**<br />Sparkコネクタ1.1.0以降、このパラメータは必須です。   |
| user                                | None          | StarRocksクラスタのアカウントのユーザー名。ユーザーはStarRocksテーブルの[SELECT権限](../sql-reference/sql-statements/account-management/GRANT.md)が必要です。   |
| starrocks.user                      | None          | StarRocksクラスタのアカウントのユーザー名。このパラメータはSparkコネクタ1.1.0以降でサポートされています。このパラメータは`user`と同等です。どちらか一方のみを設定する必要があります。Sparkコネクタ1.1.0以降では、`starrocks.user`を使用することをお勧めします。   |
| password                            | None          | StarRocksクラスタのアカウントのパスワード。    |
| starrocks.password                  | None          | StarRocksクラスタのアカウントのパスワード。このパラメータはSparkコネクタ1.1.0以降でサポートされています。このパラメータは`password`と同等です。どちらか一方のみを設定する必要があります。Sparkコネクタ1.1.0以降では、`starrocks.password`を使用することをお勧めします。   |
| starrocks.filter.query.in.max.count | 100           | プレディケートプッシュダウン中のIN式でサポートされる値の最大数。IN式で指定された値の数がこの制限を超える場合、IN式で指定されたフィルタ条件はSparkで処理されます。   |

### Spark RDD用のパラメータ

次のパラメータは、Spark RDDの読み取り方法にのみ適用されます。

| パラメータ                       | デフォルト値 | 説明                                                    |
| ------------------------------- | ------------- | ------------------------------------------------------------ |
| starrocks.request.auth.user     | None          | StarRocksクラスタのアカウントのユーザー名。              |
| starrocks.request.auth.password | None          | StarRocksクラスタのアカウントのパスワード。              |
| starrocks.read.field            | None          | データを読み取るためにStarRocksテーブルの列。複数の列を指定できますが、カンマ（,）で区切る必要があります。 |

## StarRocksとSparkのデータ型のマッピング

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

StarRocksが使用する基礎ストレージエンジンの処理ロジックは、DATEおよびDATETIMEデータ型を直接使用した場合に期待される時間範囲をカバーすることができません。そのため、Sparkコネクタは、StarRocksのDATEおよびDATETIMEデータ型をSparkのSTRINGデータ型にマッピングし、StarRocksから読み取った日付と時刻データに一致する読みやすい文字列テキストを生成します。

## Sparkコネクタのアップグレード

### バージョン1.0.0からバージョン1.1.0へのアップグレード

- 1.1.1以降、Sparkコネクタは、GPLライセンスを使用している`mysql-connector-java`の制限により、公式のMySQL JDBCドライバ`mysql-connector-java`を提供していません。
  ただし、Sparkコネクタは、テーブルメタデータを取得するためにStarRocksに接続するために`mysql-connector-java`が必要です。そのため、ドライバをSparkのクラスパスに手動で追加する必要があります。ドライバは[MySQLサイト](https://dev.mysql.com/downloads/connector/j/)または[Maven Central](https://repo1.maven.org/maven2/mysql/mysql-connector-java/)で入手できます。
  
- バージョン1.1.0では、SparkコネクタはJDBCを使用してStarRocksにアクセスし、より詳細なテーブル情報を取得します。そのため、`starrocks.fe.jdbc.url`を設定する必要があります。

- バージョン1.1.0では、一部のパラメータの名前が変更されました。古いパラメータと新しいパラメータの両方が現在も保持されています。対応するパラメータのペアごとに、どちらか一方のみを設定する必要がありますが、古いパラメータは非推奨になる可能性があるため、新しいパラメータを使用することをお勧めします。
  - `starrocks.fenodes`は`starrocks.fe.http.url`に名前が変更されました。
  - `user`は`starrocks.user`に名前が変更されました。
  - `password`は`starrocks.password`に名前が変更されました。

- バージョン1.1.0では、一部のデータ型のマッピングがSpark 3.xに基づいて調整されました。
  - StarRocksの`DATE`はSparkの`DataTypes.DateType`（元々は`DataTypes.StringType`）にマッピングされます。
  - StarRocksの`DATETIME`はSparkの`DataTypes.TimestampType`（元々は`DataTypes.StringType`）にマッピングされます。

## 例

以下の例では、StarRocksクラスタで`test`という名前のデータベースを作成し、ユーザー`root`の権限を持っているものとします。例のパラメータ設定は、Sparkコネクタ1.1.0を基にしています。

### データの例

次の手順でサンプルテーブルを準備します。

1. `test`データベースに移動し、`score_board`という名前のテーブルを作成します。

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

3. `score_board`テーブルをクエリします。

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

1. Sparkディレクトリで次のコマンドを実行して、Spark SQLを起動します。

   ```Plain
   sh spark-sql
   ```

2. 次のコマンドを実行して、`test.score_board`に対して`spark_starrocks`という名前の一時ビューを作成します。

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

   Sparkは次のデータを返します。

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

1. Sparkディレクトリで次のコマンドを実行して、Sparkシェルを起動します。

   ```Plain
   sh spark-shell
   ```

2. 次のコマンドを実行して、`test.score_board`に対して`starrocksSparkDF`という名前のDataFrameを作成します。

   ```Scala
   scala> val starrocksSparkDF = spark.read.format("starrocks")
              .option("starrocks.table.identifier", s"test.score_board")
              .option("starrocks.fe.http.url", s"<fe_host>:<fe_http_port>")
              .option("starrocks.fe.jdbc.url", s"jdbc:mysql://<fe_host>:<fe_query_port>")
              .option("starrocks.user", s"root")
              .option("starrocks.password", s"")
              .load()
   ```

3. DataFrameからデータを読み取ります。たとえば、最初の10行を読み取る場合は、次のコマンドを実行します。

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
   > 読み取る行数を指定しない場合、Sparkはデフォルトで最初の20行を返します。

### Spark RDDを使用してデータを読み取る

1. Sparkディレクトリで次のコマンドを実行して、Sparkシェルを起動します。

   ```Plain
   sh spark-shell
   ```

2. 次のコマンドを実行して、`test.score_board`に対して`starrocksSparkRDD`という名前のRDDを作成します。

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

3. RDDからデータを読み取ります。たとえば、最初の10要素を読み取る場合は、次のコマンドを実行します。

   ```Scala
   scala> starrocksSparkRDD.take(10)
   ```

   Sparkは次のデータを返します。

   ```Scala
   res0: Array[AnyRef] = Array([1, Bob, 21], [2, Stan, 21], [3, Sam, 22], [4, Tony, 22], [5, Alice, 22], [6, Lucy, 23], [7, Polly, 23], [8, Tom, 23], [9, Rose, 24], [10, Jerry, 24])
   ```

   RDD全体を読み取るには、次のコマンドを実行します。

   ```Scala
   scala> starrocksSparkRDD.collect()
   ```

   Sparkは次のデータを返します。

   ```Scala
   res1: Array[AnyRef] = Array([1, Bob, 21], [2, Stan, 21], [3, Sam, 22], [4, Tony, 22], [5, Alice, 22], [6, Lucy, 23], [7, Polly, 23], [8, Tom, 23], [9, Rose, 24], [10, Jerry, 24], [11, Jason, 24], [12, Lily, 25], [13, Stephen, 25], [14, David, 25], [15, Eddie, 26], [16, Kate, 27], [17, Cathy, 27], [18, Judy, 27], [19, Julia, 28], [20, Robert, 28], [21, Jack, 29])
   ```

## ベストプラクティス

Sparkコネクタを使用してStarRocksからデータを読み取る際に、データのプルコストを削減するために、Sparkがパーティション、バケット、およびプレフィックスインデックスを削減するためのフィルタ条件を指定するために、`starrocks.filter.query`パラメータを使用することができます。このセクションでは、Spark DataFrameを使用して、これを実現する方法を示します。

### 環境のセットアップ

| コンポーネント | バージョン                                                      |
| --------------- | ------------------------------------------------------------ |
| Spark           | Spark 2.4.4およびScala 2.11.12（OpenJDK 64-Bit Server VM、Java 1.8.0_302） |
| StarRocks       | 2.2.0                                                        |
| Sparkコネクタ | starrocks-spark2_2.11-1.0.0.jar                              |

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

1. Sparkディレクトリで以下のコマンドを実行し、`test`データベースの`mytable`テーブルに対して`df`という名前のDataFrameを作成します。

   ```Scala
   scala>  val df = spark.read.format("starrocks")
           .option("starrocks.table.identifier", s"test.mytable")
           .option("starrocks.fenodes", s"<fe_host>:<fe_http_port>")
           .option("user", s"root")
           .option("password", s"")
           .load()
   ```

2. StarRocksクラスタのFEログファイル**fe.log**を表示し、データを読み取るために実行されたSQLステートメントを見つけます。例：

   ```SQL
   2022-08-09 18:57:38,091 INFO (nioEventLoopGroup-3-10|196) [TableQueryPlanAction.executeWithoutPassword():126] receive SQL statement [select `k`,`b`,`dt`,`v` from `test`.`mytable`] from external service [ user ['root'@'%']] for database [test] table [mytable]
   ```

3. `test`データベースで、SELECT `k`,`b`,`dt`,`v` from `test`.`mytable`ステートメントの実行計画を取得するためにEXPLAINを使用します。

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
   26 rows in set (0.00 sec)
   ```

この例では、プルーニングは行われません。したがって、Sparkはデータを保持する3つのパーティション（`partitions=3/3`で示される）およびそれらの3つのパーティション内の9つのタブレット（`tabletRatio=9/9`で示される）をすべてスキャンします。

### パーティションプルーニング

1. Sparkディレクトリで以下のコマンドを実行し、`starrocks.filter.query`パラメータを使用してパーティションプルーニングのためのフィルタ条件`dt='2022-01-02 08:00:00'`を指定して、`test`データベースの`mytable`テーブルに対して`df`という名前のDataFrameを作成します。

   ```Scala
   scala> val df = spark.read.format("starrocks")
          .option("starrocks.table.identifier", s"test.mytable")
          .option("starrocks.fenodes", s"<fe_host>:<fe_http_port>")
          .option("user", s"root")
          .option("password", s"")
          .option("starrocks.filter.query", "dt='2022-01-02 08:00:00'")
          .load()
   ```

2. StarRocksクラスタのFEログファイル**fe.log**を表示し、データを読み取るために実行されたSQLステートメントを見つけます。例：

   ```SQL
   2022-08-09 19:02:31,253 INFO (nioEventLoopGroup-3-14|204) [TableQueryPlanAction.executeWithoutPassword():126] receive SQL statement [select `k`,`b`,`dt`,`v` from `test`.`mytable` where dt='2022-01-02 08:00:00'] from external service [ user ['root'@'%']] for database [test] table [mytable]
   ```

3. `test`データベースで、SELECT `k`,`b`,`dt`,`v` from `test`.`mytable` where dt='2022-01-02 08:00:00'ステートメントの実行計画を取得するためにEXPLAINを使用します。

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

この例では、パーティションプルーニングのみが行われ、バケットプルーニングは行われません。したがって、Sparkはデータを保持する3つのパーティション（`partitions=1/3`で示される）のうちの1つと、そのパーティション内のすべてのタブレット（`tabletRatio=3/3`で示される）をスキャンします。

### バケットプルーニング

1. Sparkディレクトリで以下のコマンドを実行し、`starrocks.filter.query`パラメータを使用してバケットプルーニングのためのフィルタ条件`k=1`を指定して、`test`データベースの`mytable`テーブルに対して`df`という名前のDataFrameを作成します。

   ```Scala
   scala> val df = spark.read.format("starrocks")
          .option("starrocks.table.identifier", s"test.mytable")
          .option("starrocks.fenodes", s"<fe_host>:<fe_http_port>")
          .option("user", s"root")
          .option("password", s"")
          .option("starrocks.filter.query", "k=1")
          .load()
   ```

2. StarRocksクラスタのFEログファイル**fe.log**を表示し、データを読み取るために実行されたSQLステートメントを見つけます。例：

   ```SQL
   2022-08-09 19:04:44,479 INFO (nioEventLoopGroup-3-16|208) [TableQueryPlanAction.executeWithoutPassword():126] receive SQL statement [select `k`,`b`,`dt`,`v` from `test`.`mytable` where k=1] from external service [ user ['root'@'%']] for database [test] table [mytable]
   ```

3. `test`データベースで、SELECT `k`,`b`,`dt`,`v` from `test`.`mytable` where k=1ステートメントの実行計画を取得するためにEXPLAINを使用します。

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

この例では、バケットプルーニングのみが行われ、パーティションプルーニングは行われません。したがって、Sparkはデータを保持する3つのパーティション（`partitions=3/3`で示される）およびそれらの3つのパーティション内の3つのタブレット（`tabletRatio=3/9`で示される）をスキャンし、これらの3つのパーティション内で`k = 1`フィルタ条件に一致するハッシュ値を取得します。

### パーティションプルーニングとバケットプルーニング

1. Sparkディレクトリで以下のコマンドを実行し、`starrocks.filter.query`パラメータを使用してパーティションプルーニングとバケットプルーニングのためのフィルタ条件`k=7`および`dt='2022-01-02 08:00:00'`を指定して、`test`データベースの`mytable`テーブルに対して`df`という名前のDataFrameを作成します。

   ```Scala
   scala> val df = spark.read.format("starrocks")
          .option("starrocks.table.identifier", s"test.mytable")
          .option("starrocks.fenodes", s"<fe_host>:<fe_http_port>")
          .option("user", s"")
          .option("password", s"")
          .option("starrocks.filter.query", "k=7 and dt='2022-01-02 08:00:00'")
          .load()
   ```

2. StarRocksクラスタのFEログファイル**fe.log**を表示し、データを読み取るために実行されたSQLステートメントを見つけます。例：

   ```SQL
   2022-08-09 19:06:34,939 INFO (nioEventLoopGroup-3-18|212) [TableQueryPlanAction.executeWithoutPassword():126] receive SQL statement [select `k`,`b`,`dt`,`v` from `test`.`mytable` where k=7 and dt='2022-01-02 08:00:00'] from external service [ user ['root'@'%']] for database [test] t
   able [mytable]
   ```

3. `test`データベースで、SELECT `k`,`b`,`dt`,`v` from `test`.`mytable` where k=7 and dt='2022-01-02 08:00:00'ステートメントの実行計画を取得するためにEXPLAINを使用します。

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

この例では、パーティションプルーニングとバケットプルーニングの両方が行われます。したがって、Sparkはデータを保持する3つのパーティション（`partitions=1/3`で示される）のうちの1つと、そのパーティション内の1つのタブレット（`tabletRatio=1/3`で示される）をスキャンします。

### プレフィックスインデックスフィルタリング

1. `mytable`テーブルのパーティションにさらにデータレコードを挿入します。

   ```Scala
   MySQL [test]> INSERT INTO mytable
   VALUES
       (1, 11, "2022-01-02 08:00:00", 111), 
       (3, 33, "2022-01-02 08:00:00", 333), 
       (3, 33, "2022-01-02 08:00:00", 333), 
       (3, 33, "2022-01-02 08:00:00", 333);
   ```

2. `mytable`テーブルをクエリします。

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

3. Sparkディレクトリで以下のコマンドを実行し、`starrocks.filter.query`パラメータを使用してプレフィックスインデックスフィルタリングのためのフィルタ条件`k=1`を指定して、`test`データベースの`mytable`テーブルに対して`df`という名前のDataFrameを作成します。

   ```Scala
   scala> val df = spark.read.format("starrocks")
          .option("starrocks.table.identifier", s"test.mytable")
          .option("starrocks.fenodes", s"<fe_host>:<fe_http_port>")
          .option("user", s"root")
          .option("password", s"")
          .option("starrocks.filter.query", "k=1")
          .load()
   ```

4. `test`データベースで、プロファイルレポートを有効にするために`is_report_success`を`true`に設定します。

   ```SQL
   MySQL [test]> SET is_report_success = true;
   Query OK, 0 rows affected (0.00 sec)
   ```

5. ブラウザを使用して`http://<fe_host>:<http_http_port>/query`ページを開き、SELECT * FROM mytable where k=1ステートメントのプロファイルを表示します。例：

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

この例では、フィルタ条件`k = 1`はプレフィックスインデックスにヒットするため、3行（`ShortKeyFilterRows: 3`で示される）をフィルタリングすることができます。
