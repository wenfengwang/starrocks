---
displayed_sidebar: English
---

# Flinkコネクタを使用してStarRocksからデータを読み取る

StarRocksは、Apache Flink®を使用してStarRocksクラスターから大量のデータを読み取るための自社開発のコネクタであるStarRocks Connector for Apache Flink®（以下、Flinkコネクタ）を提供しています。

Flinkコネクタは、Flink SQLとFlink DataStreamの2つの読み取り方法をサポートしています。Flink SQLが推奨されます。

> **注記**
>
> Flinkコネクタは、Flinkによって読み取られたデータを別のStarRocksクラスターやストレージシステムに書き込むこともサポートしています。詳細は[Apache Flink®からデータを継続的にロードする](../loading/Flink-connector-starrocks.md)を参照してください。

## 背景情報

Flinkが提供するJDBCコネクタとは異なり、StarRocksのFlinkコネクタは、StarRocksクラスターの複数のBEからデータを並行して読み取ることをサポートし、読み取りタスクの速度を大幅に向上させます。以下の比較は、2つのコネクタの実装の違いを示しています。

- StarRocksのFlinkコネクタ

  StarRocksのFlinkコネクタを使用すると、Flinkはまず責任FEからクエリプランを取得し、その後取得したクエリプランをパラメータとして関連するすべてのBEに配布し、最終的にBEから返されるデータを取得します。

  ![StarRocksのFlinkコネクタ](../assets/5.3.2-1.png)

- FlinkのJDBCコネクタ

  FlinkのJDBCコネクタを使用すると、Flinkは一度に1つのFEからしかデータを読み取ることができず、データの読み取りが遅くなります。

  ![FlinkのJDBCコネクタ](../assets/5.3.2-2.png)

## バージョン要件

| コネクタ | Flink                    | StarRocks     | Java | Scala     |
|-----------|--------------------------|---------------| ---- |-----------|
| 1.2.9     | 1.15,1.16,1.17,1.18      | 2.1以降       | 8    | 2.11,2.12 |
| 1.2.8     | 1.13,1.14,1.15,1.16,1.17 | 2.1以降       | 8    | 2.11,2.12 |
| 1.2.7     | 1.11,1.12,1.13,1.14,1.15 | 2.1以降       | 8    | 2.11,2.12 |

## 前提条件

Flinkがデプロイされています。Flinkがまだデプロイされていない場合は、以下の手順に従ってデプロイしてください：

1. オペレーティングシステムにJava 8またはJava 11をインストールして、Flinkが正常に実行されることを確認します。以下のコマンドを使用してJavaインストールのバージョンをチェックできます：

   ```SQL
   java -version
   ```

   例えば、以下の情報が返された場合、Java 8がインストールされています：

   ```SQL
   openjdk version "1.8.0_322"
   OpenJDK Runtime Environment (Temurin)(build 1.8.0_322-b06)
   OpenJDK 64-Bit Server VM (Temurin)(build 25.322-b06, mixed mode)
   ```

