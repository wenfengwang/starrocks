---
displayed_sidebar: "Japanese"
---

# Flinkコネクタを使用してStarRocksからデータを読み取る

StarRocksは、StarRocksコネクタ（略してFlinkコネクタ）と呼ばれる自社開発のApache Flink®コネクタを提供し、Flinkを使用してStarRocksクラスタから大量のデータを読み取るのを支援します。

Flinkコネクタは、Flink SQLとFlink DataStreamの2つの読み取り方法をサポートしています。Flink SQLが推奨されています。

> **注意**
>
> Flinkコネクタは、Flinkによって読み取られたデータを別のStarRocksクラスタまたはストレージシステムに書き込むこともサポートしています。[Apache Flink®からデータを継続的に読み込む](../loading/Flink-connector-starrocks.md)を参照してください。

## 背景情報

Flinkが提供するJDBCコネクタとは異なり、StarRocksのFlinkコネクタはStarRocksクラスタの複数のBEから並列にデータを読み取ることをサポートしており、読み取りタスクを大幅に加速します。以下の比較は、2つのコネクタの実装の違いを示しています。

- StarRocksのFlinkコネクタ

  StarRocksのFlinkコネクタでは、Flinkはまず責任を持つFEからクエリプランを取得し、次に取得したクエリプランをすべての関連するBEにパラメータとして配布し、最後にBEから返されたデータを取得します。

  ![- StarRocksのFlinkコネクタ](../assets/5.3.2-1.png)

- FlinkのJDBCコネクタ

  FlinkのJDBCコネクタでは、Flinkは1度に1つのFEからのみデータを読み取ることができます。データ読み取りが遅いです。

  ![FlinkのJDBCコネクタ](../assets/5.3.2-2.png)

## バージョン要件

| コネクタ | Flink                    | StarRocks     | Java | Scala     |
|-----------|--------------------------|---------------| ---- |-----------|
| 1.2.8     | 1.13,1.14,1.15,1.16,1.17 | 2.1 以降      | 8    | 2.11,2.12 |
| 1.2.7     | 1.11,1.12,1.13,1.14,1.15 | 2.1 以降      | 8    | 2.11,2.12 |

## 前提条件

Flinkがデプロイされている必要があります。Flinkがデプロイされていない場合は、次の手順に従ってデプロイしてください。

1. Flinkが正常に実行できるように、オペレーティングシステムにJava 8またはJava 11をインストールしてください。以下のコマンドを使用して、Javaのインストールバージョンを確認できます。

   ```SQL
   java -version
   ```

   たとえば、次の情報が返された場合、Java 8がインストールされています。

   ```SQL
   openjdk version "1.8.0_322"
   OpenJDK Runtime Environment (Temurin)(build 1.8.0_322-b06)
   OpenJDK 64-Bit Server VM (Temurin)(build 25.322-b06, mixed mode)
   ```

