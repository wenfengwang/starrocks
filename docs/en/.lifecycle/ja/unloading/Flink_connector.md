---
displayed_sidebar: "Japanese"
---

# Flinkコネクタを使用してStarRocksからデータを読み取る

StarRocksは、Apache Flink®（以下、Flinkコネクタ）用のStarRocksコネクタという自社開発のコネクタを提供しており、Flinkを使用してStarRocksクラスタから大量のデータを一括で読み取ることができます。

Flinkコネクタは、Flink SQLとFlink DataStreamの2つの読み取り方法をサポートしています。Flink SQLが推奨されています。

> **注記**
>
> Flinkコネクタは、Flinkによって読み取られたデータを別のStarRocksクラスタまたはストレージシステムに書き込むこともサポートしています。[Apache Flink®からデータを連続的に読み込む](../loading/Flink-connector-starrocks.md)を参照してください。

## 背景情報

Flinkが提供するJDBCコネクタとは異なり、StarRocksのFlinkコネクタは、StarRocksクラスタの複数のBEからデータを並列に読み取ることができ、読み取りタスクを大幅に高速化することができます。以下の比較では、2つのコネクタの実装の違いを示しています。

- StarRocksのFlinkコネクタ

  StarRocksのFlinkコネクタでは、Flinkはまず責任を持つFEからクエリプランを取得し、取得したクエリプランをすべての関連するBEにパラメータとして配布し、最後にBEから返されたデータを取得します。

  ![- StarRocksのFlinkコネクタ](../assets/5.3.2-1.png)

- FlinkのJDBCコネクタ

  FlinkのJDBCコネクタでは、Flinkは個々のFEからデータを1つずつ読み取ることしかできません。データの読み取りは遅いです。

  ![FlinkのJDBCコネクタ](../assets/5.3.2-2.png)

## バージョン要件

| コネクタ | Flink                    | StarRocks     | Java | Scala     |
|-----------|--------------------------|---------------| ---- |-----------|
| 1.2.8     | 1.13,1.14,1.15,1.16,1.17 | 2.1以降       | 8    | 2.11,2.12 |
| 1.2.7     | 1.11,1.12,1.13,1.14,1.15 | 2.1以降       | 8    | 2.11,2.12 |

## 前提条件

Flinkがデプロイされていることを前提とします。Flinkがデプロイされていない場合は、次の手順に従ってデプロイしてください。

1. オペレーティングシステムにJava 8またはJava 11をインストールして、Flinkが正常に実行できることを確認します。次のコマンドを使用して、Javaのインストールバージョンを確認できます。

   ```SQL
   java -version
   ```

   たとえば、次の情報が返された場合、Java 8がインストールされています。

   ```SQL
   openjdk version "1.8.0_322"
   OpenJDK Runtime Environment (Temurin)(build 1.8.0_322-b06)
   OpenJDK 64-Bit Server VM (Temurin)(build 25.322-b06, mixed mode)
   ```

