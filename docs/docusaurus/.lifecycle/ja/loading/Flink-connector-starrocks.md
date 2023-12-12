---
displayed_sidebar: "Japanese"
---

# Apache Flink®からデータを連続的に読み込む

StarRocksでは、Apache Flink®向けにFlink connector（Flink connectorとも呼ばれます）という自己開発のコネクタを提供しており、これを使用してFlinkを利用してStarRocksテーブルにデータを読み込むことができます。基本的な原則は、データを蓄積してから、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を通じて一度にすべてStarRocksに読み込むことです。

Flink connectorはDataStream API、Table API & SQL、およびPython APIをサポートしています。これは、Apache Flink®が提供する[flink-connector-jdbc](https://nightlies.apache.org/flink/flink-docs-master/docs/connectors/table/jdbc/)よりも高速かつ安定したパフォーマンスを持っています。

> **注意**
>
> Flink connectorを使用してStarRocksテーブルにデータを読み込むには、SELECTおよびINSERT権限が必要です。これらの権限がない場合は、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)で指示されている手順に従って、接続するStarRocksクラスタのユーザーにこれらの権限を付与してください。

## バージョン要件

| コネクタ | Flink                    | StarRocks     | Java | Scala     |
|-----------|--------------------------|---------------| ---- |-----------|
| 1.2.8     | 1.13,1.14,1.15,1.16,1.17 | 2.1 以降     | 8    | 2.11,2.12 |
| 1.2.7     | 1.11,1.12,1.13,1.14,1.15 | 2.1 以降     | 8    | 2.11,2.12 |

## Flink connectorの取得方法

以下の方法でFlink connectorのJARファイルを取得することができます。

- コンパイルされたFlink connectorのJARファイルを直接ダウンロードします。
- MavenプロジェクトにFlink connectorを依存関係として追加し、その後JARファイルをダウンロードします。
- Flink connectorのソースコードを自分でコンパイルしてJARファイルを作成します。

Flink connectorのJARファイルの命名形式は次のとおりです。

- Flink 1.15以降は、`flink-connector-starrocks-${connector_version}_flink-${flink_version}.jar`です。例えば、Flink 1.15をインストールし、Flink connector 1.2.7を使用したい場合は、`flink-connector-starrocks-1.2.7_flink-1.15.jar`を使用できます。

- Flink 1.15より前は、`flink-connector-starrocks-${connector_version}_flink-${flink_version}_${scala_version}.jar`です。例えば、環境にFlink 1.14とScala 2.12をインストールしており、Flink connector 1.2.7を使用したい場合は、`flink-connector-starrocks-1.2.7_flink-1.14_2.12.jar`を使用できます。

> **注意**
>
> 一般的に、Flink connectorの最新バージョンは、Flinkの最新の3つのバージョンと互換性があります。

### コンパイルされたJarファイルのダウンロード

[Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks)から、対応するバージョンのFlink connector Jarファイルを直接ダウンロードします。

### Mavenの依存関係

Mavenプロジェクトの`pom.xml`ファイルで、次の形式に従ってFlink connectorを依存関係として追加します。それぞれのバージョンに対応する`flink_version`、`scala_version`、`connector_version`に置き換えてください。

- Flink 1.15以降

    ```xml
    <dependency>
        <groupId>com.starrocks</groupId>
        <artifactId>flink-connector-starrocks</artifactId>
        <version>${connector_version}_flink-${flink_version}</version>
    </dependency>
    ```

- Flink 1.15より前のバージョン

    ```xml
    <dependency>
        <groupId>com.starrocks</groupId>
        <artifactId>flink-connector-starrocks</artifactId>
        <version>${connector_version}_flink-${flink_version}_${scala_version}</version>
    </dependency>
    ```

### 自分でコンパイルする

