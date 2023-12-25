---
displayed_sidebar: English
---

# Spark コネクタを使用して StarRocks からデータを読み取る

StarRocks は、Spark を使用して StarRocks テーブルからデータを読み取るための自社開発のコネクタである StarRocks Connector for Apache Spark™（通称 Spark コネクタ）を提供しています。Spark は、StarRocks から読み取ったデータに対して複雑な処理や機械学習を行うために使用できます。

Spark コネクタは、Spark SQL、Spark DataFrame、Spark RDD の 3 つの読み取り方法をサポートしています。

Spark SQL を使用して StarRocks テーブルに一時ビューを作成し、その一時ビューを使用して StarRocks テーブルからデータを直接読み取ることができます。

また、StarRocks テーブルを Spark DataFrame または Spark RDD にマッピングし、それらからデータを読み取ることもできます。Spark DataFrame の使用を推奨します。

> **注意**
>
> StarRocks テーブルに対する SELECT 権限を持つユーザーのみが、このテーブルからデータを読み取ることができます。ユーザーに権限を付与する方法については、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md) の指示に従ってください。

## 使用上の注意

- データを読み取る前に StarRocks でデータをフィルタリングすることで、転送されるデータ量を減らすことができます。
- データ読み取りのオーバーヘッドが大きい場合は、適切なテーブル設計とフィルター条件を使用して、Spark が一度に過剰な量のデータを読み取らないようにすることができます。これにより、ディスクとネットワーク接続の I/O 圧力を軽減し、日常的なクエリを適切に実行できるようになります。

## バージョン要件

| Spark コネクタ | Spark         | StarRocks       | Java | Scala |
|---------------- | ------------- | --------------- | ---- | ----- |
| 1.1.1           | 3.2, 3.3, 3.4 | 2.5 以降   | 8    | 2.12  |
| 1.1.0           | 3.2, 3.3, 3.4 | 2.5 以降   | 8    | 2.12  |
| 1.0.0           | 3.x           | 1.18 以降  | 8    | 2.12  |
| 1.0.0           | 2.x           | 1.18 以降  | 8    | 2.11  |

