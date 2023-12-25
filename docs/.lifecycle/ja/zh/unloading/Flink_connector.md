---
displayed_sidebar: Chinese
---

# Flink Connector を使用してデータを読み取る

StarRocks は独自開発した Apache Flink® Connector（StarRocks Connector for Apache Flink®）を提供し、Flink を通じて StarRocks クラスター内のデータをバッチで読み取ることができます。

Flink Connector は、Flink SQL と Flink DataStream の2つのデータ読み取り方法をサポートしています。Flink SQL の使用を推奨します。

> **説明**
>
> Flink Connector は、Flink で読み取ったデータを別の StarRocks クラスターや他のストレージシステムに書き込むこともサポートしています。[Apache Flink からの継続的なインポート](../loading/Flink-connector-starrocks.md)を参照してください。

## 機能紹介

Flink の公式に提供されている Flink JDBC Connector（以下、JDBC Connector）と比較して、StarRocks の独自開発した Flink Connector は StarRocks クラスター内の各 BE ノードからデータを並行して読み取る能力を持ち、データ読み取り効率を大幅に向上させています。以下は2つの Connector の実装方法の比較です：

- Flink Connector

  Flink はまず FE ノードからクエリプラン（Query Plan）を取得し、そのクエリプランをパラメータとして BE ノードに送信し、最後に BE ノードから返されたデータを取得します。

  ![データのアンロード - Flink Connector](../assets/unload_flink_connector_1.png)

- Flink JDBC Connector

  Flink JDBC Connector は FE ノードの単一ポイントからのみデータをシリアルに読み取ることができ、データ読み取り効率は低いです。

  ![データのアンロード - JDBC Connector](../assets/unload_flink_connector_2.png)

## バージョン要件

| Connector | Flink       | StarRocks  | Java | Scala      |
| --------- | ----------- | ---------- | ---- | ---------- |
| 1.2.9     | 1.15 ～ 1.18 | 2.1 以上   | 8    | 2.11、2.12 |
| 1.2.8     | 1.13 ～ 1.17 | 2.1 以上   | 8    | 2.11、2.12 |
| 1.2.7     | 1.11 ～ 1.15 | 2.1 以上   | 8    | 2.11、2.12 |

## 前提条件

Flink が既にデプロイされています。もし Flink をまだデプロイしていない場合は、以下の手順に従ってデプロイを完了してください：

1. オペレーティングシステムに Java 8 または Java 11 をインストールして、Flink を正常に実行します。以下のコマンドでインストールされている Java のバージョンを確認できます：

   ```SQL
   java -version
   ```

   例えば、以下のようなコマンドの出力があれば、Java 8 がインストールされていることを意味します：

   ```SQL
   openjdk version "1.8.0_322"
   OpenJDK Runtime Environment (Temurin)(build 1.8.0_322-b06)
   OpenJDK 64-Bit Server VM (Temurin)(build 25.322-b06, mixed mode)
   ```