1. [Flink connector package](https://github.com/StarRocks/starrocks-connector-for-apache-flink)をダウンロードします。
2. 次のコマンドを実行してFlink connectorのソースコードをJARファイルにコンパイルします。`flink_version`は対応するFlinkバージョンに置き換えてください。

      ```bash
      sh build.sh <flink_version>
      ```

   例えば、環境のFlinkバージョンが1.15の場合、次のコマンドを実行する必要があります。

      ```bash
      sh build.sh 1.15
      ```

3. コンパイル後、`target/`ディレクトリに移動して、コンパイル中に生成された`flink-connector-starrocks-1.2.7_flink-1.15-SNAPSHOT.jar`などのFlink connector JARファイルを見つけます。

> **注意**
>
> 正式にリリースされていないFlink connectorの名前には、`SNAPSHOT`接尾辞が含まれています。

## オプション

| **オプション**                    | **必須** | **デフォルト値** | **説明**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
|-----------------------------------|----------|-------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| connector                         | Yes      | NONE              | 使用するコネクタ。値は「starrocks」でなければなりません。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| jdbc-url                          | Yes      | NONE              | FEのMySQLサーバに接続するためのアドレス。複数のアドレスを指定することができ、コンマ（,）で区切られていなければなりません。形式: `jdbc:mysql://<fe_host1>:<fe_query_port1>,<fe_host2>:<fe_query_port2>,<fe_host3>:<fe_query_port3>`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| load-url                          | Yes      | NONE              | StarRocksクラスタ内のFEのHTTP URL。複数のURLを指定することができ、セミコロン（;）で区切られていなければなりません。形式: `<fe_host1>:<fe_http_port1>;<fe_host2>:<fe_http_port2>`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| database-name                     | Yes      | NONE              | データを読み込むStarRocksデータベースの名前。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| table-name                        | Yes      | NONE              | StarRocksにデータを読み込むために使用するテーブルの名前。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| username                          | Yes      | NONE              | StarRocksにデータを読み込むために使用するアカウントのユーザー名。アカウントには[SELECTおよびINSERT権限](../sql-reference/sql-statements/account-management/GRANT.md)が必要です。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| password                          | Yes      | NONE              | 前述のアカウントのパスワード。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| sink.semantic                     | No       | at-least-once     | Sinkによって保証される意味。有効な値：**at-least-once**および**exactly-once**。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| sink.version                      | No       | AUTO              | データを読み込むために使用されるインタフェース。このパラメータは、Flink connectorバージョン1.2.4以降でサポートされています。<ul><li>`V1`: [Stream Load](../loading/StreamLoad.md)インタフェースを使用してデータを読み込む。1.2.4以前のコネクタはこのモードのみをサポートしています。 </li> <li>`V2`: [Stream Load transaction](../loading/Stream_Load_transaction_interface.md)インタフェースを使用してデータを読み込む。StarRocksは少なくともバージョン2.4である必要があります。メモリ使用量を最適化し、より安定したexactly-once実装を提供するため、`V2`を推奨します。 </li> <li>`AUTO`: StarRocksのバージョンがトランザクションStream Loadをサポートしている場合は自動的に`V2`を選択し、それ以外の場合は`V1`を選択します</li></ul> |
| sink.label-prefix | No | NONE | Stream Loadで使用するラベルの接頭辞。コネクタ1.2.8以降でexactly-onceを使用している場合は、構成することを推奨します。[exactly-onceの使用注意事項](#exactly-once)を参照してください。 |
| sink.buffer-flush.max-bytes       | No       | 94371840(90M)     | StarRocksに一度に送信される前にメモリに蓄積できるデータの最大サイズ。最大値は64MBから10GBまでです。このパラメータを大きな値に設定することで読み込みパフォーマンスを向上させることができますが、読み込みの遅延が発生する可能性があります。                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| sink.buffer-flush.max-rows        | No       | 500000            | StarRocksに一度に送信される前にメモリに蓄積できる行の最大数。このパラメータは、`sink.version`を`V1`に設定し、`sink.semantic`を`at-least-once`に設定している場合にのみ利用可能です。有効な値：64000～5000000。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| sink.buffer-flush.interval-ms     | No           | 300,000            | データがフラッシュされる間隔。このパラメータは`sink.semantic`が`at-least-once`に設定されている場合のみ利用可能です。有効な値: 1000 ～ 3600000。単位: ms。|
| sink.max-retries                  | No           | 3                 | システムがストリームロードジョブを実行し直す回数。このパラメータは`sink.version`が`V1`に設定されている場合のみ利用可能です。有効な値: 0 ～ 10。|
| sink.connect.timeout-ms           | No           | 1,000              | HTTP接続を確立するためのタイムアウト。有効な値: 100 ～ 60,000。単位: ms。|
| sink.wait-for-continue.timeout-ms | No           | 10,000             | バージョン1.2.7からサポートされています。FEからHTTP 100-continueの応答を待機するタイムアウト。有効な値: `3000` ～ `60000`。単位: ms。|
| sink.ignore.update-before         | No           | true              | バージョン1.2.8からサポートされています。Flinkからプライマリキーテーブルにデータをロードする際に`UPDATE_BEFORE`レコードを無視するかどうか。このパラメータが`false`に設定されている場合、そのレコードはStarRocksテーブルにおいて削除操作として扱われます。|
| sink.properties.*                 | No           | NONE              | ストリームロードの動作を制御するために使用されるパラメータ。例えば、パラメータ`sink.properties.format`は、CSVやJSONなどのストリームロードに使用するフォーマットを指定します。サポートされているパラメータとその説明については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。|
| sink.properties.format            | No           | csv               | ストリームロードに使用されるフォーマット。Flinkコネクタはデータの各バッチをStarRocksに送信する前に、それらを指定のフォーマットに変換します。有効な値: `csv` および `json`。|
| sink.properties.row_delimiter     | No           | \n                | CSV形式のデータの行区切り文字。|
| sink.properties.column_separator  | No           | \t                | CSV形式のデータの列区切り文字。|
| sink.properties.max_filter_ratio  | No           | 0                 | ストリームロードの最大エラートレランス。データ品質が不十分なためにフィルタリングされるデータレコードの最大パーセンテージ。有効な値: `0` ～ `1`。デフォルト値: `0`。詳細については[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。|
| sink.parallelism                  | No           | NONE              | コネクタの並列処理。Flink SQL専用です。未設定の場合、Flinkプランナーが並列処理を決定します。マルチ並列処理のシナリオでは、ユーザーはデータが正しい順序で書き込まれることを保証する必要があります。|
| sink.properties.strict_mode | No | false | ストリームロードの厳密モードを有効にするかどうかを指定します。これは一貫性のない列値などの資格のない行が存在する場合のロード動作に影響します。有効な値: `true` および `false`。デフォルト値: `false`。詳細については[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。|

## FlinkとStarRocksのデータ型マッピング

| Flinkのデータ型                   | StarRocksのデータ型   |
|-----------------------------------|-----------------------|
| BOOLEAN                           | BOOLEAN               |
| TINYINT                           | TINYINT               |
| SMALLINT                          | SMALLINT              |
| INTEGER                           | INTEGER               |
| BIGINT                            | BIGINT                |
| FLOAT                             | FLOAT                 |
| DOUBLE                            | DOUBLE                |
| DECIMAL                           | DECIMAL               |
| BINARY                            | INT                   |
| CHAR                              | STRING                |
| VARCHAR                           | STRING                |
| STRING                            | STRING                |
| DATE                              | DATE                  |
| TIMESTAMP_WITHOUT_TIME_ZONE(N)    | DATETIME              |
| TIMESTAMP_WITH_LOCAL_TIME_ZONE(N) | DATETIME              |
| ARRAY&lt;T&gt;                        | ARRAY&lt;T&gt;              |
| MAP&lt;KT,VT&gt;                        | JSON STRING           |
| ROW&lt;arg T...&gt;                     | JSON STRING           |

## 使用上の注意

### 正確に一度だけ

- シンクが正確に一度だけのセマンティクスを保証する場合は、StarRocksをバージョン2.5以上にアップグレードし、Flinkコネクタをバージョン1.2.4以上にアップグレードすることをお勧めします
  - Flinkコネクタ1.2.4以降では、正確な一度だけは、StarRocksの2.4以降で提供されている[Stream Loadトランザクションインターフェース](https://docs.starrocks.io/en-us/latest/loading/Stream_Load_transaction_interface)に基づいて再設計されています。非トランザクショナルインターフェースに基づいた以前の実装と比較して、新しい実装はメモリ使用量とチェックポイントのオーバーヘッドを削減し、リアルタイム性能と安定性を向上させています。

  - StarRocksのバージョンが2.4よりも古い場合やFlinkコネクタのバージョンが1.2.4よりも古い場合は、シンクは自動的に非トランザクショナルインターフェースに基づいた実装を選択します。

- 正確な一度だけを保証するための設定

  - `sink.semantic`の値は`exactly-once`である必要があります。

  - Flinkコネクタのバージョンが1.2.8以降の場合、`sink.label-prefix`の値を指定することが推奨されます。ただし、ラベルプレフィックスはFlinkジョブ、Routine Load、Broker LoadなどStarRocksのあらゆるロードタイプでユニークである必要があります。

    - ラベルプレフィックスが指定されている場合、Flinkコネクタは、いくつかのFlinkの障害シナリオで発生する可能性がある残存トランザクションをクリーンアップするためにラベルプレフィックスを使用します。これらの残存トランザクションは通常、`PREPARED`の状態にあります。Flinkジョブがチェックポイントが進行中の際にFlinkジョブが失敗した場合などのFlinkジョブが回復する際、Flinkコネクタはこれらの残存トランザクションをチェックポイント内のラベルプレフィックスと一部の情報に基づいて見つけ、中止します。Flinkジョブが終了すると、Flinkコネクタは正確な一度だけを実現するための二段階コミットメカニズムによって、これらのトランザクションを中止できません。Flinkジョブが終了すると、Flinkチェックポイントコーディネータからトランザクションを正常なチェックポイントに含めるかどうかの通知をまだ受信していないため、これらのトランザクションを中止してしまう可能性があります。そのため、Flinkにおけるエンドツーエンドの正確に一度だけを実現する方法については、この[ブログポスト](https://flink.apache.org/2018/02/28/an-overview-of-end-to-end-exactly-once-processing-in-apache-flink-with-apache-kafka-too/)を参照してください。

    - ラベルプレフィックスが指定されていない場合、残存トランザクションはStarRocksによってタイムアウト後にクリーンアップされます。ただし、Flinkジョブが頻繁に失敗すると、残存トランザクションの数がStarRocks `max_running_txn_num_per_db`の制限に達する可能性があります。タイムアウトの長さは、StarRocks FE構成の`prepared_transaction_default_timeout_second`によって制御されます。デフォルト値は`86400`（1日）です。これをより小さな値に設定することで、ラベルプレフィックスが指定されていない場合にトランザクションをより早く期限切れにすることができます。

- Flinkジョブが長時間停止することによる停止または継続的なフェイルオーバーのため、最終的にチェックポイントまたはセーブポイントからのFlinkジョブの回復が保証されている場合は、データ損失を防ぐために次のStarRocksの構成を調整してください。

  - `prepared_transaction_default_timeout_second`：StarRocks FEの構成。デフォルト値は `86400`です。この構成の値は、Flinkジョブのダウンタイムよりも大きくする必要があります。そうでないと、成功したチェックポイントに含まれている残存トランザクションは、Flinkジョブを再起動する前にタイムアウトのため中止される可能性があります。

    この構成に大きな値を設定する場合、ラベルプレフィックスの値を指定して、チェックポイント内のトランザクションラベルを使用してStarRocks内のトランザクションの状態を確認し、これらのトランザクションがコミットされたかどうかを判別できるようにした方が良いでしょう。

  - `label_keep_max_second` および `label_keep_max_num`：StarRocks FE構成。デフォルト値はそれぞれ `259200` および `1000` です。詳細については、[FE configurations](../loading/Loading_intro.md#fe-configurations)をご覧ください。 `label_keep_max_second`の値は、Flinkジョブのダウンタイムよりも大きくする必要があります。そうでないと、FlinkコネクタはFlinkのセーブポイントまたはチェックポイントに保存されたトランザクションラベルを使用してStarRocks内のトランザクションの状態を確認し、これらのトランザクションがコミットされたかどうかを判断できません。

  これらの構成は変更可能であり、`ADMIN SET FRONTEND CONFIG`を使用して変更できます。

  ```SQL
    ADMIN SET FRONTEND CONFIG ("prepared_transaction_default_timeout_second" = "3600");
    ADMIN SET FRONTEND CONFIG ("label_keep_max_second" = "259200");
    ADMIN SET FRONTEND CONFIG ("label_keep_max_num" = "1000");
  ```

### フラッシュポリシー
フリンク・コネクタはデータをメモリ内にバッファし、それをバッチでStarRocksにStream Loadを通じてフラッシュします。フラッシュがトリガーされる方法は、少なくとも一度とまさに一度の間で異なります。

少なくとも一度の場合、以下のいずれかの条件が満たされたときにフラッシュがトリガーされます。

- バッファリングされた行のバイトが`sink.buffer-flush.max-bytes`の制限に達した場合
- バッファリングされた行の数が`sink.buffer-flush.max-rows`の制限に達した場合（シンクバージョンV1にのみ有効）
- 最後のフラッシュ以降の経過時間が`sink.buffer-flush.interval-ms`の制限に達した場合
- チェックポイントがトリガーされた場合

まさに一度の場合、フラッシュはチェックポイントがトリガーされたときのみ発生します。

### ロードメトリクスのモニタリング

フリンクコネクタは、以下のメトリクスを提供してローディングを監視します。

| メトリクス                 | タイプ    | 説明                                                     |
|--------------------------|---------|-----------------------------------------------------------------|
| totalFlushBytes          | カウンター | 成功したフラッシュバイト数                                     |
| totalFlushRows           | カウンター | 成功した行の数                                      |
| totalFlushSucceededTimes | カウンター | データが成功してフラッシュされた回数  |
| totalFlushFailedTimes    | カウンター | データのフラッシュに失敗した回数                |
| totalFilteredRows        | カウンター | フィルタリングされた行の数。これはtotalFlushRowsにも含まれます。  |

## 例

以下の例では、Flinkコネクタを使用してFlink SQLまたはFlink DataStreamを使用してStarRocksテーブルにデータをロードする方法を示しています。

### 準備

#### StarRocksテーブルの作成

データベース`test`を作成し、プライマリーキーテーブル`score_board`を作成します。

```sql
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

#### Flink環境のセットアップ

- Flinkのバイナリ[Flink 1.15.2](https://archive.apache.org/dist/flink/flink-1.15.2/flink-1.15.2-bin-scala_2.12.tgz)をダウンロードし、`flink-1.15.2`ディレクトリに展開します。
- [Flink connector 1.2.7](https://repo1.maven.org/maven2/com/starrocks/flink-connector-starrocks/1.2.7_flink-1.15/flink-connector-starrocks-1.2.7_flink-1.15.jar)をダウンロードし、`flink-1.15.2/lib`ディレクトリに配置します。
- 次のコマンドを実行してFlinkクラスターを起動します。

    ```shell
    cd flink-1.15.2
    ./bin/start-cluster.sh
    ```

### Flink SQLで実行

- 次のコマンドを実行してFlink SQLクライアントを起動します。

    ```shell
    ./bin/sql-client.sh
    ```

- Flinkテーブル`score_board`を作成し、Flink SQLクライアントを介してテーブルに値を挿入します。StarRocksのプライマリーキーテーブルにデータをロードする場合は、Flink DDLでプライマリーキーを定義する必要があります。その他の種類のStarRocksテーブルの場合はオプショナルです。

    ```SQL
    CREATE TABLE `score_board` (
        `id` INT,
        `name` STRING,
        `score` INT,
        PRIMARY KEY (id) NOT ENFORCED
    ) WITH (
        'connector' = 'starrocks',
        'jdbc-url' = 'jdbc:mysql://127.0.0.1:9030',
        'load-url' = '127.0.0.1:8030',
        'database-name' = 'test',
        
        'table-name' = 'score_board',
        'username' = 'root',
        'password' = ''
    );

    INSERT INTO `score_board` VALUES (1, 'starrocks', 100), (2, 'flink', 100);
    ```

### Flink DataStreamで実行

入力レコードのタイプに応じて、CSV Java `String`、JSON Java `String`、またはカスタムJavaオブジェクトのFlink DataStreamジョブを実装する方法がいくつかあります。

- 入力レコードがCSV形式の`String`の場合は、完全な例については[LoadCsvRecords](https://github.com/StarRocks/starrocks-connector-for-apache-flink/tree/main/examples/src/main/java/com/starrocks/connector/flink/examples/datastream/LoadCsvRecords.java)を参照してください。

    ```java
    /**
     * CSV形式のレコードを生成します。各レコードは"\t"で区切られた3つの値を持ちます。
     * これらの値はStarRocksテーブルの`id`、`name`、および`score`にロードされます。
     */
    String[] records = new String[]{
            "1\tstarrocks-csv\t100",
            "2\tflink-csv\t100"
    };
    DataStream<String> source = env.fromElements(records);

    /**
     * 必要なプロパティでコネクタを構成します。
     * また、プロパティ"sink.properties.format"および"sink.properties.column_separator"を追加する必要があります。
     * これにより、入力レコードがCSV形式であること、および列の区切り記号が"\t"であることをコネクタに伝えることができます。
     */
    StarRocksSinkOptions options = StarRocksSinkOptions.builder()
            .withProperty("jdbc-url", jdbcUrl)
            .withProperty("load-url", loadUrl)
            .withProperty("database-name", "test")
            .withProperty("table-name", "score_board")
            .withProperty("username", "root")
            .withProperty("password", "")
            .withProperty("sink.properties.format", "csv")
            .withProperty("sink.properties.column_separator", "\t")
            .build();
    // オプションを使用してシンクを作成します。
    SinkFunction<String> starRockSink = StarRocksSink.sink(options);
    source.addSink(starRockSink);
    ```

- 入力レコードがJSON形式の`String`の場合は、完全な例については[LoadJsonRecords](https://github.com/StarRocks/starrocks-connector-for-apache-flink/tree/main/examples/src/main/java/com/starrocks/connector/flink/examples/datastream/LoadJsonRecords.java)を参照してください。

    ```java
    /**
     * JSON形式のレコードを生成します。
     * 各レコードにはStarRocksテーブルの`id`、`name`、`score`に対応する3つのキーバリューペアがあります。
     */
    String[] records = new String[]{
            "{\"id\":1, \"name\":\"starrocks-json\", \"score\":100}",
            "{\"id\":2, \"name\":\"flink-json\", \"score\":100}",
    };
    DataStream<String> source = env.fromElements(records);

    /** 
     * 必要なプロパティでコネクタを構成します。
     * また、プロパティ"sink.properties.format"と"sink.properties.strip_outer_array"を追加する必要があります。
     * これにより、入力レコードがJSON形式であり、最も外側の配列構造を剥がすことをコネクタに伝えることができます。
     */
    StarRocksSinkOptions options = StarRocksSinkOptions.builder()
            .withProperty("jdbc-url", jdbcUrl)
            .withProperty("load-url", loadUrl)
            .withProperty("database-name", "test")
            .withProperty("table-name", "score_board")
            .withProperty("username", "root")
            .withProperty("password", "")
            .withProperty("sink.properties.format", "json")
            .withProperty("sink.properties.strip_outer_array", "true")
            .build();
    // オプションを使用してシンクを作成します。
    SinkFunction<String> starRockSink = StarRocksSink.sink(options);
    source.addSink(starRockSink);
    ```

- 入力レコードがカスタムJavaオブジェクトの場合は、[LoadCustomJavaRecords](https://github.com/StarRocks/starrocks-connector-for-apache-flink/tree/main/examples/src/main/java/com/starrocks/connector/flink/examples/datastream/LoadCustomJavaRecords.java)の完全な例を参照してください。
```java
          .build();

    /**
     * The Flink connector will use a Java object array (Object[]) to represent a row to be loaded into the StarRocks table,
     * and each element is the value for a column.
     * You need to define the schema of the Object[] which matches that of the StarRocks table.
     */
    TableSchema schema = TableSchema.builder()
            .field("id", DataTypes.INT().notNull())
            .field("name", DataTypes.STRING())
            .field("score", DataTypes.INT())
            // When the StarRocks table is a Primary Key table, you must specify notNull(), for example, DataTypes.INT().notNull(), for the primary key `id`.
            .primaryKey("id")
            .build();
    // Transform the RowData to the Object[] according to the schema.
    RowDataTransformer transformer = new RowDataTransformer();
    // Create the sink with the schema, options, and transformer.
    SinkFunction<RowData> starRockSink = StarRocksSink.sink(schema, options, transformer);
    source.addSink(starRockSink);
    ```

  - The `RowDataTransformer` in the main program is defined as follows:

    ```java
    private static class RowDataTransformer implements StarRocksSinkRowBuilder<RowData> {
    
        /**
         * Set each element of the object array according to the input RowData.
         * The schema of the array matches that of the StarRocks table.
         */
        @Override
        public void accept(Object[] internalRow, RowData rowData) {
            internalRow[0] = rowData.id;
            internalRow[1] = rowData.name;
            internalRow[2] = rowData.score;
            // When the StarRocks table is a Primary Key table, you need to set the last element to indicate whether the data loading is an UPSERT or DELETE operation.
            internalRow[internalRow.length - 1] = StarRocksSinkOP.UPSERT.ordinal();
        }
    }  
    ```

## Best practices

### Load data to a Primary Key table

This section will show how to load data to a StarRocks Primary Key table to achieve partial updates and conditional updates.
You can see [Change data through loading](https://docs.starrocks.io/en-us/latest/loading/Load_to_Primary_Key_tables) for the introduction of those features.
These examples use Flink SQL.

#### Preparations

Create a database `test` and create a Primary Key table `score_board` in StarRocks.

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

#### Partial update

This example will show how to load data only to columns `id` and `name`.

1. Insert two data rows into the StarRocks table `score_board` in MySQL client.

    ```SQL
    mysql> INSERT INTO `score_board` VALUES (1, 'starrocks', 100), (2, 'flink', 100);

    mysql> select * from score_board;
    +------+-----------+-------+
    | id   | name      | score |
    +------+-----------+-------+
    |    1 | starrocks |   100 |
    |    2 | flink     |   100 |
    +------+-----------+-------+
    2 rows in set (0.02 sec)
    ```

2. Create a Flink table `score_board` in Flink SQL client.

   - Define the DDL which only includes the columns `id` and `name`.
   - Set the option `sink.properties.partial_update` to `true` which tells the Flink connector to perform partial updates.
   - If the Flink connector version `<=` 1.2.7, you also need to set the option `sink.properties.columns` to `id,name,__op` to tells the Flink connector which columns need to be updated. Note that you need to append the field `__op` at the end. The field `__op` indicates that the data loading is an UPSERT or DELETE operation, and its values are set by the connector automatically.

    ```SQL
    CREATE TABLE `score_board` (
        `id` INT,
        `name` STRING,
        PRIMARY KEY (id) NOT ENFORCED
    ) WITH (
        'connector' = 'starrocks',
        'jdbc-url' = 'jdbc:mysql://127.0.0.1:9030',
        'load-url' = '127.0.0.1:8030',
        'database-name' = 'test',
        'table-name' = 'score_board',
        'username' = 'root',
        'password' = '',
        'sink.properties.partial_update' = 'true',
        -- only for Flink connector version <= 1.2.7
        'sink.properties.columns' = 'id,name,__op'
    ); 
    ```

3. Insert two data rows into the Flink table. The primary keys of the data rows are as same as these of rows in the StarRocks table. but the values in the column `name` are modified.

    ```SQL
    INSERT INTO `score_board` VALUES (1, 'starrocks-update'), (2, 'flink-update');
    ```

4. Query the StarRocks table in MySQL client.
  
    ```SQL
    mysql> select * from score_board;
    +------+------------------+-------+
    | id   | name             | score |
    +------+------------------+-------+
    |    1 | starrocks-update |   100 |
    |    2 | flink-update     |   100 |
    +------+------------------+-------+
    2 rows in set (0.02 sec)
    ```

    You can see that only values for `name` change, and the values for `score` do not change.

#### Conditional update

This example will show how to do conditional update according to the value of column `score`. The update for an `id`
takes effect only when the new value for `score` is has a greater or equal to the old value.

1. Insert two data rows into the StarRocks table in MySQL client.

    ```SQL
    mysql> INSERT INTO `score_board` VALUES (1, 'starrocks', 100), (2, 'flink', 100);
    
    mysql> select * from score_board;
    +------+-----------+-------+
    | id   | name      | score |
    +------+-----------+-------+
    |    1 | starrocks |   100 |
    |    2 | flink     |   100 |
    +------+-----------+-------+
    2 rows in set (0.02 sec)
    ```

2. Create a Flink table `score_board` in the following ways:
  
    - Define the DDL including all of columns.
    - Set the option `sink.properties.merge_condition` to `score` to tell the connector to use the column `score`
    as the condition.
    - Set the option `sink.version` to `V1` which tells the connector to use Stream Load.

    ```SQL
    CREATE TABLE `score_board` (
        `id` INT,
        `name` STRING,
        `score` INT,
        PRIMARY KEY (id) NOT ENFORCED
    ) WITH (
        'connector' = 'starrocks',
        'jdbc-url' = 'jdbc:mysql://127.0.0.1:9030',
        'load-url' = '127.0.0.1:8030',
        'database-name' = 'test',
        'table-name' = 'score_board',
        'username' = 'root',
        'password' = '',
        'sink.properties.merge_condition' = 'score',
        'sink.version' = 'V1'
        );
    ```

3. Insert two data rows into the Flink table. The primary keys of the data rows are as same as these of rows in the StarRocks table. The first data row has a smaller value in the column `score`, and the second data row has a larger  value in the column `score`.

    ```SQL
    INSERT INTO `score_board` VALUES (1, 'starrocks-update', 99), (2, 'flink-update', 101);
    ```

4. Query the StarRocks table in MySQL client.

    ```SQL
    mysql> select * from score_board;
    +------+--------------+-------+
    | id   | name         | score |
    +------+--------------+-------+
    |    1 | starrocks    |   100 |
    |    2 | flink-update |   101 |
    +------+--------------+-------+
    2 rows in set (0.03 sec)
    ```

   You can see that only the values of the second data row change, and the values of the first data row do not change.

### Load data into columns of BITMAP type

[`BITMAP`](https://docs.starrocks.io/en-us/latest/sql-reference/sql-statements/data-types/BITMAP) is often used to accelerate count distinct, such as counting UV, see [Use Bitmap for exact Count Distinct](https://docs.starrocks.io/en-us/latest/using_starrocks/Using_bitmap).
```SQL
      + 
      + 
    + 
  + 
```
```SQL
      + 
      + 
    + 
  + 
```