2. [Flinkパッケージ](https://flink.apache.org/downloads.html)をダウンロードして展開します。

   > **注記**
   >
   > Flink v1.14以降を使用することをお勧めします。サポートされる最小のFlinkバージョンはv1.11です。

   ```SQL
   # Flinkパッケージをダウンロードします。
   wget https://dlcdn.apache.org/flink/flink-1.14.5/flink-1.14.5-bin-scala_2.11.tgz
   # Flinkパッケージを展開します。
   tar -xzf flink-1.14.5-bin-scala_2.11.tgz
   # Flinkディレクトリに移動します。
   cd flink-1.14.5
   ```

3. Flinkクラスタを起動します。

   ```SQL
   # Flinkクラスタを起動します。
   ./bin/start-cluster.sh
         
   # 次の情報が表示された場合、Flinkクラスタが正常に起動しています。
   Starting cluster.
   Starting standalonesession daemon on host.
   Starting taskexecutor daemon on host.
   ```

[Flinkドキュメント](https://nightlies.apache.org/flink/flink-docs-release-1.13/docs/try-flink/local_installation/)で提供されている手順に従ってFlinkをデプロイすることもできます。

## 開始する前に

Flinkコネクタをデプロイするために、次の手順に従ってください。

1. 使用しているFlinkバージョンに一致する[flink-connector-starrocks](https://github.com/StarRocks/flink-connector-starrocks/releases)のJARパッケージを選択してダウンロードします。

   > **注意**
   >
   > Flinkのバージョンが1.2.x以上で、一致するFlinkバージョンの最初の2桁が使用しているFlinkバージョンと同じであるFlinkコネクタパッケージをダウンロードすることをお勧めします。たとえば、Flink v1.14.xを使用している場合、`flink-connector-starrocks-1.2.4_flink-1.14_x.yy.jar`をダウンロードできます。

2. コードのデバッグが必要な場合は、Flinkコネクタパッケージをビジネス要件に合わせてコンパイルします。

3. ダウンロードまたはコンパイルしたFlinkコネクタパッケージをFlinkの`lib`ディレクトリに配置します。

4. Flinkクラスタを再起動します。

## パラメータ

### 共通パラメータ

次のパラメータは、Flink SQLとFlink DataStreamの両方の読み取り方法に適用されます。

| パラメータ                   | 必須     | データ型 | 説明                                                         |
| --------------------------- | -------- | --------- | ------------------------------------------------------------ |
| connector                   | Yes      | STRING    | データを読み取るために使用するコネクタのタイプを設定します。値を`starrocks`に設定します。                                |
| scan-url                    | Yes      | STRING    | WebサーバーからFEに接続するために使用されるアドレスです。フォーマット：`<fe_host>:<fe_http_port>`。デフォルトのポートは`8030`です。複数のアドレスを指定することもできますが、カンマ（,）で区切る必要があります。例：`192.168.xxx.xxx:8030,192.168.xxx.xxx:8030`。 |
| jdbc-url                    | Yes      | STRING    | FEのMySQLクライアントに接続するために使用されるアドレスです。フォーマット：`jdbc:mysql://<fe_host>:<fe_query_port>`。デフォルトのポート番号は`9030`です。 |
| username                    | Yes      | STRING    | StarRocksクラスタのアカウントのユーザー名です。アカウントは、読み取り対象のStarRocksテーブルに対して読み取り権限を持っている必要があります。[ユーザー権限](../administration/User_privilege.md)を参照してください。 |
| password                    | Yes      | STRING    | StarRocksクラスタのアカウントのパスワードです。              |
| database-name               | Yes      | STRING    | 読み取り対象のStarRocksテーブルが所属するStarRocksデータベースの名前です。 |
| table-name                  | Yes      | STRING    | 読み取り対象のStarRocksテーブルの名前です。                  |
| scan.connect.timeout-ms     | No       | STRING    | FlinkコネクタからStarRocksクラスタへの接続がタイムアウトするまでの最大時間です。単位：ミリ秒。デフォルト値：`1000`。接続の確立にかかる時間がこの制限を超えると、読み取りタスクは失敗します。 |
| scan.params.keep-alive-min  | No       | STRING    | 読み取りタスクがアクティブな状態を維持する最大時間です。アクティブな状態は、ポーリングメカニズムを使用して定期的にチェックされます。単位：分。デフォルト値：`10`。このパラメータを`5`以上の値に設定することをお勧めします。 |
| scan.params.query-timeout-s | No       | STRING    | 読み取りタスクがタイムアウトするまでの最大時間です。タイムアウトの期間は、タスクの実行中にチェックされます。単位：秒。デフォルト値：`600`。指定された時間の経過後に読み取り結果が返されない場合、読み取りタスクは停止します。 |
| scan.params.mem-limit-byte  | No       | STRING    | 各BEのクエリごとに許可されるメモリの最大量です。単位：バイト。デフォルト値：`1073741824`（1GBと等しい）。 |
| scan.max-retries            | No       | STRING    | 読み取りタスクが失敗した場合に再試行できる最大回数です。デフォルト値：`1`。読み取りタスクの再試行回数がこの制限を超えると、読み取りタスクはエラーを返します。 |

### Flink DataStream用のパラメータ

次のパラメータは、Flink DataStreamの読み取り方法にのみ適用されます。

| パラメータ    | 必須     | データ型 | 説明                                                         |
| ------------ | -------- | --------- | ------------------------------------------------------------ |
| scan.columns | No       | STRING    | 読み取りたい列を指定します。複数の列を指定することができますが、カンマ（,）で区切る必要があります。 |
| scan.filter  | No       | STRING    | データをフィルタリングするためのフィルタ条件です。 |

Flinkで3つの列（`c1`、`c2`、`c3`）からなるテーブルを作成した場合、このFlinkテーブルの`c1`列の値が`100`と等しい行を読み取るために、`"scan.columns, "c1"`と`"scan.filter, "c1 = 100"`という2つのフィルタ条件を指定できます。

## StarRocksとFlinkのデータ型マッピング

次のデータ型マッピングは、FlinkがStarRocksからデータを読み取る場合にのみ有効です。FlinkからStarRocksにデータを書き込むためのデータ型マッピングについては、[Apache Flink®からデータを連続的に読み込む](../loading/Flink-connector-starrocks.md)を参照してください。

| StarRocks  | Flink     |
| ---------- | --------- |
| NULL       | NULL      |
| BOOLEAN    | BOOLEAN   |
| TINYINT    | TINYINT   |
| SMALLINT   | SMALLINT  |
| INT        | INT       |
| BIGINT     | BIGINT    |
| LARGEINT   | STRING    |
| FLOAT      | FLOAT     |
| DOUBLE     | DOUBLE    |
| DATE       | DATE      |
| DATETIME   | TIMESTAMP |
| DECIMAL    | DECIMAL   |
| DECIMALV2  | DECIMAL   |
| DECIMAL32  | DECIMAL   |
| DECIMAL64  | DECIMAL   |
| DECIMAL128 | DECIMAL   |
| CHAR       | CHAR      |
| VARCHAR    | STRING    |

## 例

以下の例では、StarRocksクラスタに`test`という名前のデータベースを作成し、ユーザー`root`の権限を持っていることを前提としています。

> **注記**
>
> 読み取りタスクが失敗した場合、再作成する必要があります。

### データの例

1. `test`データベースに`score_board`という名前のテーブルを作成します。

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
   PROPERTIES
   (
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
   21 rows in set (0.00 sec)
   ```

### Flink SQLを使用してデータを読み取る

1. Flinkクラスタで、ソースとなるStarRocksテーブルのスキーマに基づいて`flink_test`という名前のテーブルを作成します。テーブル作成コマンドでは、Flinkコネクタ、ソースのStarRocksデータベース、ソースのStarRocksテーブルに関する読み取りタスクのプロパティを設定する必要があります。

   ```SQL
   CREATE TABLE flink_test
   (
       `id` INT,
       `name` STRING,
       `score` INT
   )
   WITH
   (
       'connector'='starrocks',
       'scan-url'='192.168.xxx.xxx:8030',
       'jdbc-url'='jdbc:mysql://192.168.xxx.xxx:9030',
       'username'='xxxxxx',
       'password'='xxxxxx',
       'database-name'='test',
       'table-name'='score_board'
   );
   ```

2. SELECTを使用してStarRocksからデータを読み取ります。

   ```SQL
   SELECT id, name FROM flink_test WHERE score > 20;
   ```

Flink SQLを使用してデータを読み取る場合は、次の点に注意してください。

- StarRocksからデータを読み取るためには、`SELECT ... FROM <table_name> WHERE ...`のようなSQLステートメントのみを使用できます。集計関数のうち、`count`のみがサポートされています。
- プレディケートプッシュダウンがサポートされています。たとえば、クエリにフィルタ条件`char_1 <> 'A' and int_1 = -126`が含まれている場合、フィルタ条件はFlinkコネクタにプッシュダウンされ、クエリが実行される前にStarRocksで実行可能なステートメントに変換されます。追加の設定は必要ありません。
- LIMITステートメントはサポートされていません。
- StarRocksはチェックポイントメカニズムをサポートしていません。そのため、読み取りタスクが失敗した場合、データの整合性を保証することはできません。

### Flink DataStreamを使用してデータを読み取る

1. `pom.xml`ファイルに次の依存関係を追加します。

   ```SQL
   <dependency>
       <groupId>com.starrocks</groupId>
       <artifactId>flink-connector-starrocks</artifactId>
       <!-- for Apache Flink® 1.15 -->
       <version>x.x.x_flink-1.15</version>
       <!-- for Apache Flink® 1.14 -->
       <version>x.x.x_flink-1.14_2.11</version>
       <version>x.x.x_flink-1.14_2.12</version>
       <!-- for Apache Flink® 1.13 -->
       <version>x.x.x_flink-1.13_2.11</version>
       <version>x.x.x_flink-1.13_2.12</version>
       <!-- for Apache Flink® 1.12 -->
       <version>x.x.x_flink-1.12_2.11</version>
       <version>x.x.x_flink-1.12_2.12</version>
       <!-- for Apache Flink® 1.11 -->
       <version>x.x.x_flink-1.11_2.11</version>
       <version>x.x.x_flink-1.11_2.12</version>
   </dependency>
   ```

   上記のコード例では、最新の使用しているFlinkコネクタバージョンである`x.x.x`を使用してください。[バージョン情報](https://search.maven.org/search?q=g:com.starrocks)を参照してください。

2. Flinkコネクタを呼び出してStarRocksからデータを読み取ります。

   ```Java
   import com.starrocks.connector.flink.StarRocksSource;
   import com.starrocks.connector.flink.table.source.StarRocksSourceOptions;
   import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
   import org.apache.flink.table.api.DataTypes;
   import org.apache.flink.table.api.TableSchema;
   
   public class StarRocksSourceApp {
           public static void main(String[] args) throws Exception {
               StarRocksSourceOptions options = StarRocksSourceOptions.builder()
                      .withProperty("scan-url", "192.168.xxx.xxx:8030")
                      .withProperty("jdbc-url", "jdbc:mysql://192.168.xxx.xxx:9030")
                      .withProperty("username", "root")
                      .withProperty("password", "")
                      .withProperty("table-name", "score_board")
                      .withProperty("database-name", "test")
                      .build();
               TableSchema tableSchema = TableSchema.builder()
                      .field("id", DataTypes.INT())
                      .field("name", DataTypes.STRING())
                      .field("score", DataTypes.INT())
                      .build();
               StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
               env.addSource(StarRocksSource.source(tableSchema, options)).setParallelism(5).print();
               env.execute("StarRocks flink source");
           }

       }
   ```

## 次のステップ

FlinkがStarRocksからデータを正常に読み取った後、[Flink WebUI](https://nightlies.apache.org/flink/flink-docs-master/docs/try-flink/flink-operations-playground/#flink-webui)を使用して読み取りタスクを監視できます。たとえば、WebUIの**Metrics**ページで`totalScannedRows`メトリックを表示して、正常に読み取られた行数を取得することができます。また、読み取ったデータに対して結合などの計算をFlink SQLを使用して実行することもできます。
