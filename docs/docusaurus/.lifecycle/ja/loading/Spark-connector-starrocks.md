---
displayed_sidebar: "Japanese"
---

# Sparkコネクタを使用してデータをロードする（推奨）

StarRocksでは、Apache Spark™（以下Sparkコネクタと呼ぶ）用のStarRocksコネクタを提供し、このコネクタを使用してデータをStarRocksテーブルにロードすることができます。基本的な原則は、データを蓄積し、それをすべて一度にStarRocksに[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)することです。Sparkコネクタは、Spark DataSource V2に基づいて実装されています。DataSourceは、Spark DataFramesまたはSpark SQLを使用して作成できます。バッチおよび構造化ストリーミングモードの両方がサポートされています。

> **注意**
>
> StarRocksテーブルに対するSELECTおよびINSERT権限を持つユーザーのみが、このテーブルにデータをロードできます。ユーザーにこれらの権限を付与するための手順については、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)に記載された手順に従うことができます。

## バージョン要件

| Sparkコネクタ | Spark            | StarRocks     | Java | Scala |
| --------------- | ---------------- | ------------- | ---- | ----- |
| 1.1.1 | 3.2、3.3または3.4 | 2.5以降 | 8 | 2.12 |
| 1.1.0           | 3.2、3.3または3.4 | 2.5以降 | 8    | 2.12  |