2. お好みの[Flinkパッケージをダウンロードして解凍](https://flink.apache.org/downloads.html)します。

   > **注記**
   >
   > Flink v1.14以降の使用を推奨します。サポートされている最小Flinkバージョンはv1.11です。

   ```SQL
   # Flinkパッケージをダウンロードします。
   wget https://dlcdn.apache.org/flink/flink-1.14.5/flink-1.14.5-bin-scala_2.11.tgz
   # Flinkパッケージを解凍します。
   tar -xzf flink-1.14.5-bin-scala_2.11.tgz
   # Flinkディレクトリに移動します。
   cd flink-1.14.5
   ```

3. Flinkクラスターを起動します。

   ```SQL
   # Flinkクラスターを起動します。
   ./bin/start-cluster.sh
         
   # 以下の情報が表示されたら、Flinkクラスターが正常に起動しています：
   Starting cluster.
   Starting standalonesession daemon on host.
   Starting taskexecutor daemon on host.
   ```

[Flinkのドキュメント](https://nightlies.apache.org/flink/flink-docs-release-1.13/docs/try-flink/local_installation/)に記載されている手順に従ってFlinkをデプロイすることもできます。

## 始める前に

以下の手順に従ってFlinkコネクタをデプロイしてください：

1. 使用しているFlinkバージョンに一致する[flink-connector-starrocks](https://github.com/StarRocks/flink-connector-starrocks/releases) JARパッケージを選択してダウンロードします。

   > **注意**
   >
   > バージョンが1.2.x以降で、一致するFlinkバージョンの最初の2桁が使用しているFlinkバージョンと同じであるFlinkコネクタパッケージをダウンロードすることを推奨します。例えば、Flink v1.14.xを使用している場合は、`flink-connector-starrocks-1.2.4_flink-1.14_x.yy.jar`をダウンロードできます。

2. コードデバッグが必要な場合は、ビジネス要件に合わせてFlinkコネクタパッケージをコンパイルします。

3. ダウンロードまたはコンパイルしたFlinkコネクタパッケージをFlinkの`lib`ディレクトリに配置します。

4. Flinkクラスターを再起動します。

## パラメータ

### 共通パラメータ

以下のパラメータは、Flink SQLとFlink DataStreamの読み取り方法の両方に適用されます。

| パラメータ                   | 必須 | データ型 | 説明                                                  |
| --------------------------- | -------- | --------- | ------------------------------------------------------------ |
| connector                   | はい      | STRING    | 使用するコネクタの種類。`starrocks`に設定します。                                |
| scan-url                    | はい      | STRING    | WebサーバーからFEに接続するために使用されるアドレス。形式: `<fe_host>:<fe_http_port>`。デフォルトポートは`8030`です。複数のアドレスを指定することができ、カンマ(,)で区切る必要があります。例: `192.168.xxx.xxx:8030,192.168.xxx.xxx:8030`。 |
| jdbc-url                    | はい      | STRING    | FEのMySQLクライアントに接続するために使用されるアドレス。形式: `jdbc:mysql://<fe_host>:<fe_query_port>`。デフォルトのポート番号は`9030`です。 |
| username                    | はい      | STRING    | StarRocksクラスターアカウントのユーザー名。アカウントには、読み取りたいStarRocksテーブルへの読み取り権限が必要です。[ユーザー権限](../administration/User_privilege.md)を参照してください。 |
| password                    | はい      | STRING    | StarRocksクラスターアカウントのパスワード。              |
| database-name               | はい      | STRING    | 読み取りたいStarRocksテーブルが属するStarRocksデータベースの名前。 |
| table-name                  | はい      | STRING    | 読み取りたいStarRocksテーブルの名前。            |
| scan.connect.timeout-ms     | いいえ       | STRING    | FlinkコネクタからStarRocksクラスタへの接続がタイムアウトするまでの最大時間。単位はミリ秒。デフォルト値は`1000`。接続の確立にかかる時間がこの制限を超えると、読み取りタスクは失敗します。 |
| scan.params.keep-alive-min  | いいえ       | STRING    | 読み取りタスクがアクティブである最大時間。キープアライブ時間は、ポーリングメカニズムを使用して定期的にチェックされます。単位は分。デフォルト値は`10`。このパラメータは`5`以上の値に設定することを推奨します。 |
| scan.params.query-timeout-s | いいえ       | STRING    | 読み取りタスクがタイムアウトするまでの最大時間。タイムアウト期間は、タスク実行中にチェックされます。単位は秒。デフォルト値は`600`。指定された時間が経過しても読み取り結果が返されない場合、読み取りタスクは停止します。 |
| scan.params.mem-limit-byte  | いいえ       | STRING    | 各BEでのクエリごとに許可されるメモリの最大量。単位はバイト。デフォルト値は`1073741824`、つまり1GBです。 |
| scan.max-retries            | いいえ       | STRING    | 読み取りタスクが失敗により再試行される最大回数。デフォルト値は`1`。読み取りタスクの再試行回数がこの制限を超えると、読み取りタスクはエラーを返します。 |

### Flink DataStreamのパラメータ

以下のパラメータは、Flink DataStreamの読み取り方法にのみ適用されます。

| パラメーター    | 必須 | データ型 | 説明                                                  |
| ------------ | -------- | --------- | ------------------------------------------------------------ |
| scan.columns | いいえ       | STRING    | 読み取りたい列。複数の列を指定することができ、コンマ (,) で区切る必要があります。 |
| scan.filter  | いいえ       | STRING    | データをフィルタリングするための条件式。 |

Flink で `c1`、`c2`、`c3` の3つの列からなるテーブルを作成したと仮定します。この Flink テーブルの `c1` 列の値が `100` に等しい行を読み取るには、`"scan.columns", "c1"` と `"scan.filter", "c1 = 100"` の2つのフィルタ条件を指定できます。

## StarRocks と Flink 間のデータ型マッピング

以下のデータ型マッピングは、Flink が StarRocks からデータを読み取る場合にのみ有効です。Flink が StarRocks にデータを書き込むために使用されるデータ型マッピングについては、[Apache Flink® からデータを継続的にロードする](../loading/Flink-connector-starrocks.md)を参照してください。

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

以下の例では、StarRocks クラスターに `test` という名前のデータベースを作成し、`root` ユーザーの権限を持っていることを前提としています。

> **注記**
>
> 読み取りタスクが失敗した場合は、それを再作成する必要があります。

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

### Flink SQL を使用したデータの読み取り

1. Flink クラスターで、ソース StarRocks テーブル（この例では `score_board`）のスキーマに基づいて `flink_test` という名前のテーブルを作成します。テーブル作成コマンドでは、Flink コネクタ、ソース StarRocks データベース、ソース StarRocks テーブルに関する情報など、読み取りタスクのプロパティを設定する必要があります。

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

Flink SQL を使用してデータを読み取る場合、以下の点に注意してください：

- StarRocks からデータを読み取るには `SELECT ... FROM <table_name> WHERE ...` のような SQL 文のみを使用できます。集約関数の中で `count` のみがサポートされています。
- 述語プッシュダウンがサポートされています。例えば、クエリに `char_1 <> 'A' and int_1 = -126` のフィルタ条件が含まれている場合、フィルタ条件は Flink コネクタにプッシュダウンされ、クエリが実行される前に StarRocks で実行可能なステートメントに変換されます。追加の設定は必要ありません。
- LIMIT 文はサポートされていません。
- StarRocks はチェックポイントメカニズムをサポートしていないため、読み取りタスクが失敗した場合、データの整合性を保証することはできません。

### Flink DataStream を使用したデータの読み取り

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

   `x.x.x` を使用している最新の Flink コネクタバージョンに置き換えてください。[バージョン情報](https://search.maven.org/search?q=g:com.starrocks)を参照してください。

2. Flink コネクタを呼び出して StarRocks からデータを読み取ります。

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
               env.execute("StarRocks Flink source");
           }

       }
   ```

## 次のステップ


FlinkがStarRocksからデータを正常に読み取ると、[Flink WebUI](https://nightlies.apache.org/flink/flink-docs-master/docs/try-flink/flink-operations-playground/#flink-webui)を利用して読み取りタスクをモニタリングできます。例えば、WebUIの**Metrics**ページで`totalScannedRows`メトリックを確認し、正常に読み取られた行数を把握できます。また、読み取ったデータに対してFlink SQLを用いて、ジョインなどの計算を行うことも可能です。