2. [Flink](https://flink.apache.org/downloads.html) をダウンロードして解凍します。

   > **説明**
   >
   > バージョン 1.14 以上を推奨し、最低でも 1.11 がサポートされています。

   ```SQL
   # Flink をダウンロード
   wget https://dlcdn.apache.org/flink/flink-1.14.5/flink-1.14.5-bin-scala_2.11.tgz
   # Flink を解凍
   tar -xzf flink-1.14.5-bin-scala_2.11.tgz
   # Flink ディレクトリに移動
   cd flink-1.14.5
   ```

3. Flink クラスターを起動します。

   ```SQL
   # Flink クラスターを起動
   ./bin/start-cluster.sh
      
   # 以下のような情報が表示されれば、Flink クラスターの起動に成功しています
   Starting cluster.
   Starting standalonesession daemon on host.
   Starting taskexecutor daemon on host.
   ```

[Flink 公式ドキュメント](https://nightlies.apache.org/flink/flink-docs-release-1.13/docs/try-flink/local_installation/)を参照してデプロイを完了することもできます。

## 準備作業

以下の手順で Flink Connector のデプロイを完了してください：

1. Flink のバージョンに応じて、対応するバージョンの [flink-connector-starrocks](https://github.com/StarRocks/flink-connector-starrocks/releases) JAR パッケージを選択してダウンロードします。

   > **注意**
   >
   > Flink Connector のバージョンは 1.2.x 以上を推奨し、Flink のバージョンがビジネス環境にインストールされている Flink のバージョンと前2桁が一致する JAR パッケージをダウンロードすることをお勧めします。例えば、ビジネス環境にインストールされている Flink のバージョンが 1.14.x の場合は、`flink-connector-starrocks-1.2.4_flink-1.14_x.yy.jar` をダウンロードできます。

2. コードのデバッグが必要な場合は、対応するブランチのコードを自分でコンパイルすることができます。

3. ダウンロードまたはコンパイルした JAR パッケージを Flink の `lib` ディレクトリに配置します。

4. Flink を再起動します。

## パラメータ説明

### 共通パラメータ

以下のパラメータは Flink SQL と Flink DataStream の2つの読み取り方法に適用されます。

| パラメータ                        | 必須 | データ型 | 説明                                                         |
| --------------------------- | ---- | ---- | ------------------------------------------------------------ |
| connector                   | はい | STRING   | `starrocks` に固定設定してください。                         |
| scan-url                    | はい | STRING   | FE ノードの接続アドレスで、Web サーバーを通じて FE ノードにアクセスするために使用します。形式は `<fe_host>:<fe_http_port>` です。デフォルトのポート番号は `8030` です。複数のアドレスはカンマ (,) で区切ります。例：`192.168.xxx.xxx:8030,192.168.xxx.xxx:8030`。 |
| jdbc-url                    | はい | STRING   | FE ノードの接続アドレスで、FE ノード上の MySQL クライアントにアクセスするために使用します。形式は `jdbc:mysql://<fe_host>:<fe_query_port>` です。デフォルトのポート番号は `9030` です。 |
| username                    | はい | STRING   | StarRocks クラスターにアクセスするためのユーザー名です。このアカウントには読み取り対象の StarRocks テーブルの読み取り権限が必要です。ユーザー権限については[ユーザー権限](../administration/privilege_overview.md)を参照してください。 |
| password                    | はい | STRING   | StarRocks クラスターにアクセスするためのパスワードです。    |
| database-name               | はい | STRING   | 読み取り対象の StarRocks データベースの名前です。            |
| table-name                  | はい | STRING   | 読み取り対象の StarRocks テーブルの名前です。                |
| scan.connect.timeout-ms     | いいえ | STRING   | Flink Connector が StarRocks クラスターに接続するためのタイムアウト上限です。単位はミリ秒で、デフォルト値は `1000` です。この時間を超えると、データ読み取りタスクはエラーになります。 |
| scan.params.keep-alive-min  | いいえ | STRING   | データ読み取りタスクの保持時間で、定期的にポーリングによるチェックを行います。単位は分で、デフォルト値は `10` です。`5` 以上の値を推奨します。 |
| scan.params.query-timeout-s | いいえ | STRING   | データ読み取りタスクのタイムアウト時間で、タスク実行中にチェックを行います。単位は秒で、デフォルト値は `600` です。この時間を超えても読み取り結果が返されない場合は、データ読み取りタスクを停止します。 |
| scan.params.mem-limit-byte  | いいえ | STRING   | BE ノードでの単一クエリのメモリ上限です。単位はバイトで、デフォルト値は `1073741824`（1 GB）です。 |
| scan.max-retries            | いいえ | STRING   | データ読み取り失敗時の最大リトライ回数です。デフォルト値は `1` です。この数を超えると、データ読み取りタスクはエラーになります。 |

### Flink DataStream 専用パラメータ

以下のパラメータは Flink DataStream 読み取り方法にのみ適用されます。

| パラメータ         | 必須 | データ型 | 説明                                            |
| ------------ | ---- | ---- | ----------------------------------------------- |
| scan.columns | いいえ | STRING   | 読み取り対象の列を指定します。複数の列はカンマ (,) で区切ります。 |
| scan.filter  | いいえ | STRING   | フィルタ条件を指定します。                      |


Flinkで作成したテーブルに`c1`、`c2`、`c3`の3つの列が含まれているとします。Flinkテーブルの`c1`列の値が`100`である行を読み取るには、`"scan.columns", "c1"`と`"scan.filter", "c1 = 100"`を指定します。

## データ型のマッピング関係

以下のデータ型のマッピング関係は、FlinkがStarRocksからデータを読み取る場合にのみ適用されます。FlinkがデータをStarRocksに書き込む際のデータ型のマッピング関係については、[Apache Flink®からの連続インポート](../loading/Flink-connector-starrocks.md)を参照してください。

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

## 使用例

StarRocksクラスタに`test`データベースが既に作成されており、`root`アカウントの権限を持っているとします。

> **説明**
>
> 読み取りタスクが失敗した場合は、読み取りタスクを再作成する必要があります。

### データ例

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
   PROPERTIES
   (
       "replication_num" = "3"
   );
   ```

2. `score_board`テーブルにデータを挿入します。

   ```SQL
   MySQL [test]> INSERT INTO score_board values
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
   21 rows in set (0.00 sec)
   ```

### Flink SQLを使用してデータを読み取る

1. StarRocksのテーブルからインポートするデータに基づいて、Flink内で`flink_test`というテーブルを作成し、読み取りタスクのプロパティを設定します。これには、Flink Connectorとデータベースの情報を含む設定が含まれます：

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

2. SQL文を使用してStarRocksのデータを読み取ります：

   ```SQL
   SELECT id, name FROM flink_test WHERE score > 20;
   ```

Flink SQLを使用してデータを読み取る際には、以下の点に注意してください：

- StarRocksのデータを読み取るために、一部のSQL文のみがサポートされています。例えば`SELECT ... FROM <table_name> WHERE ...`。`count`以外の集約関数はサポートされていません。
- SQL文を使用する際には、自動的に述語のプッシュダウンがサポートされています。例えば、フィルタ条件`char_1 <> 'A' and int_1 = -126`は、Flink Connectorにプッシュダウンされ、StarRocksに適した文に変換された後にクエリが実行されます。追加の設定は必要ありません。
- LIMIT文はサポートされていません。
- StarRocksは現在、Checkpointメカニズムをサポートしていません。したがって、読み取りタスクが失敗した場合、データの一貫性を保証することはできません。

### Flink DataStreamを使用してデータを読み取る

1. **pom.xml**ファイルに依存関係を追加します。以下のように示されています：

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

   上記のコード例では、`x.x.x`をFlink Connectorの最新バージョン番号に置き換える必要があります。詳細は[バージョン情報](https://search.maven.org/search?q=g:com.starrocks)を参照してください。

2. Flink Connectorを呼び出し、StarRocksのデータを読み取ります。以下のように示されています：

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

## 後続操作


Flink が StarRocks からデータを正常に読み取った後、[Flink WebUI](https://nightlies.apache.org/flink/flink-docs-master/zh/docs/try-flink/flink-operations-playground/#flink-webui-界面) を使用して読み取りタスクを観察することができます。例えば、**Metrics** ページで `totalScannedRows` メトリックを確認し、正常に読み取られた行数を知ることができます。また、Flink SQL を使用して読み取ったデータに対して計算を行うこともできます。例えば、Join などです。