> **注意**
>
> - 異なるコネクタ バージョン間の動作変更については、「[Spark コネクタのアップグレード](#upgrade-spark-connector)」を参照してください。
> - コネクタはバージョン 1.1.1 以降、MySQL JDBC ドライバーを提供していません。ドライバーを Spark クラスパスに手動でインポートする必要があります。ドライバーは [Maven Central](https://repo1.maven.org/maven2/mysql/mysql-connector-java/) で見つけることができます。
> - バージョン 1.0.0 では、Spark コネクタは StarRocks からのデータ読み取りのみをサポートしています。バージョン 1.1.0 以降、Spark コネクタは StarRocks からのデータ読み取りと StarRocks へのデータ書き込みの両方をサポートしています。
> - バージョン 1.0.0 は、パラメーターとデータ型マッピングの点でバージョン 1.1.0 と異なります。詳細については、「[Spark コネクタのアップグレード](#upgrade-spark-connector)」を参照してください。
> - 一般的に、バージョン 1.0.0 に新機能は追加されません。できるだけ早く Spark コネクタをアップグレードすることを推奨します。

## Spark コネクタを取得する

以下の方法のいずれかを使用して、ビジネスニーズに合った Spark コネクタの **.jar** パッケージを取得してください：

- コンパイル済みのパッケージをダウンロードする。
- Maven を使用して、Spark コネクタに必要な依存関係を追加する。（この方法は、Spark コネクタ 1.1.0 以降のみサポートされています。）
- パッケージを手動でコンパイルする。

### Spark コネクタ 1.1.0 以降

Spark コネクタの **.jar** パッケージは、以下の形式で命名されています：

`starrocks-spark-connector-${spark_version}_${scala_version}-${connector_version}.jar`

例えば、Spark 3.2 と Scala 2.12 を使用して Spark コネクタ 1.1.0 を使用したい場合、`starrocks-spark-connector-3.2_2.12-1.1.0.jar` を選択できます。

> **注意**
>
> 通常、最新の Spark コネクタ バージョンは、最新の 3 つの Spark バージョンと互換性があります。

#### コンパイル済みのパッケージをダウンロードする

さまざまなバージョンの Spark コネクタ **.jar** パッケージは、[Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks) で入手できます。

#### Maven 依存関係を追加する

Spark コネクタに必要な依存関係を以下のように設定します：

> **注意**
>
> `spark_version`、`scala_version`、および `connector_version` を使用している Spark バージョン、Scala バージョン、および Spark コネクタ バージョンに置き換えてください。

```xml
<dependency>
  <groupId>com.starrocks</groupId>
  <artifactId>starrocks-spark-connector-${spark_version}_${scala_version}</artifactId>
  <version>${connector_version}</version>
</dependency>
```

例えば、Spark 3.2 と Scala 2.12 を使用して Spark コネクタ 1.1.0 を使用する場合、依存関係を以下のように設定します：

```xml
<dependency>
  <groupId>com.starrocks</groupId>
  <artifactId>starrocks-spark-connector-3.2_2.12</artifactId>
  <version>1.1.0</version>
</dependency>
```

#### パッケージを手動でコンパイルする

1. [Spark コネクタのコードをダウンロードします](https://github.com/StarRocks/starrocks-connector-for-apache-spark)。

2. 以下のコマンドを使用して Spark コネクタをコンパイルします：

   > **注意**
   >
   > 使用している Spark のバージョンで `spark_version` を置き換えてください。

   ```shell
   sh build.sh <spark_version>
   ```

   例えば、Spark 3.2 で Spark コネクタを使用する場合、以下のようにコンパイルします：

   ```shell
   sh build.sh 3.2
   ```

3. コンパイルが完了すると、`target/` ディレクトリに `starrocks-spark-connector-3.2_2.12-1.1.0-SNAPSHOT.jar` のような Spark コネクタの **.jar** パッケージが生成されます。

   > **注意**
   >
   > 正式にリリースされていないバージョンの Spark コネクタを使用している場合、生成された Spark コネクタ **.jar** パッケージの名前には `SNAPSHOT` がサフィックスとして含まれます。

### Spark コネクタ 1.0.0

#### コンパイル済みのパッケージをダウンロードする

- [Spark 2.x 用](https://cdn-thirdparty.starrocks.com/spark/starrocks-spark2_2.11-1.0.0.jar)
- [Spark 3.x 用](https://cdn-thirdparty.starrocks.com/spark/starrocks-spark3_2.12-1.0.0.jar)

#### パッケージを手動でコンパイルする

1. [Spark コネクタのコードをダウンロードします](https://github.com/StarRocks/starrocks-connector-for-apache-spark/tree/spark-1.0)。

   > **注意**
   >
   > `spark-1.0` ブランチに切り替える必要があります。

2. Spark 2.x または Spark 3.x を使用している場合、以下のコマンドを実行して Spark コネクタをコンパイルします：

   - Spark 2.x を使用している場合、以下のコマンドを実行して、デフォルトで Spark 2.3.4 に適合するように Spark コネクタをコンパイルします：

     ```shell
     sh build.sh 2
     ```

   - Spark 3.x を使用している場合、以下のコマンドを実行して、デフォルトで Spark 3.1.2 に適合するように Spark コネクタをコンパイルします：

     ```shell
     sh build.sh 3
     ```

3. コンパイルが完了すると、`output/` ディレクトリに `starrocks-spark2_2.11-1.0.0.jar` ファイルが生成されます。その後、ファイルを Spark のクラスパスにコピーします：

   - Spark クラスタが `Local` モードで実行されている場合は、ファイルを `jars/` ディレクトリに配置します。
   - Spark クラスタが `Yarn` モードで実行されている場合は、ファイルを事前デプロイメントパッケージに配置します。

ファイルを指定された場所に配置した後にのみ、Spark コネクタを使用して StarRocks からデータを読み取ることができます。

## パラメータ

このセクションでは、Spark コネクタを使用して StarRocks からデータを読み取る際に設定する必要があるパラメータについて説明します。

### 共通パラメータ

以下のパラメータは、Spark SQL、Spark DataFrame、Spark RDD の 3 つの読み取り方法すべてに適用されます。

| パラメータ                            | 既定値     | 説明                                                    |
| ------------------------------------ | ----------------- | ------------------------------------------------------------ |
| starrocks.fenodes                    | なし              | StarRocks クラスタ内の FE の HTTP URL。形式は `<fe_host>:<fe_http_port>`。複数の URL を指定する場合は、コンマ (,) で区切ってください。 |

| starrocks.table.identifier           | None              | StarRocksテーブルの名前。フォーマット: `<database_name>.<table_name>`。 |
| starrocks.request.retries            | 3                 | SparkがStarRocksへの読み取りリクエストを再送することができる最大回数。 |
| starrocks.request.connect.timeout.ms | 30000             | StarRocksへ送信された読み取りリクエストがタイムアウトするまでの最大時間。 |
| starrocks.request.read.timeout.ms    | 30000             | StarRocksへ送信されたリクエストの読み取りがタイムアウトするまでの最大時間。 |
| starrocks.request.query.timeout.s    | 3600              | StarRocksからのデータクエリがタイムアウトするまでの最大時間。デフォルトのタイムアウト期間は1時間です。`-1`はタイムアウト期間が指定されていないことを意味します。 |
| starrocks.request.tablet.size        | Integer.MAX_VALUE | 各Spark RDDパーティションにグループ化されるStarRocksタブレットの数。このパラメータの値が小さいほど、生成されるSpark RDDパーティションの数が多くなります。Spark RDDパーティションの数が多いほど、Sparkの並列性は高くなりますが、StarRocksへの負荷も大きくなります。 |
| starrocks.batch.size                 | 4096              | 一度にBEから読み取ることができる最大行数。このパラメータの値を増やすと、SparkとStarRocks間で確立される接続の数を減らすことができ、ネットワークの遅延による余分な時間オーバーヘッドを軽減できます。 |
| starrocks.exec.mem.limit             | 2147483648        | クエリごとに許可されるメモリの最大量。単位はバイト。デフォルトのメモリ制限は2GBです。 |
| starrocks.deserialize.arrow.async    | false             | ArrowメモリフォーマットをSparkコネクタのイテレーションに必要なRowBatchesに非同期で変換するかどうかを指定します。 |
| starrocks.deserialize.queue.size     | 64                | ArrowメモリフォーマットをRowBatchesに非同期で変換するタスクを保持する内部キューのサイズ。このパラメータは`starrocks.deserialize.arrow.async`が`true`に設定されている場合に有効です。 |
| starrocks.filter.query               | None              | StarRocksでデータをフィルタリングする条件。複数のフィルタ条件を指定でき、`and`で結合する必要があります。StarRocksは指定されたフィルタ条件に基づいてStarRocksテーブルからデータをフィルタリングし、その後Sparkがデータを読み取ります。 |
| starrocks.timezone                   | JVMのデフォルトタイムゾーン | 1.1.1以降でサポートされています。StarRocksの`DATETIME`をSparkの`TimestampType`に変換するために使用されるタイムゾーン。デフォルトは`ZoneId#systemDefault()`によって返されるJVMのタイムゾーンです。形式は`Asia/Shanghai`のようなタイムゾーン名、または`+08:00`のようなゾーンオフセットです。 |

### Spark SQLとSpark DataFrameのパラメータ

以下のパラメータは、Spark SQLとSpark DataFrameの読み取りメソッドにのみ適用されます。

| パラメータ                           | デフォルト値 | 説明                                                    |
| ----------------------------------- | ------------- | ------------------------------------------------------------ |
| starrocks.fe.http.url               | None          | FEのHTTP IPアドレス。このパラメータはSparkコネクタ1.1.0以降でサポートされています。`starrocks.fenodes`と同等ですが、どちらか一方のみを設定する必要があります。Sparkコネクタ1.1.0以降では、`starrocks.fenodes`が非推奨になる可能性があるため、`starrocks.fe.http.url`の使用を推奨します。 |
| starrocks.fe.jdbc.url               | None          | FEのMySQLサーバーへの接続に使用されるアドレス。フォーマット: `jdbc:mysql://<fe_host>:<fe_query_port>`。<br />**注意**<br />Sparkコネクタ1.1.0以降では、このパラメータは必須です。   |
| user                                | None          | StarRocksクラスタアカウントのユーザー名。ユーザーはStarRocksテーブルに対する[SELECT権限](../sql-reference/sql-statements/account-management/GRANT.md)が必要です。   |
| starrocks.user                      | None          | StarRocksクラスタアカウントのユーザー名。このパラメータはSparkコネクタ1.1.0以降でサポートされています。`user`と同等ですが、どちらか一方のみを設定する必要があります。Sparkコネクタ1.1.0以降では、`user`が非推奨になる可能性があるため、`starrocks.user`の使用を推奨します。   |
| password                            | None          | StarRocksクラスタアカウントのパスワード。    |
| starrocks.password                  | None          | StarRocksクラスタアカウントのパスワード。このパラメータはSparkコネクタ1.1.0以降でサポートされています。`password`と同等ですが、どちらか一方のみを設定する必要があります。Sparkコネクタ1.1.0以降では、`password`が非推奨になる可能性があるため、`starrocks.password`の使用を推奨します。   |
| starrocks.filter.query.in.max.count | 100           | 述語プッシュダウン中にIN式でサポートされる値の最大数。IN式で指定された値の数がこの制限を超える場合、IN式で指定されたフィルタ条件はSparkで処理されます。   |

### Spark RDDのパラメータ

以下のパラメータは、Spark RDD読み取りメソッドにのみ適用されます。

| パラメータ                       | デフォルト値 | 説明                                                    |
| ------------------------------- | ------------- | ------------------------------------------------------------ |
| starrocks.request.auth.user     | None          | StarRocksクラスタアカウントのユーザー名。              |
| starrocks.request.auth.password | None          | StarRocksクラスタアカウントのパスワード。              |
| starrocks.read.field            | None          | データを読み取るStarRocksテーブルの列。複数の列を指定でき、列はコンマ(,)で区切る必要があります。 |

## StarRocksとSparkのデータ型マッピング

### Sparkコネクタ1.1.0以降

| StarRocksデータ型 | Sparkデータ型           |
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

| StarRocksデータ型  | Sparkデータ型        |
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

StarRocksが使用する基盤となるストレージエンジンの処理ロジックは、DATEおよびDATETIMEデータ型が直接使用される場合に予想される時間範囲をカバーできません。そのため、Spark コネクタは、StarRocks の DATE および DATETIME データ型を Spark の STRING データ型にマップし、StarRocks から読み取った日付と時刻のデータに一致する読み取り可能な文字列テキストを生成します。

## Spark コネクタのアップグレード

### バージョン 1.0.0 からバージョン 1.1.0 へのアップグレード

- 1.1.1 以降、Spark コネクタは `mysql-connector-java` を提供しません。これは MySQL 用の公式 JDBC ドライバーであり、GPL ライセンスの制限が原因です。
  しかし、Spark コネクタはテーブルメタデータを取得するために StarRocks への接続に `mysql-connector-java` が必要ですので、ドライバーを Spark クラスパスに手動で追加する必要があります。ドライバーは [MySQL サイト](https://dev.mysql.com/downloads/connector/j/) または [Maven Central](https://repo1.maven.org/maven2/mysql/mysql-connector-java/) で見つけることができます。
  
- バージョン 1.1.0 では、Spark コネクタは JDBC を使用して StarRocks にアクセスし、より詳細なテーブル情報を取得します。したがって、`starrocks.fe.jdbc.url` を構成する必要があります。

- バージョン 1.1.0 では、いくつかのパラメータが改名されました。現在は古いパラメータと新しいパラメータの両方が保持されています。同等のパラメータのペアごとに、どちらか一方のみを設定する必要がありますが、古いパラメータは将来非推奨になる可能性があるため、新しいパラメータの使用を推奨します。
  - `starrocks.fenodes` は `starrocks.fe.http.url` に改名されました。
  - `user` は `starrocks.user` に改名されました。
  - `password` は `starrocks.password` に改名されました。

- バージョン 1.1.0 では、Spark 3.x に基づいていくつかのデータ型のマッピングが調整されました：
  - StarRocks の `DATE` は Spark の `DataTypes.DateType`（元々は `DataTypes.StringType`）にマッピングされます。
  - StarRocks の `DATETIME` は Spark の `DataTypes.TimestampType`（元々は `DataTypes.StringType`）にマッピングされます。

## 例

以下の例では、StarRocks クラスターに `test` という名前のデータベースを作成し、ユーザー `root` の権限を持っていることを前提としています。例のパラメータ設定は Spark Connector 1.1.0 に基づいています。

### データ例

サンプルテーブルを準備するには、以下の手順を実行します：

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
   MySQL [test]> INSERT INTO `score_board`
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
   MySQL [test]> SELECT * FROM `score_board`;
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

1. Spark ディレクトリで以下のコマンドを実行して Spark SQL を起動します：

   ```Plain
   sh spark-sql
   ```

2. 次のコマンドを実行して、`test` データベースに属する `score_board` テーブルの一時ビュー `spark_starrocks` を作成します：

   ```SQL
   spark-sql> CREATE TEMPORARY VIEW `spark_starrocks`
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

3. 次のコマンドを実行して一時ビューからデータを読み取ります：

   ```SQL
   spark-sql> SELECT * FROM `spark_starrocks`;
   ```

   Spark は以下のデータを返します：

   ```SQL
   1        Bob        21
   2        Stan       21
   3        Sam        22
   4        Tony       22
   5        Alice      22
   6        Lucy       23
   7        Polly      23
   8        Tom        23
   9        Rose       24
   10       Jerry      24
   11       Jason      24
   12       Lily       25
   13       Stephen    25
   14       David      25
   15       Eddie      26
   16       Kate       27
   17       Cathy      27
   18       Judy       27
   19       Julia      28
   20       Robert     28
   21       Jack       29
   Time taken: 1.883 seconds, Fetched 21 row(s)
   22/08/09 15:29:36 INFO thriftserver.SparkSQLCLIDriver: Time taken: 1.883 seconds, Fetched 21 row(s)
   ```

### Spark DataFrame を使用してデータを読み取る

1. Spark ディレクトリで以下のコマンドを実行して Spark Shell を起動します：

   ```Plain
   sh spark-shell
   ```

2. 次のコマンドを実行して、`test` データベースに属する `score_board` テーブルの DataFrame `starrocksSparkDF` を作成します：

   ```Scala
   scala> val starrocksSparkDF = spark.read.format("starrocks")
              .option("starrocks.table.identifier", "test.score_board")
              .option("starrocks.fe.http.url", "<fe_host>:<fe_http_port>")
              .option("starrocks.fe.jdbc.url", "jdbc:mysql://<fe_host>:<fe_query_port>")
              .option("starrocks.user", "root")
              .option("starrocks.password", "")
              .load()
   ```

3. DataFrame からデータを読み取ります。たとえば、最初の 10 行を読み取る場合は、以下のコマンドを実行します：

   ```Scala
   scala> starrocksSparkDF.show(10)
   ```

   Spark は以下のデータを返します：

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
   上位 10 行のみ表示
   ```

   > **注記**
   >
   > デフォルトでは、読み取る行数を指定しない場合、Spark は最初の 20 行を返します。

### Spark RDD を使用してデータを読み取る

1. Spark ディレクトリで以下のコマンドを実行して Spark Shell を起動します：

   ```Plain
   sh spark-shell
   ```

2. 次のコマンドを実行して、`test` データベースに属する `score_board` テーブルの RDD `starrocksSparkRDD` を作成します：

   ```Scala
   scala> import com.starrocks.connector.spark._
   scala> val starrocksSparkRDD = sc.starrocksRDD(
              tableIdentifier = Some("test.score_board"),
              cfg = Some(Map(
                  "starrocks.fe.http.url" -> "<fe_host>:<fe_http_port>",
                  "starrocks.request.auth.user" -> "root",
                  "starrocks.request.auth.password" -> ""
              ))
              )
   ```

3. RDD からデータを読み取ります。たとえば、最初の 10 要素を読み取る場合は、以下のコマンドを実行します：

   ```Scala
   scala> starrocksSparkRDD.take(10)
   ```

   Spark は以下のデータを返します：

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

Sparkコネクタを使用してStarRocksからデータを読み取る場合、`starrocks.filter.query`パラメータを使用してフィルター条件を指定し、Sparkがパーティション、バケット、プレフィックスインデックスをプルーニングしてデータ取得のコストを削減することができます。このセクションでは、Spark DataFrameを例にして、これを実現する方法を示します。

### 環境設定

| コンポーネント       | バージョン                                                      |
| --------------- | ------------------------------------------------------------ |
| Spark           | Spark 2.4.4 および Scala 2.11.12 (OpenJDK 64ビットサーバーVM、Java 1.8.0_302) |
| StarRocks       | 2.2.0                                                        |
| Sparkコネクタ | starrocks-spark2_2.11-1.0.0.jar                              |

### データ例

サンプルテーブルを準備するには、以下の手順を実行します。

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

1. Sparkディレクトリで次のコマンドを実行して、`test`データベースに属する`mytable`テーブルのDataFrame `df`を作成します。

   ```Scala
   scala> val df = spark.read.format("starrocks")
           .option("starrocks.table.identifier", "test.mytable")
           .option("starrocks.fenodes", "<fe_host>:<fe_http_port>")
           .option("user", "root")
           .option("password", "")
           .load()
   ```

2. StarRocksクラスターのFEログファイル**fe.log**を確認し、データを読み取るために実行されたSQLステートメントを探します。例：

   ```SQL
   2022-08-09 18:57:38,091 INFO (nioEventLoopGroup-3-10|196) [TableQueryPlanAction.executeWithoutPassword():126] receive SQL statement [select `k`,`b`,`dt`,`v` from `test`.`mytable`] from external service [ user ['root'@'%']] for database [test] table [mytable]
   ```

3. `test`データベースでEXPLAINを使用して、`SELECT `k`,`b`,`dt`,`v` from `test`.`mytable``の実行プランを取得します。

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

この例では、プルーニングは実行されません。したがって、Sparkはデータを保持する3つのパーティション（`partitions=3/3`と示される）をすべてスキャンし、これら3つのパーティション内の9つのタブレット（`tabletRatio=9/9`と示される）をすべてスキャンします。

### パーティションプルーニング

1. 次のコマンドをSparkディレクトリで実行し、`starrocks.filter.query`パラメータを使用してパーティションプルーニングのフィルター条件`dt='2022-01-02 08:00:00'`を指定し、`test`データベースに属する`mytable`テーブルのDataFrame `df`を作成します。

   ```Scala
   scala> val df = spark.read.format("starrocks")
          .option("starrocks.table.identifier", "test.mytable")
          .option("starrocks.fenodes", "<fe_host>:<fe_http_port>")
          .option("user", "root")
          .option("password", "")
          .option("starrocks.filter.query", "dt='2022-01-02 08:00:00'")
          .load()
   ```

2. StarRocksクラスターのFEログファイル**fe.log**を確認し、データを読み取るために実行されたSQLステートメントを探します。例：

   ```SQL
   2022-08-09 19:02:31,253 INFO (nioEventLoopGroup-3-14|204) [TableQueryPlanAction.executeWithoutPassword():126] receive SQL statement [select `k`,`b`,`dt`,`v` from `test`.`mytable` where dt='2022-01-02 08:00:00'] from external service [ user ['root'@'%']] for database [test] table [mytable]
   ```

3. `test`データベースでEXPLAINを使用して、`SELECT `k`,`b`,`dt`,`v` from `test`.`mytable` where dt='2022-01-02 08:00:00'`の実行プランを取得します。

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

この例では、パーティションプルーニングのみが実行され、バケットプルーニングは実行されません。そのため、Sparkは3つのパーティションのうち1つ（`partitions=1/3`と示される）と、そのパーティション内のすべてのタブレット（`tabletRatio=3/3`と示される）をスキャンします。

### バケットプルーニング

1. 次のコマンドをSparkディレクトリで実行し、`starrocks.filter.query`パラメータを使用してバケットプルーニングのフィルター条件`k=1`を指定し、`test`データベースに属する`mytable`テーブルのDataFrame `df`を作成します。

   ```Scala
   scala> val df = spark.read.format("starrocks")
          .option("starrocks.table.identifier", "test.mytable")
          .option("starrocks.fenodes", s"<fe_host>:<fe_http_port>")
          .option("user", s"root")
          .option("password", s"")
          .option("starrocks.filter.query", "k=1")
          .load()
   ```

2. StarRocksクラスターのFEログファイル**fe.log**を確認し、データ読み取りに実行されたSQLステートメントを探します。例えば：

   ```SQL
   2022-08-09 19:04:44,479 INFO (nioEventLoopGroup-3-16|208) [TableQueryPlanAction.executeWithoutPassword():126] receive SQL statement [select `k`,`b`,`dt`,`v` from `test`.`mytable` where k=1] from external service [ user ['root'@'%']] for database [test] table [mytable]
   ```

3. `test`データベースで、`EXPLAIN`を使用して`SELECT `k`,`b`,`dt`,`v` from `test`.`mytable` where k=1`ステートメントの実行計画を取得します：

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

この例では、バケットプルーニングのみが実行され、パーティションプルーニングは実行されません。そのため、Sparkはデータを保持する3つのパーティション（`partitions=3/3`で示されるように）をすべてスキャンし、3つのタブレット（`tabletRatio=3/9`で示されるように）をすべてスキャンして、これら3つのパーティション内のフィルター条件`k = 1`を満たすハッシュ値を取得します。

### パーティションプルーニングとバケットプルーニング

1. 次のコマンドを実行し、`starrocks.filter.query`パラメーターを使用して2つのフィルター条件`k=7`と`dt='2022-01-02 08:00:00'`を指定して、Sparkディレクトリで`test`データベースの`mytable`テーブルに`df`という名前のDataFrameを作成します：

   ```Scala
   scala> val df = spark.read.format("starrocks")
          .option("starrocks.table.identifier", s"test.mytable")
          .option("starrocks.fenodes", s"<fe_host>:<fe_http_port>")
          .option("user", s"")
          .option("password", s"")
          .option("starrocks.filter.query", "k=7 and dt='2022-01-02 08:00:00'")
          .load()
   ```

2. StarRocksクラスターのFEログファイル**fe.log**を確認し、データ読み取りに実行されたSQLステートメントを探します。例えば：

   ```SQL
   2022-08-09 19:06:34,939 INFO (nioEventLoopGroup-3-18|212) [TableQueryPlanAction.executeWithoutPassword():126] receive SQL statement [select `k`,`b`,`dt`,`v` from `test`.`mytable` where k=7 and dt='2022-01-02 08:00:00'] from external service [ user ['root'@'%']] for database [test] table [mytable]
   ```

3. `test`データベースで、`EXPLAIN`を使用して`SELECT `k`,`b`,`dt`,`v` from `test`.`mytable` where k=7 and dt='2022-01-02 08:00:00'`ステートメントの実行計画を取得します：

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

この例では、パーティションプルーニングとバケットプルーニングの両方が実行されます。したがって、Sparkは3つのパーティションのうち1つだけ（`partitions=1/3`で示されるように）をスキャンし、そのパーティション内の1つのタブレットのみ（`tabletRatio=1/3`で示されるように）をスキャンします。

### プレフィックスインデックスフィルタリング

1. `test`データベースの`mytable`テーブルのパーティションにさらにデータレコードを挿入します：

   ```Scala
   MySQL [test]> INSERT INTO mytable
   VALUES
       (1, 11, "2022-01-02 08:00:00", 111), 
       (3, 33, "2022-01-02 08:00:00", 333), 
       (3, 33, "2022-01-02 08:00:00", 333), 
       (3, 33, "2022-01-02 08:00:00", 333);
   ```

2. `mytable`テーブルをクエリします：

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

3. 次のコマンドをSparkディレクトリで実行し、`starrocks.filter.query`パラメーターを使用してプレフィックスインデックスフィルタリングのためのフィルタ条件`k=1`を指定して、`test`データベースの`mytable`テーブルに`df`という名前のDataFrameを作成します：

   ```Scala
   scala> val df = spark.read.format("starrocks")
          .option("starrocks.table.identifier", s"test.mytable")
          .option("starrocks.fenodes", s"<fe_host>:<fe_http_port>")
          .option("user", s"root")
          .option("password", s"")
          .option("starrocks.filter.query", "k=1")
          .load()
   ```

4. `test`データベースで、プロファイルレポートを有効にするために`is_report_success`を`true`に設定します：

   ```SQL
   MySQL [test]> SET is_report_success = true;
   Query OK, 0 rows affected (0.00 sec)
   ```

5. ブラウザを使用して`http://<fe_host>:<http_http_port>/query`ページを開き、`SELECT * FROM mytable where k=1`ステートメントのプロファイルを確認します。例：

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

この例では、フィルター条件 `k = 1` がプレフィックスインデックスにヒットするため、Sparkは3行をフィルタリングできます（`ShortKeyFilterRows: 3` に示されているように）。