2. [Flinkパッケージ](https://flink.apache.org/downloads.html)を選択してダウンロードし、展開してください。

   > **注意**
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
         
   # 以下の情報が表示された場合、Flinkクラスタが正常に起動しています:
   Starting cluster.
   Starting standalonesession daemon on host.
   Starting taskexecutor daemon on host.
   ```

[Flinkドキュメント](https://nightlies.apache.org/flink/flink-docs-release-1.13/docs/try-flink/local_installation/)に記載された手順に従っても、Flinkをデプロイできます。

## 開始する前に

Flinkコネクタをデプロイするには、次の手順に従ってください。

1. 使用しているFlinkバージョンに一致する[flink-connector-starrocks](https://github.com/StarRocks/flink-connector-starrocks/releases) JARパッケージを選択してダウンロードしてください。

   > **注意**
   >
   > 使用するFlinkバージョンと同じ最初の2桁を持つFlinkバージョンの1.2.x以降のFlinkコネクタパッケージをダウンロードすることをお勧めします。たとえば、Flink v1.14.xを使用する場合は、`flink-connector-starrocks-1.2.4_flink-1.14_x.yy.jar`をダウンロードできます。

2. コードのデバッグが必要な場合は、Flinkコネクタパッケージをビジネス要件に合わせてコンパイルしてください。

3. ダウンロードまたはコンパイルしたFlinkコネクタパッケージを、Flinkの`lib`ディレクトリに配置してください。

4. Flinkクラスタを再起動します。

## パラメータ

### 共通パラメータ

次のパラメータは、Flink SQLとFlink DataStreamの両方の読み取り方法に適用されます。

| パラメータ                   | 必須     | データ型 | 説明                                        |
| --------------------------- | -------- | --------- | -------------------------------------------- |
| connector                   | Yes      | STRING    | データを読み取るために使用するコネクタの種類を設定します。値を`starrocks`に設定します。           |
| scan-url                    | Yes      | STRING    | WebサーバからFEに接続するためのアドレスです。形式: `<fe_host>:<fe_http_port>`。デフォルトポートは `8030` です。複数のアドレスを指定できますが、コンマ (,) で区切る必要があります。例: `192.168.xxx.xxx:8030,192.168.xxx.xxx:8030`。 |
| jdbc-url                    | Yes      | STRING    | FEのMySQLクライアントに接続するためのアドレスです。形式: `jdbc:mysql://<fe_host>:<fe_query_port>`。デフォルトポート番号は `9030` です。 |
| username                    | Yes      | STRING    | StarRocksクラスタアカウントのユーザー名です。アカウントは、読み取りたいStarRocksテーブルに対する読み取り権限を持っている必要があります。[ユーザ権限](../administration/User_privilege.md)を参照してください。 |
| password                    | Yes      | STRING    | StarRocksクラスタアカウントのパスワードです。   |
| database-name               | Yes      | STRING    | 読み取りたいStarRocksテーブルが属するStarRocksデータベースの名前です。 |
| table-name                  | Yes      | STRING    | 読み取りたいStarRocksテーブルの名前です。      |
| scan.connect.timeout-ms     | No       | STRING    | FlinkコネクタからStarRocksクラスタへの接続がタイムアウトするまでの最大時間です。単位: ミリ秒。デフォルト値: `1000`。接続を確立するのにかかる時間がこの制限を超えると、読み取りタスクが失敗します。 |
| scan.params.keep-alive-min  | No       | STRING    | 読み取りタスクを維持する最大時間です。ポーリングメカニズムを使用して定期的にKeep-Alive時間がチェックされます。単位: 分。デフォルト値: `10`。このパラメータは、`5`以上の値に設定することをお勧めします。 |
| scan.params.query-timeout-s | No       | STRING    | 読み取りタスクがタイムアウトするまでの最大時間です。タスクの実行中にタイムアウト期間がチェックされます。単位: 秒。デフォルト値: `600`。指定した時間後に読み取り結果が返されない場合、読み取りタスクが中止されます。 |
| scan.params.mem-limit-byte  | No       | STRING    | 各BEのクエリごとに許可される最大メモリ量です。単位: バイト。デフォルト値: `1073741824`、つまり 1 GB です。 |
| scan.max-retries            | No       | STRING    | 読み取りタスクが失敗した場合に再試行できる最大回数です。デフォルト値: `1`。再試行回数がこの制限を超えると、読み取りタスクはエラーを返します。 |

### Flink DataStream用パラメータ

次のパラメータは、Flink DataStreamの読み取り方法にのみ適用されます。

| パラメータ    | 必須     | データ型 | 説明                                        |
| ------------ | -------- | --------- | -------------------------------------------- |
| scan.columns | No       | STRING    | 読み取りたい列です。複数の列を指定できますが、コンマ (,) で区切る必要があります。 |
| scan.filter  | No       | STRING    | データをフィルタリングしたいフィルタ条件です。 |

Flinkで`c1`、`c2`、`c3`の3列からなるテーブルを作成したとします。このFlinkテーブルの`c1`列の値が`100`に等しい行を読み取る場合は、2つのフィルタ条件 `"scan.columns, "c1"` と `"scan.filter, "c1 = 100"` を指定できます。

## StarRocksとFlink間のデータ型のマッピング
Flink でデータを StarRocks から読み取るためのデータ型マッピングは、以下の通りです。Flink から StarRocks へのデータ書き込みに使用されるデータ型マッピングについては、[Apache Flink® からデータを連続的にロードする](../loading/Flink-connector-starrocks.md) をご覧ください。

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

以下の例では、StarRocks クラスタに `test` という名前のデータベースが作成されており、ユーザー `root` に対する権限があると仮定しています。

> **注記**
>
> 読み取りタスクが失敗した場合、再作成する必要があります。

### データ例

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
   PROPERTIES
   (
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
   21 rows in set (0.00 sec)
   ```

### Flink SQL を使用してデータを読み取る

1. Flink クラスタで、ソース StarRocks テーブルのスキーマ（この例では `score_board`）に基づいて `flink_test` という名前のテーブルを作成します。テーブル作成コマンドでは、Flink コネクタ、ソース StarRocks データベース、およびソース StarRocks テーブルの情報を含む読み取りタスクのプロパティを構成する必要があります。

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

2. SELECT を使用して StarRocks からデータを読み取ります。

   ```SQL
   SELECT id, name FROM flink_test WHERE score > 20;
   ```

Flink SQL を使用してデータを読み取る際には、次の点に注意してください。

- StarRocks からデータを読み取るには、`SELECT ... FROM <table_name> WHERE ...` のような SQL ステートメントのみを使用できます。集約関数のうち、`count` のみがサポートされています。
- プレディケートのプッシュダウンがサポートされています。たとえば、クエリにフィルター条件 `char_1 <> 'A' and int_1 = -126` が含まれている場合、フィルター条件は Flink コネクタにプッシュダウンされ、クエリ実行の前に StarRocks で実行可能なステートメントに変換されます。追加の設定を行う必要はありません。
- LIMIT ステートメントはサポートされていません。
- StarRocks はチェックポイント機構をサポートしていません。そのため、読み取りタスクに失敗した場合、データの整合性を保証することはできません。

### Flink DataStream を使用してデータを読み取る

1. `pom.xml` ファイルに以下の依存関係を追加します。

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

   以下のコード例で、前述のコード例内の `x.x.x` を使用している最新の Flink コネクタ・バージョンに置き換える必要があります。[バージョン情報](https://search.maven.org/search?q=g:com.starrocks) をご参照ください。

2. Flink コネクタを呼び出し、StarRocks からデータを読み取ります。

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
## 次は何ですか

StarRocksからデータを正常に読み込んだ後、[Flink WebUI](https://nightlies.apache.org/flink/flink-docs-master/docs/try-flink/flink-operations-playground/#flink-webui)を使用して読み込みタスクをモニタリングできます。たとえば、WebUIの**Metrics**ページで`totalScannedRows`メトリックスを表示して、正常に読み込まれた行数を取得できます。また、読み込んだデータを使用してFlink SQLを使用して結合などの計算を行うこともできます。