> **注意**
>
> - Sparkコネクタの異なるバージョン間での動作の変更については、[Sparkコネクタのアップグレード](#upgrade-spark-connector)をご覧ください。
> - Sparkコネクタは、バージョン1.1.1以降、MySQL JDBCドライバを提供しておらず、ドライバをsparkのクラスパスに手動でインポートする必要があります。ドライバは[MySQLサイト](https://dev.mysql.com/downloads/connector/j/)または[Maven Central](https://repo1.maven.org/maven2/mysql/mysql-connector-java/)から入手できます。

## Sparkコネクタの取得

以下の方法でSparkコネクタのJARファイルを取得できます。

- コンパイル済みのSparkコネクタJARファイルを直接ダウンロードします。
- SparkコネクタをMavenプロジェクトの依存関係として追加し、その後JARファイルをダウンロードします。
- Sparkコネクタのソースコードを自分でJARファイルにコンパイルします。

SparkコネクタJARファイルの命名形式は `starrocks-spark-connector-${spark_version}_${scala_version}-${connector_version}.jar` です。

たとえば、環境にSpark 3.2とScala 2.12をインストールし、Sparkコネクタ1.1.0を使用したい場合は、`starrocks-spark-connector-3.2_2.12-1.1.0.jar` を使用できます。

> **注意**
>
> 一般的に、Sparkコネクタの最新バージョンは、最新のSparkの3つのバージョンとの互換性を維持します。

### コンパイル済みのJARファイルをダウンロード

[Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks)から、対応するバージョンのSparkコネクタJARを直接ダウンロードできます。

### Mavenの依存関係

1. Mavenプロジェクトの`pom.xml`ファイルに、以下の形式に従ってSparkコネクタを依存関係として追加します。`spark_version`、`scala_version`、`connector_version`をそれぞれのバージョンに置き換えます。

    ```xml
    <dependency>
    <groupId>com.starrocks</groupId>
    <artifactId>starrocks-spark-connector-${spark_version}_${scala_version}</artifactId>
    <version>${connector_version}</version>
    </dependency>
    ```

2. たとえば、環境のSparkのバージョンが3.2で、Scalaのバージョンが2.12で、Sparkコネクタ1.1.0を選択した場合、以下の依存関係を追加する必要があります。

    ```xml
    <dependency>
    <groupId>com.starrocks</groupId>
    <artifactId>starrocks-spark-connector-3.2_2.12</artifactId>
    <version>1.1.0</version>
    </dependency>
    ```

### 自分でコンパイルする

1. [Sparkコネクタパッケージ](https://github.com/StarRocks/starrocks-connector-for-apache-spark)をダウンロードします。
2. 次のコマンドを実行して、SparkコネクタのソースコードをJARファイルにコンパイルします。 `spark_version` は対応するSparkのバージョンに置き換えることに注意してください。

      ```bash
      sh build.sh <spark_version>
      ```

   たとえば、環境のSparkのバージョンが3.2の場合は、次のコマンドを実行する必要があります。

      ```bash
      sh build.sh 3.2
      ```

3. `target/`ディレクトリに移動して、コンパイル後に生成された `starrocks-spark-connector-3.2_2.12-1.1.0-SNAPSHOT.jar` などのSparkコネクタJARファイルを見つけます。

> **注意**
>
> 正式にリリースされていないSparkコネクタの名前には、 `SNAPSHOT`接尾辞が含まれています。

## パラメータ

| パラメータ                                      | 必須 | デフォルト値 | 説明                                                  |
| ---------------------------------------------- | ---- | ------------- | ------------------------------------------------------------ |
| starrocks.fe.http.url                          | YES      | None          | StarRocksクラスター内のFEのHTTP URLです。複数のURLを指定でき、その間はコンマ(,)で区切る必要があります。フォーマット: `<fe_host1>:<fe_http_port1>,<fe_host2>:<fe_http_port2>`。バージョン1.1.1以降は、URLに`http://`の接頭辞を追加することもできます。たとえば、`http://<fe_host1>:<fe_http_port1>,http://<fe_host2>:<fe_http_port2>`。|
| starrocks.fe.jdbc.url                          | YES      | None          | FEのMySQLサーバーに接続するためのアドレスです。フォーマット: `jdbc:mysql://<fe_host>:<fe_query_port>`。 |
| starrocks.table.identifier                     | YES      | None          | StarRocksテーブルの名前です。フォーマット: `<database_name>.<table_name>`。 |
| starrocks.user                                 | YES      | None          | StarRocksクラスターアカウントのユーザー名です。ユーザーには、[StarRocksテーブルのSELECTおよびINSERT権限](../sql-reference/sql-statements/account-management/GRANT.md)が必要です。             |
| starrocks.password                             | YES      | None          | StarRocksクラスターアカウントのパスワードです。              |
| starrocks.write.label.prefix                   | NO       | spark-        | Stream Loadで使用されるラベルのプレフィックスです。                        |
| starrocks.write.enable.transaction-stream-load | NO | TRUE | データをロードする際に[Stream Loadトランザクションインターフェース](../loading/Stream_Load_transaction_interface.md)を使用するかどうか。StarRocks v2.5以降が必要です。この機能を使用すると、トランザクションごとにより少ないメモリ使用量でより多くのデータをロードし、パフォーマンスを向上させることができます。<br/> **注意:** 1.1.1以降、このパラメータは、`starrocks.write.max.retries`の値が非正の場合のみ有効です。なぜなら、Stream Loadトランザクションインターフェースはリトライをサポートしていないからです。 |
| starrocks.write.buffer.size                    | NO       | 104857600     | 一度にStarRocksに送信される前にメモリに蓄積できるデータの最大サイズです。このパラメータをより大きな値に設定すると、ロードのパフォーマンスが向上する可能性がありますが、ロードの遅延が増加する可能性があります。 |
| starrocks.write.buffer.rows | NO | Integer.MAX_VALUE | メモリに蓄積できるデータの最大行数です。バージョン1.1.1以降でサポートされています。 |
| starrocks.write.flush.interval.ms              | NO       | 300000        | データをStarRocksに送信する間隔です。このパラメータは、ロードの遅延を制御するために使用されます。 |
| starrocks.write.max.retries                    | NO       | 3             | リトライを行う場合の同じデータのバッチのStream Loadを実行するコネクタのリトライ回数です。<br/> **注意:** 1.1.1以降、このパラメータが正の値になっている場合、コネクタは常にStream Loadインタフェースを使用し、`starrocks.write.enable.transaction-stream-load`の値を無視します。 |
| starrocks.write.retry.interval.ms              | NO       | 10000         | 同じデータのバッチのロードが失敗した場合に、Stream Loadをリトライする間隔です。 |
| starrocks.columns                              | NO       | None          | データをロードしたいStarRocksテーブルのカラムです。たとえば、`"col0,col1,col2"`のように複数のカラムを指定できます。 |
### starrocks.column.types                         
- 必須: いいえ
- デフォルト: なし
- サポートされているバージョン: 1.1.1以降
- StarRocksテーブルと[デフォルトのマッピング](#data-type-mapping-between-spark-and-starrocks)から推論されたものではなく、Sparkのカラムデータ型をカスタマイズします。パラメータの値は、Sparkの[StructType#toDDL](https://github.com/apache/spark/blob/master/sql/api/src/main/scala/org/apache/spark/sql/types/StructType.scala#L449)の出力と同じDDL形式のスキーマです。例: `col0 INT, col1 STRING, col2 BIGINT`。カスタマイズが必要なカラムのみを指定する必要があります。[BITMAP](#load-data-into-columns-of-bitmap-type)型または[HLL](#load-data-into-columns-of-hll-type)型のカラムにデータをロードする場合など、1つのユースケースです。

### starrocks.write.properties.*
- 必須: いいえ
- デフォルト: なし
- ストリームロードの動作を制御するために使用されるパラメータです。例: パラメータ `starrocks.write.properties.format`は、CSVまたはJSONなどのデータのフォーマットを指定します。サポートされているパラメータとその説明の一覧については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。

### starrocks.write.properties.format
- 必須: いいえ
- デフォルト: CSV
- SparkコネクタがデータをStarRocksに送信する前に各データのバッチを変換するファイル形式です。有効な値は、CSVとJSONです。

### starrocks.write.properties.row_delimiter
- 必須: いいえ
- デフォルト: \n
- CSV形式のデータの行区切り記号です。

### starrocks.write.properties.column_separator
- 必須: いいえ
- デフォルト: \t
- CSV形式のデータの列区切り記号です。

### starrocks.write.num.partitions
- 必須: いいえ
- デフォルト: なし
- Sparkがデータを並列で書き込むためのパーティションの数です。データ量が少ない場合、パーティションの数を減らして、ロードの並行性と頻度を下げることができます。このパラメータのデフォルト値はSparkによって決定されますが、この方法はSpark Shuffleコストを引き起こす可能性があります。

### starrocks.write.partition.columns
- 必須: いいえ
- デフォルト: なし
- Sparkのパーティション列です。`starrocks.write.num.partitions`が指定されている場合にのみ、このパラメータが有効になります。このパラメータが指定されていない場合、書き込まれているすべての列がパーティション分割に使用されます。

### starrocks.timezone
- 必須: いいえ
- デフォルト: JVMのデフォルトタイムゾーン
- サポートされているバージョン: 1.1.1以降
- Sparkの`TimestampType`をStarRocksの`DATETIME`に変換するために使用されるタイムゾーンです。デフォルトは`ZoneId#systemDefault()`によって返されるJVMのタイムゾーンです。フォーマットは`Asia/Shanghai`などのタイムゾーン名、または`+08:00`などのゾーンオフセットが使用できます。
    | id   | name      | score |
    +------+-----------+-------+
    |    1 | starrocks |   100 |
    |    2 | spark     |   100 |
    +------+-----------+-------+
    2 行がセットされました (0.00 秒)

#### 構造化ストリーミング

CSV ファイルからデータをストリーミング読み込んで、StarRocks テーブルにデータをロードしてください。

1. `csv-data` ディレクトリ内で、以下のデータを含む CSV ファイル `test.csv` を作成します:

    ```csv
    3,starrocks,100
    4,spark,100
    ```

2. Scala または Python を使用して Spark アプリケーションを記述できます。

  Scala を使用する場合、`spark-shell` で次のコードを実行します:

    ```Scala
    import org.apache.spark.sql.types.StructType

    // 1. CSV から DataFrame を作成します。
    val schema = (new StructType()
            .add("id", "integer")
            .add("name", "string")
            .add("score", "integer")
        )
    val df = (spark.readStream
            .option("sep", ",")
            .schema(schema)
            .format("csv") 
            // "csv-data" ディレクトリへのパスに置き換えてください。
            .load("/path/to/csv-data")
        )
    
    // 2. フォーマットを "starrocks" に設定し、以下のオプションを使用して StarRocks に書き込みます。
    // 環境に応じてオプションを変更する必要があります。
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

  Python を使用する場合、`pyspark` で次のコードを実行します:
   
   ```python
   from pyspark.sql import SparkSession
   from pyspark.sql.types import IntegerType, StringType, StructType, StructField
   
   spark = SparkSession \
        .builder \
        .appName("StarRocks SS Example") \
        .getOrCreate()
   
    # 1. CSV から DataFrame を作成します。
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
        // "csv-data" ディレクトリへのパスに置き換えてください
        .load("/path/to/csv-data")
    )

    // "starrocks" と以下のオプションでフォーマットを設定し、StarRocks に書き込みます。
    // 環境に応じてオプションを変更する必要があります。
    query = (
        df.writeStream.format("starrocks")
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

3. StarRocks テーブル内のデータをクエリします。

    ```SQL
    MySQL [test]> select * from score_board;
    +------+-----------+-------+
    | id   | name      | score |
    +------+-----------+-------+
    |    4 | spark     |   100 |
    |    3 | starrocks |   100 |
    +------+-----------+-------+
    2 行がセットされました (0.67 秒)
    ```

### Spark SQL でデータを読み込む

次の例では、[Spark SQL CLI](https://spark.apache.org/docs/latest/sql-distributed-sql-engine-spark-sql-cli.html) の `INSERT INTO` ステートメントを使用して Spark SQL でデータを読み込む方法について説明します。

1. `spark-sql` で次の SQL ステートメントを実行します:

    ```SQL
    -- 1. データソースを `starrocks` に設定し、以下のオプションを使用してテーブルを作成します。
    // 環境に応じてオプションを変更する必要があります。
    CREATE TABLE `score_board`
    USING starrocks
    OPTIONS(
    "starrocks.fe.http.url"="127.0.0.1:8030",
    "starrocks.fe.jdbc.url"="jdbc:mysql://127.0.0.1:9030",
    "starrocks.table.identifier"="test.score_board",
    "starrocks.user"="root",
    "starrocks.password"=""
    );

    -- 2. テーブルに 2 つの行を挿入します。
    INSERT INTO `score_board` VALUES (5, "starrocks", 100), (6, "spark", 100);
    ```

2. StarRocks テーブル内のデータをクエリします。

    ```SQL
    MySQL [test]> select * from score_board;
    +------+-----------+-------+
    | id   | name      | score |
    +------+-----------+-------+
    |    6 | spark     |   100 |
    |    5 | starrocks |   100 |
    +------+-----------+-------+
    2 行がセットされました (0.00 秒)
    ```

## ベストプラクティス

### 主キーテーブルにデータを読み込む

このセクションでは、部分更新や条件付き更新を実行するために、StarRocks の主キーテーブルにデータを読み込む方法を示します。
これらの機能の詳細については、[loading](../loading/Load_to_Primary_Key_tables.md) を参照してください。
これらの例では Spark SQL が使用されます。

#### 準備

StarRocks でデータベース `test` を作成し、主キーテーブル `score_board` を作成します。

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

この例では、`name` 列のデータのみを更新する方法を示します:

1. MySQL クライアントで、初期データを StarRocks テーブルに挿入します。

   ```sql
   mysql> INSERT INTO `score_board` VALUES (1, 'starrocks', 100), (2, 'spark', 100);

   mysql> select * from score_board;
   +------+-----------+-------+
   | id   | name      | score |
   +------+-----------+-------+
   |    1 | starrocks |   100 |
   |    2 | spark     |   100 |
   +------+-----------+-------+
   2 行がセットされました (0.02 秒)
   ```

2. Spark SQL クライアントで、Spark テーブル `score_board` を作成します。

   - `starrocks.write.properties.partial_update` オプションを `true` に設定し、部分更新を行うようにコネクタに通知します。
   - `starrocks.columns` オプションを `"id,name"` に設定し、書き込む列をコネクタに通知します。

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

3. Spark SQL クライアントでテーブルにデータを挿入し、`name` 列のみを更新します。

   ```SQL
   INSERT INTO `score_board` VALUES (1, 'starrocks-update'), (2, 'spark-update');
   ```

4. MySQL クライアントで StarRocks テーブルをクエリします。

   `name` のみが変更され、`score` の値は変更されないことがわかります。

   ```SQL
   mysql> select * from score_board;
   +------+------------------+-------+
   | id   | name             | score |
   +------+------------------+-------+
   |    1 | starrocks-update |   100 |
   |    2 | spark-update     |   100 |
   +------+------------------+-------+
   2 行がセットされました (0.02 秒)
   ```

#### 条件付き更新

この例では、`score` 列の値に応じて条件付き更新する方法を示します。`id` の更新は、新しい `score` の値が古い値以上の場合のみ有効です。
1. StarRocksテーブルに初期データをMySQLクライアントで挿入します。

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

2. 以下の方法でSparkテーブル `score_board` を作成します。

   - オプション `starrocks.write.properties.merge_condition` を `score` に設定して、コネクタに列 `score` を条件として使用するよう指示します。
   - Sparkコネクタがこの機能をサポートしていないため、ストリームロードトランザクションインターフェースではなく、ストリームロードインターフェースを使用することを確認してください。

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

   `id`が2の行だけが変更され、`id`が1の行は変更されていないことがわかります。

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

[`BITMAP`](../sql-reference/sql-statements/data-types/BITMAP.md)は、UVのカウントなどの高速なカウントを促進するためによく使用されます。詳細については、[Use Bitmap for exact Count Distinct](../using_starrocks/Using_bitmap.md)を参照してください。
ここでは、`BITMAP`タイプの列にデータをロードする方法を示す例として、UVのカウントを取ることを挙げます。 **`BITMAP` はバージョン1.1.1からサポートされています**。

1. StarRocks集計テーブルを作成します。

   データベース`test`で、`visit_users`が`BITMAP_UNION`として設定された`BITMAP`タイプで定義された`page_uv`という集計テーブルを作成します。

    ```SQL
    CREATE TABLE `test`.`page_uv` (
      `page_id` INT NOT NULL COMMENT 'page ID',
      `visit_date` datetime NOT NULL COMMENT 'access time',
      `visit_users` BITMAP BITMAP_UNION NOT NULL COMMENT 'user ID'
    ) ENGINE=OLAP
    AGGREGATE KEY(`page_id`, `visit_date`)
    DISTRIBUTED BY HASH(`page_id`);
    ```

2. Sparkテーブルを作成します。

   SparkテーブルのスキーマはStarRocksテーブルから推論されますが、Sparkは`BITMAP`タイプをサポートしていません。そのため、Sparkで対応する列のデータ型をカスタマイズする必要があります。たとえば、オプション`"starrocks.column.types"="visit_users BIGINT"`を構成して、`BIGINT`としてカスタマイズできます。データをストリームロードで取り込む場合、コネクタは[`to_bitmap`](../sql-reference/sql-functions/bitmap-functions/to_bitmap.md)関数を使用して`BIGINT`タイプのデータを`BITMAP`タイプに変換します。

    `spark-sql`で次のDDLを実行します。

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

    `spark-sql`で次のDMLを実行します。

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
> コネクタは、Sparkの`TINYINT`、`SMALLINT`、`INTEGER`、`BIGINT`タイプのデータをStarRocksの`BITMAP`タイプに変換するために[`to_bitmap`](../sql-reference/sql-functions/bitmap-functions/to_bitmap.md)関数を使用し、他のSparkデータ型には[`bitmap_hash`](../sql-reference/sql-functions/bitmap-functions/bitmap_hash.md)関数を使用します。

### HLL型の列にデータをロードする

[`HLL`](../sql-reference/sql-statements/data-types/HLL.md)は、近似カウントの場合に使用できます。詳細については、[Use HLL for approximate count distinct](../using_starrocks/Using_HLL.md)を参照してください。

ここでは、UVのカウントを例にして、`HLL`タイプの列にデータをロードする方法を示します。 **`HLL` はバージョン1.1.1からサポートされています**。

1. StarRocks集計テーブルを作成します。

   データベース`test`で、`visit_users`が`HLL_UNION`として設定された`HLL`タイプで定義された`hll_uv`という集計テーブルを作成します。

    ```SQL
    CREATE TABLE `hll_uv` (
    `page_id` INT NOT NULL COMMENT 'page ID',
    `visit_date` datetime NOT NULL COMMENT 'access time',
    `visit_users` HLL HLL_UNION NOT NULL COMMENT 'user ID'
    ) ENGINE=OLAP
    AGGREGATE KEY(`page_id`, `visit_date`)
    DISTRIBUTED BY HASH(`page_id`);
    ```

2. Sparkテーブルを作成します。

   SparkテーブルのスキーマはStarRocksテーブルから推論されますが、Sparkは`HLL`タイプをサポートしていません。そのため、Sparkで対応する列のデータ型をカスタマイズする必要があります。たとえば、オプション`"starrocks.column.types"="visit_users BIGINT"`を構成して、`BIGINT`としてカスタマイズできます。データをストリームロードで取り込む場合、コネクタは[`hll_hash`](../sql-reference/sql-functions/aggregate-functions/hll_hash.md)関数を使用して`BIGINT`タイプのデータを`HLL`タイプに変換します。

    `spark-sql`で次のDDLを実行します。

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

    `spark-sql`で次のDMLを実行します。

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
    2 rows in set (0.01 sec)
    ```

### ARRAY型の列にデータをロードする

次の例は、[`ARRAY`](../sql-reference/sql-statements/data-types/Array.md)型の列にデータをロードする方法を説明しています。

1. StarRocksテーブルを作成します。

   データベース`test`で、`INT`型の列1つと`ARRAY`型の列2つを含むPrimary Keyテーブル`array_tbl`を作成します。

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

   StarRocksのバージョンによっては、`ARRAY`列のメタデータが提供されないことがありますので、コネクタはこの列の対応するSparkデータ型を推論することができません。ただし、オプション`starrocks.column.types`で列の対応するSparkデータ型を明示的に指定することができます。この例では、オプションを`a0 ARRAY<STRING>,a1 ARRAY<ARRAY<INT>>`として設定できます。

   以下のコードを`spark-shell`で実行します。

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

3. StarRocksテーブル内のデータをクエリします。

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