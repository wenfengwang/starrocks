---
displayed_sidebar: Chinese
---
---
displayed_sidebar: "Chinese"
---

# Apache Flink® からの継続的なインポート

StarRocksはApache Flink® コネクタ（以下、Flink connector）を提供しており、Flinkを介してStarRocksテーブルにデータをインポートすることができます。

基本原理は、Flink connectorがメモリ内で小バッチのデータを蓄積し、[Stream Load](./StreamLoad.md)を通じて一度にStarRocksにインポートすることです。

Flink ConnectorはDataStream API、Table API & SQL、およびPython APIをサポートしています。

StarRocksが提供するFlink connectorは、Flinkが提供する[flink-connector-jdbc](https://nightlies.apache.org/flink/flink-docs-master/docs/connectors/table/jdbc/)と比較して、パフォーマンスが優れ、安定しています。

> **注意**
>
> Flink connectorを使用してStarRocksにデータをインポートするには、対象テーブルのSELECTとINSERT権限が必要です。これらの権限がない場合は、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)を参照してユーザーに権限を付与してください。

## バージョン要件

| Connector | Flink       | StarRocks  | Java | Scala      |
| --------- | ----------- | ---------- | ---- | ---------- |
| 1.2.9     | 1.15 ～ 1.18 | 2.1 以上 | 8    | 2.11、2.12 |
| 1.2.8     | 1.13 ～ 1.17 | 2.1 以上 | 8    | 2.11、2.12 |
| 1.2.7     | 1.11 ～ 1.15 | 2.1 以上 | 8    | 2.11、2.12 |

## Flink connectorの取得

以下の方法でFlink connectorのJARファイルを取得できます：

- 既にコンパイルされたJARファイルを直接ダウンロードします。
- MavenプロジェクトのpomファイルにFlink connectorを依存関係として追加し、ダウンロードします。
- ソースコードから手動でJARファイルをコンパイルします。

Flink connector JARファイルの命名規則は以下の通りです：

- Flink 1.15以降のバージョン用のFlink connectorの命名規則は`flink-connector-starrocks-${connector_version}_flink-${flink_version}.jar`です。例えば、Flink 1.15をインストールし、1.2.7バージョンのFlink connectorを使用したい場合は、`flink-connector-starrocks-1.2.7_flink-1.15.jar`を使用できます。
- Flink 1.15以前のバージョン用のFlink connectorの命名規則は`flink-connector-starrocks-${connector_version}_flink-${flink_version}_${scala_version}.jar`です。例えば、Flink 1.14とScala 2.12をインストールし、1.2.7バージョンのFlink connectorを使用したい場合は、`flink-connector-starrocks-1.2.7_flink-1.14_2.12.jar`を使用できます。

> **注意**
>
> 通常、最新バージョンのFlink connectorは最新の3つのFlinkバージョンのみをサポートしています。

### 直接ダウンロード

[Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks)で異なるバージョンのFlink connector JARファイルを取得できます。

### Maven依存関係

Mavenプロジェクトの`pom.xml`ファイルに、以下の形式でFlink connectorを依存関係として追加します。`flink_version`、`scala_version`、`connector_version`をそれぞれ対応するバージョンに置き換えてください。

- Flink 1.15以降のバージョン用のFlink connector

    ```XML
    <dependency>
        <groupId>com.starrocks</groupId>
        <artifactId>flink-connector-starrocks</artifactId>
        <version>${connector_version}_flink-${flink_version}</version>
    </dependency>
    ```

- Flink 1.15以前のバージョン用のFlink connector

    ```XML
    <dependency>
        <groupId>com.starrocks</groupId>
        <artifactId>flink-connector-starrocks</artifactId>
        <version>${connector_version}_flink-${flink_version}_${scala_version}</version>
    </dependency>
    ```

### 手動コンパイル

1. [Flink connectorのソースコード](https://github.com/StarRocks/starrocks-connector-for-apache-flink)をダウンロードします。
2. 以下のコマンドを実行して、Flink connectorのソースコードをJARファイルにコンパイルします。`flink_version`を対応するFlinkバージョンに置き換えてください。

    ```Bash
    sh build.sh <flink_version>
    ```

    例えば、環境にFlink 1.15がインストールされている場合は、以下のコマンドを実行します：

    ```Bash
    sh build.sh 1.15
    ```

3. `target/`ディレクトリに移動し、コンパイルされたFlink connector JARファイルを探します。例えば、`flink-connector-starrocks-1.2.7_flink-1.15-SNAPSHOT.jar`はコンパイルプロセス中に生成されます。

    > **注意**：
    >
    > 正式にリリースされていないFlink connectorの名前には`SNAPSHOT`の接尾辞が含まれます。

## パラメータ説明

| パラメータ                         | 必須 | デフォルト値    | 説明                                                         |
| --------------------------------- | ---- | ------------- | ------------------------------------------------------------ |
| connector                         | はい | なし          | `starrocks`に固定設定してください。                          |
| jdbc-url                          | はい | なし          | FEノード上のMySQLサーバーにアクセスするために使用します。複数のアドレスはカンマ（,）で区切ります。形式：`jdbc:mysql://<fe_host1>:<fe_query_port1>,<fe_host2>:<fe_query_port2>`。 |
| load-url                          | はい | なし          | FEノード上のHTTPサーバーにアクセスするために使用します。複数のアドレスはセミコロン（;）で区切ります。形式：`<fe_host1>:<fe_http_port1>;<fe_host2>:<fe_http_port2>`。 |
| database-name                     | はい | なし          | StarRocksのデータベース名。                                  |
| table-name                        | はい | なし          | StarRocksのテーブル名。                                      |
| username                          | はい | なし          | StarRocksクラスタのユーザー名。Flink connectorを使用してStarRocksにデータをインポートするには、対象テーブルのSELECTとINSERT権限が必要です。これらの権限がない場合は、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)を参照してユーザーに権限を付与してください。|
| password                          | はい | なし          | StarRocksクラスタのユーザーパスワード。                      |
| sink.semantic                     | いいえ | at-least-once | sinkが保証するセマンティクス。有効な値：**at-least-once** と **exactly-once**。 |
| sink.version                      | いいえ | AUTO         | データをインポートするためのインターフェース。このパラメータはFlink connector 1.2.4からサポートされています。<ul><li>V1：[Stream Load](./StreamLoad.md)インターフェースを使用してデータをインポートします。1.2.4以前のFlink connectorはこのモードのみをサポートしていました。</li><li>V2：[Stream Loadトランザクションインターフェース](../loading/Stream_Load_transaction_interface.md)を使用してデータをインポートします。StarRocksのバージョンが2.4以上であることが要求されます。V2を選択することをお勧めします。これはメモリ使用量を削減し、より安定したexactly-onceの実装を提供します。</li><li>AUTO：StarRocksのバージョンがStream Loadトランザクションインターフェースをサポートしている場合は、自動的にV2が選択されます。そうでない場合はV1が選択されます。</li></ul> |
| sink.label-prefix                 | いいえ | なし          | Stream Loadで使用するlabelのプレフィックスを指定します。Flink connectorのバージョンが1.2.8以上で、sinkがexactly-onceセマンティクスを保証する場合は、labelプレフィックスを設定することをお勧めします。詳細は[exactly once](#exactly-once)を参照してください。|
| sink.buffer-flush.max-bytes       | いいえ | 94371840(90M) | メモリ内に蓄積されたデータのサイズがこの閾値に達すると、Stream Loadを通じて一度にStarRocksにデータがインポートされます。値の範囲：[64MB, 10GB]。このパラメータを大きな値に設定すると、インポートのパフォーマンスは向上しますが、インポートの遅延が増加する可能性があります。このパラメータは`sink.semantic`が`at-least-once`の場合にのみ有効です。`sink.semantic`が`exactly-once`の場合は、Flinkのチェックポイントがトリガーされたときにのみメモリ内のデータがフラッシュされるため、このパラメータは無効です。|
| sink.buffer-flush.max-rows        | いいえ | 500000        | メモリ内に蓄積されたデータの行数がこの閾値に達すると、Stream Loadを通じて一度にStarRocksにデータがインポートされます。値の範囲：[64000, 5000000]。このパラメータは`sink.version`が`V1`で、`sink.semantic`が`at-least-once`の場合にのみ有効です。|
| sink.buffer-flush.interval-ms     | いいえ | 300000        | データの送信間隔で、StarRocksへのデータ書き込みの遅延を制御するために使用されます。値の範囲：[1000, 3600000]。このパラメータは`sink.semantic`が`at-least-once`の場合にのみ有効です。 |
| sink.max-retries                  | いいえ | 3             | Stream Loadが失敗した後のリトライ回数。この数を超えると、データインポートタスクがエラーになります。値の範囲：[0, 10]。このパラメータは`sink.version`が`V1`の場合にのみ有効です。 |
| sink.connect.timeout-ms           | いいえ | 30000         | FEとHTTP接続を確立するためのタイムアウト時間。値の範囲：[100, 60000]。Flink connector v1.2.9以前は、デフォルト値は`1000`でした。 |
| sink.wait-for-continue.timeout-ms | いいえ | 10000         | このパラメータはFlink connector 1.2.7からサポートされています。FEのHTTP 100-continue応答を待つためのタイムアウト時間。値の範囲：[3000, 60000]。 |
| sink.ignore.update-before         | いいえ | true          | このパラメータはFlink connector 1.2.8からサポートされています。プライマリキーモデルのテーブルにデータをインポートする際に、FlinkからのUPDATE_BEFOREレコードを無視するかどうか。このパラメータをfalseに設定すると、そのレコードはプライマリキーモデルのテーブルでDELETE操作として扱われます。 |
| sink.parallelism                  | No       | NONE          | 書き込みの並列度。Flink SQLにのみ適用されます。設定されていない場合、Flink plannerが並列度を決定します。**複数の並列度のシナリオでは、ユーザーはデータが正しい順序で書き込まれることを保証する必要があります。** |
| sink.properties.*                 | No       | NONE          | Stream Loadのパラメータで、Stream Loadのインポート動作を制御します。例えば、パラメータsink.properties.formatはStream Loadでインポートされるデータの形式を示し、CSVまたはJSONとなります。すべてのパラメータと説明については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。|
| sink.properties.format            | No       | csv           | Stream Loadでのインポート時のデータ形式。Flink connectorはメモリ内のデータを対応する形式に変換し、Stream Loadを通じてStarRocksにインポートします。CSVまたはJSONを取ることができます。 |
| sink.properties.column_separator  | No       | \t            | CSVデータの列区切り文字。                                         |
| sink.properties.row_delimiter     | No       | \n            | CSVデータの行区切り文字。                                         |
| sink.properties.max_filter_ratio  | No       | 0             | インポートジョブの最大許容エラー率、つまりデータ品質が標準に達していないためにフィルタリングされたデータ行が占める最大割合。範囲：0〜1。デフォルト値：0。詳細は[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。 |

## データ型マッピング

| Flinkデータ型                    | StarRocksデータ型 |
| --------------------------------- | ------------------ |
| BOOLEAN                           | BOOLEAN            |
| TINYINT                           | TINYINT            |
| SMALLINT                          | SMALLINT           |
| INTEGER                           | INTEGER            |
| BIGINT                            | BIGINT             |
| FLOAT                             | FLOAT              |
| DOUBLE                            | DOUBLE             |
| DECIMAL                           | DECIMAL            |
| BINARY                            | INT                |
| CHAR                              | STRING             |
| VARCHAR                           | STRING             |
| STRING                            | STRING             |
| DATE                              | DATE               |
| TIMESTAMP_WITHOUT_TIME_ZONE(N)    | DATETIME           |
| TIMESTAMP_WITH_LOCAL_TIME_ZONE(N) | DATETIME           |
| ARRAY&lt;T&gt;                    | ARRAY&lt;T&gt;     |
| MAP&lt;KT,VT&gt;                  | JSON STRING        |
| ROW&lt;arg T...&gt;               | JSON STRING        |

## 使用説明

### Exactly Once

- sinkがexactly-onceセマンティクスを保証することを希望する場合は、StarRocksを2.5以上にアップグレードし、Flink connectorを1.2.4以上にアップグレードすることをお勧めします。

  - StarRocksのバージョン2.4から、[Stream Loadトランザクションインターフェース](https://docs.starrocks.io/zh-cn/latest/loading/Stream_Load_transaction_interface)がサポートされています。Flink connectorのバージョン1.2.4から、SinkはStream Loadトランザクションインターフェースに基づいてexactly-onceの実装を再設計し、従来のStream Load非トランザクションインターフェースに基づくexactly-onceと比較して、メモリ使用量とチェックポイントの時間を削減し、ジョブのリアルタイム性と安定性を向上させました。
  - Flink connectorのバージョン1.2.4から、StarRocksがStream Loadトランザクションインターフェースをサポートしている場合、SinkはデフォルトでStream Loadトランザクションインターフェースを使用します。Stream Load非トランザクションインターフェースを使用するには、`sink.version`を`V1`に設定する必要があります。
  > **注意**
  >
  > StarRocksまたはFlink connectorのみをアップグレードした場合、sinkは自動的にStream Load非トランザクションインターフェースを選択します。

- sinkがexactly-onceセマンティクスを保証するための設定
  
  - `sink.semantic`の値は`exactly-once`でなければなりません。
  
  - Flink connectorのバージョンが1.2.8以上の場合、`sink.label-prefix`の値を指定することをお勧めします。labelプレフィックスはStarRocksのすべてのタイプのインポートジョブでユニークでなければならず、Flink job、Routine Load、Broker Loadを含みます。

    - labelプレフィックスを指定した場合、Flink connectorはlabelプレフィックスを使用して、Flink jobの失敗により生成された未完了のトランザクションをクリーンアップします。例えば、チェックポイントの進行中にFlink jobが失敗した場合です。`SHOW PROC '/transactions/<db_id>/running';`を使用してこれらのトランザクションのStarRock内での状態を確認すると、トランザクションは通常`PREPARED`状態になっています。Flink jobがチェックポイントから復旧する際、Flink connectorはlabelプレフィックスとチェックポイント内の情報に基づいてこれらの未完了のトランザクションを見つけ出し、トランザクションを中止します。Flink jobが何らかの理由で終了した場合、exactly-onceセマンティクスを実現するために2フェーズコミットメカニズムを採用しているため、Flink connectorはトランザクションを中止することができません。Flink jobが終了した際、Flink connectorはまだFlinkチェックポイントコーディネーターから通知を受けておらず、これらのトランザクションが成功したチェックポイントに含まれるべきかどうかを示していません。これらのトランザクションを中止すると、データが失われる可能性があります。Flinkでエンドツーエンドのexactly-onceを実現する方法については、この[記事](https://flink.apache.org/2018/02/28/an-overview-of-end-to-end-exactly-once-processing-in-apache-flink-with-apache-kafka-too/)を参照してください。
    - labelプレフィックスを指定していない場合、未完了のトランザクションはタイムアウト後にStarRocksによってクリーンアップされます。しかし、Flink jobがトランザクションのタイムアウト前に頻繁に失敗する場合、実行中のトランザクションの数はStarRocksの`max_running_txn_num_per_db`の制限に達する可能性があります。タイムアウトの長さはStarRocks FEの設定`prepared_transaction_default_timeout_second`によって制御され、デフォルト値は`86400`(1日)です。labelプレフィックスを指定していない場合は、より小さい値を設定してトランザクションがより早くタイムアウトするようにすることができます。

- Flink jobが長時間停止した後にチェックポイントまたはセーブポイントを使用して最終的に復旧することが確実である場合、データ損失を避けるために以下のStarRocks設定を調整してください:

  - `prepared_transaction_default_timeout_second`：StarRocks FEのパラメータで、デフォルト値は`86400`です。このパラメータの値はFlink jobの停止時間よりも大きくなければなりません。そうでないと、Flink jobを再起動する前にトランザクションがタイムアウトによって中止され、成功したチェックポイントに含まれている可能性のある未完了のトランザクションがデータ損失を引き起こす可能性があります。
  
    値を大きく設定する場合は、`sink.label-prefix`の値を指定することをお勧めします。そうすると、Flink connectorはlabelプレフィックスとチェックポイント内の情報に基づいて未完了のトランザクションをクリーンアップすることができ、トランザクションがタイムアウトによってStarRocksによってクリーンアップされる（これによってデータ損失が発生する可能性があります）のを避けることができます。

  - `label_keep_max_second`と`label_keep_max_num`：StarRocks FEのパラメータで、デフォルト値はそれぞれ`259200`と`1000`です。詳細は[FE設定](../loading/Loading_intro.md#fe-配置)を参照してください。`label_keep_max_second`の値はFlink jobの停止時間よりも大きくなければなりません。そうでないと、Flink connectorはFlinkのセーブポイントまたはチェックポイントに保存されたトランザクションラベルを使用して、StarRocks内でのトランザクションの状態を確認し、これらのトランザクションがコミットされたかどうかを判断することができず、最終的にデータ損失を引き起こす可能性があります。

  上記の設定は`ADMIN SET FRONTEND CONFIG`を使用して変更することができます。

    ```SQL
    ADMIN SET FRONTEND CONFIG ("prepared_transaction_default_timeout_second" = "3600");
    ADMIN SET FRONTEND CONFIG ("label_keep_max_second" = "259200");
    ADMIN SET FRONTEND CONFIG ("label_keep_max_num" = "1000");
    ```

### Flush戦略

Flink connectorはまずメモリ内でデータをバッファリングし、次にStream Loadを通じて一度にStarRocksにフラッシュします。at-least-onceとexactly-onceのシナリオでは、異なる条件でフラッシュがトリガーされます。

at-least-onceの場合、以下のいずれかの条件を満たしたときにフラッシュがトリガーされます:

- バッファデータのバイト数が`sink.buffer-flush.max-bytes`の制限に達した場合
- バッファデータの行数が`sink.buffer-flush.max-rows`の制限に達した場合（V1バージョンのみ適用）
- 最後のフラッシュから経過した時間が`sink.buffer-flush.interval-ms`の制限に達した場合
- チェックポイントがトリガーされた場合

exactly-onceの場合、チェックポイントがトリガーされたときのみフラッシュがトリガーされます。

### インポート指標の監視

Flink connectorは以下の指標を提供してインポート状況を監視します。

| 指標名                     | タイプ   | 説明                                               |
| ------------------------ | ------ | -------------------------------------------------- |
| totalFlushBytes          | Counter| 成功したフラッシュのバイト数。                                 |
| totalFlushRows           | Counter | 成功したフラッシュの行数。                                   |
| totalFlushSucceededTimes | Counter | データフラッシュの成功回数。                           |
| totalFlushFailedTimes    | Counter | データフラッシュの失敗回数。                                   |
| totalFilteredRows        | Counter | フィルタリングされた行数。この行数はtotalFlushRowsにも含まれます。 |

### Flink CDC同期（スキーマ変更をサポート）

[Flink CDC 3.0フレームワーク](https://github.com/ververica/flink-cdc-connectors/releases)を使用すると、CDCデータソース（MySQL、Kafkaなど）からStarRocksへの[ストリーミングELTパイプライン](https://ververica.github.io/flink-cdc-connectors/master/content/overview/cdc-pipeline.html)を簡単に構築できます。このパイプラインは、データベース全体、シャーディング、およびソースからのスキーマ変更をStarRocksに同期することができます。

バージョン1.2.9以降、StarRocksが提供するFlink connectorはこのフレームワークに統合され、[StarRocks Pipeline Connector](https://ververica.github.io/flink-cdc-connectors/master/content/pipelines/starrocks-pipeline.html)として命名されています。StarRocks Pipeline Connectorは以下をサポートしています:

- データベース/テーブルの自動作成
- スキーマ変更の同期
- 全量および増分データの同期

クイックスタートガイドは[MySQLからStarRocksへのストリーミングELTパイプライン](https://ververica.github.io/flink-cdc-connectors/master/content/quickstart/mysql-starrocks-pipeline-tutorial.html)を参照してください。

## 使用例

### 準備

#### StarRocksテーブルの作成

データベース`test`を作成し、プライマリキーモデルのテーブル`score_board`を作成します。

```SQL
CREATE DATABASE test;

CREATE TABLE test.score_board(
    id int(11) NOT NULL COMMENT "",
    name varchar(65533) NULL DEFAULT "" COMMENT "",
    score int(11) NOT NULL DEFAULT "0" COMMENT ""
)
ENGINE=OLAP
PRIMARY KEY(id)
COMMENT "OLAP"
DISTRIBUTED BY HASH(id) BUCKETS 10;
```
PRIMARY KEY(id)
DISTRIBUTED BY HASH(id);
```

#### Flink 環境

- Flink のバイナリファイル [Flink 1.15.2](https://archive.apache.org/dist/flink/flink-1.15.2/flink-1.15.2-bin-scala_2.12.tgz) をダウンロードし、`flink-1.15.2` ディレクトリに解凍します。
- [Flink connector 1.2.7](https://repo1.maven.org/maven2/com/starrocks/flink-connector-starrocks/1.2.7_flink-1.15/flink-connector-starrocks-1.2.7_flink-1.15.jar) をダウンロードし、`flink-1.15.2/lib` ディレクトリに配置します。
- 以下のコマンドを実行して Flink クラスターを起動します:

    ```Bash
    cd flink-1.15.2
    ./bin/start-cluster.sh
    ```

### Flink SQL を使用してデータを書き込む

- 以下のコマンドを実行して Flink SQL クライアントを起動します。

    ```Bash
    ./bin/sql-client.sh
    ```

- Flink SQL クライアントで、`score_board` テーブルを作成しデータを挿入します。StarRocks の主キーモデルテーブルにデータをインポートする場合、Flink テーブルの DDL で主キーを定義する必要があります。他の StarRocks テーブルタイプでは、これはオプションです。

    ```sql
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

### Flink DataStream を使用してデータを書き込む

入力レコードのタイプに応じて、対応する Flink DataStream ジョブを作成します。例えば、入力レコードが CSV 形式の Java `String`、JSON 形式の Java `String`、またはカスタム Java オブジェクトの場合です。

- 入力レコードが CSV 形式の `String` の場合、対応する Flink DataStream ジョブの主要なコードは以下の通りです。完全なコードは [LoadCsvRecords](https://github.com/StarRocks/starrocks-connector-for-apache-flink/tree/main/examples/src/main/java/com/starrocks/connector/flink/examples/datastream/LoadCsvRecords.java) を参照してください。

    ```Java
    /**
     * CSV 形式のレコードを生成します。各レコードは "\t" で区切られた 3 つの値を持ちます。
     * これらの値は StarRocks テーブルの `id`、`name`、`score` 列にロードされます。
     */
    String[] records = new String[]{
            "1\tstarrocks-csv\t100",
            "2\tflink-csv\t100"
    };
    DataStream<String> source = env.fromElements(records);
    
    /**
     * 必要なプロパティを使用して Flink コネクタを設定します。
     * 入力レコードが CSV 形式であり、列の区切り文字が "\t" であることを Flink コネクタに指示するために、
     * "sink.properties.format" と "sink.properties.column_separator" のプロパティを追加する必要があります。
     * CSV 形式のレコードで他の列区切り文字を使用することも可能ですが、
     * "sink.properties.column_separator" をそれに応じて変更する必要があります。
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

- 入力レコードが JSON 形式の `String` の場合、対応する Flink DataStream ジョブの主要なコードは以下の通りです。完全なコードは [LoadJsonRecords](https://github.com/StarRocks/starrocks-connector-for-apache-flink/tree/main/examples/src/main/java/com/starrocks/connector/flink/examples/datastream/LoadJsonRecords.java) を参照してください。

    ```Java
    /**
     * JSON 形式のレコードを生成します。
     * 各レコードは StarRocks テーブルの列 id、name、score に対応する 3 つのキー値ペアを持ちます。
     */
    String[] records = new String[]{
            "{\"id\":1, \"name\":\"starrocks-json\", \"score\":100}",
            "{\"id\":2, \"name\":\"flink-json\", \"score\":100}",
    };
    DataStream<String> source = env.fromElements(records);
    
    /** 
     * 必要なプロパティを使用して Flink コネクタを設定します。
     * 入力レコードが JSON 形式であり、最も外側の配列構造を取り除くことを Flink コネクタに指示するために、
     * "sink.properties.format" と "sink.properties.strip_outer_array" のプロパティを追加する必要があります。
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

- 入力レコードがカスタム Java オブジェクトの場合、対応する Flink DataStream ジョブの主要なコードは以下の通りです。完全なコードは [LoadCustomJavaRecords](https://github.com/StarRocks/starrocks-connector-for-apache-flink/tree/main/examples/src/main/java/com/starrocks/connector/flink/examples/datastream/LoadCustomJavaRecords.java) を参照してください。

  - この例では、入力レコードはシンプルな POJO `RowData` です。

    ```Java
    public static class RowData {
            public int id;
            public String name;
            public int score;
      
            public RowData() {}
      
            public RowData(int id, String name, int score) {
                this.id = id;
                this.name = name;
                this.score = score;
            }
        }
    ```

  - 主要なコードは以下の通りです：

    ```Java
    // RowData をコンテナとして使用してレコードを生成します。
    RowData[] records = new RowData[]{
            new RowData(1, "starrocks-rowdata", 100),
            new RowData(2, "flink-rowdata", 100),
        };
    DataStream<RowData> source = env.fromElements(records);
    
    // 必要なプロパティを使用して Flink コネクタを設定します。
    StarRocksSinkOptions options = StarRocksSinkOptions.builder()
            .withProperty("jdbc-url", jdbcUrl)
            .withProperty("load-url", loadUrl)
            .withProperty("database-name", "test")
            .withProperty("table-name", "score_board")
            .withProperty("username", "root")
            .withProperty("password", "")
            .build();
    
    /**
     * Flink コネクタは Java オブジェクト配列 (Object[]) を使用して StarRocks テーブルにロードされる行を表します。
     * 各要素は列の値です。
     * StarRocks テーブルのスキーマに一致する Object[] のスキーマを定義する必要があります。
     */
    TableSchema schema = TableSchema.builder()
            .field("id", DataTypes.INT().notNull())
            .field("name", DataTypes.STRING())
            .field("score", DataTypes.INT())
            // StarRocks テーブルがプライマリキーテーブルの場合、プライマリキー `id` には notNull() を指定する必要があります。
            .primaryKey("id")
            .build();
    // スキーマに従って RowData を Object[] に変換します。
    RowDataTransformer transformer = new RowDataTransformer();
    // スキーマ、オプション、トランスフォーマーを使用してシンクを作成します。
    SinkFunction<RowData> starRockSink = StarRocksSink.sink(schema, options, transformer);
    source.addSink(starRockSink);
    ```

  - `RowDataTransformer` は以下のように定義されています：

    ```Java
    private static class RowDataTransformer implements StarRocksSinkRowBuilder<RowData> {
    
        /**
         * 入力された RowData に従ってオブジェクト配列の各要素を設定します。
         * 配列のスキーマは StarRocks テーブルのスキーマと一致します。
         */
        @Override
        public void accept(Object[] internalRow, RowData rowData) {
            internalRow[0] = rowData.id;
            internalRow[1] = rowData.name;
            internalRow[2] = rowData.score;
            // StarRocks テーブルがプライマリキーテーブルの場合、データロードが UPSERT または DELETE 操作であるかを示すために、最後の要素を設定する必要があります。
            internalRow[internalRow.length - 1] = StarRocksSinkOP.UPSERT.ordinal();
        }
    }  
    ```

## ベストプラクティス

### 主キーモデルテーブルへのインポート

このセクションでは、StarRocks の主キーモデルテーブルにデータをインポートして、部分更新と条件更新を実現する方法を示します。以下の例では Flink SQL を使用しています。部分更新と条件更新の詳細については、[データ変更のためのインポート](./Load_to_Primary_Key_tables.md) を参照してください。

#### 準備

StarRocks に `test` データベースを作成し、その中に `score_board` という主キーモデルテーブルを作成します。

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

#### 部分更新

この例では、StarRocks テーブルの `name` 列の値のみを更新するためにデータをインポートする方法を示します。

1. MySQL クライアントを使用して、StarRocks テーブル `score_board` に 2 行のデータを挿入します。

      ```sql
      mysql> INSERT INTO `score_board` VALUES (1, 'starrocks', 100), (2, 'flink', 100);
      
      mysql> SELECT * FROM `score_board`;
      +------+-----------+-------+
      | id   | name      | score |
      +------+-----------+-------+
      |    1 | starrocks |   100 |
      |    2 | flink     |   100 |
      +------+-----------+-------+
      2行がセットされました (0.02秒)
      ```

2. Flink SQLクライアントで `score_board` テーブルを作成します。
   - DDLには `id` と `name` の列定義のみを含みます。
   - `sink.properties.partial_update` オプションを `true` に設定し、Flinkコネクタに部分更新を実行させます。
   - Flinkコネクタのバージョンが1.2.7以下の場合は、`sink.properties.columns` オプションを `id,name,__op` に設定し、更新が必要な列をFlinkコネクタに通知する必要があります。`__op` フィールドを末尾に追加することに注意してください。`__op` フィールドは、インポートがUPSERT操作かDELETE操作かを示し、その値はFlinkコネクタによって自動的に設定されます。

      ```sql
      CREATE TABLE score_board (
          id INT,
          name STRING,
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

3. テーブルに2行のデータを挿入します。データ行の主キーはStarRocksテーブルのデータ行の主キーと同じですが、`name` 列の値が変更されています。

      ```SQL
      INSERT INTO `score_board` VALUES (1, 'starrocks-update'), (2, 'flink-update');
      ```

4. MySQLクライアントでStarRocksテーブルをクエリします。

      ```SQL
      mysql> select * from score_board;
      +------+------------------+-------+
      | id   | name             | score |
      +------+------------------+-------+
      |    1 | starrocks-update |   100 |
      |    2 | flink-update     |   100 |
      +------+------------------+-------+
      2行がセットされました (0.02秒)
      ```

    `name` 列の値のみが変更され、`score` 列の値は変更されていないことがわかります。

#### 条件付き更新

この例では、`score` 列の値に基づいて条件付き更新を行う方法を示します。インポートされたデータ行の `score` 列の値がStarRocksテーブルの現在の値以上の場合にのみ、そのデータ行が更新されます。

1. MySQLクライアントでStarRocksテーブルに2行のデータを挿入します。

    ```SQL
    mysql> INSERT INTO score_board VALUES (1, 'starrocks', 100), (2, 'flink', 100);

    mysql> select * from score_board;
    +------+-----------+-------+
    +------+-----------+-------+
    +------+-----------+-------+
    2行がセットされました (0.02秒)
    ```

2. Flink SQLクライアントで以下のように `score_board` テーブルを作成します:
   - DDLにはすべての列の定義が含まれています。
   - `sink.properties.merge_condition` オプションを `score` に設定し、更新条件として `score` 列を使用するようFlinkコネクタに要求します。
   - `sink.version` オプションを `V1` に設定し、Stream Loadインターフェースを使用してデータをインポートするようFlinkコネクタに要求します。条件付き更新はStream Loadインターフェースでのみサポートされています。

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

3. Flink SQLクライアントでテーブルに2行のデータを挿入します。データ行の主キーはStarRocksテーブルの行と同じです。最初の行のデータは `score` 列でより小さい値を持ち、2番目の行のデータは `score` 列でより大きい値を持ちます。

      ```SQL
      INSERT INTO `score_board` VALUES (1, 'starrocks-update', 99), (2, 'flink-update', 101);
      ```

4. MySQLクライアントでStarRocksテーブルをクエリします。

      ```SQL
      mysql> select * from score_board;
      +------+--------------+-------+
      | id   | name         | score |
      +------+--------------+-------+
      |    1 | starrocks    |   100 |
      |    2 | flink-update |   101 |
      +------+--------------+-------+
      2行がセットされました (0.03秒)
      ```

    2番目の行のデータのみが変更され、最初の行のデータは変更されていないことがわかります。

### Bitmap列へのインポート

`BITMAP`は精密な重複排除カウントを加速するためによく使用されます。例えば、ユニークビジター数(UV)の計算などです。詳細については、[Bitmapを使用した精密な重複排除](../using_starrocks/Using_bitmap.md)を参照してください。

この例では、ユニークビジター数(UV)の計算を例に、StarRocksテーブルの `BITMAP` 列にデータをインポートする方法を示します。

1. MySQLクライアントでStarRocksの集約テーブルを作成します。

   `test`データベースに `page_uv` 集約テーブルを作成し、`visit_users` 列を `BITMAP` 型として定義し、集約関数 `BITMAP_UNION` を設定します。

      ```SQL
      CREATE TABLE `test`.`page_uv` (
        `page_id` INT NOT NULL COMMENT 'ページID',
        `visit_date` datetime NOT NULL COMMENT '訪問日時',
        `visit_users` BITMAP BITMAP_UNION NOT NULL COMMENT 'ユーザーID'
      ) ENGINE=OLAP
      AGGREGATE KEY(`page_id`, `visit_date`)
      DISTRIBUTED BY HASH(`page_id`);
      ```

2. Flink SQLクライアントでテーブルを作成します。

   `visit_user_id` 列が `BIGINT` 型であり、この列のデータをStarRocksテーブルの `BITMAP` 型の `visit_users` 列にインポートしたい場合、DDLの定義時に以下の点に注意する必要があります:

   - Flinkは `BITMAP` 型をサポートしていないため、`visit_user_id` 列を `BIGINT` 型として定義し、StarRocksテーブルの `visit_users` 列を代表させます。
   - `sink.properties.columns` オプションを `page_id,visit_date,visit_user_id,visit_users=to_bitmap(visit_user_id)` に設定し、Flinkコネクタにテーブルの列とStarRocksテーブルの列のマッピング方法を指示します。また、`BIGINT` 型の `visit_user_id` 列のデータを `BITMAP` 型に変換するために `to_bitmap` 関数を使用する必要があります。

      ```SQL
      CREATE TABLE `page_uv` (
          `page_id` INT,
          `visit_date` TIMESTAMP,
          `visit_user_id` BIGINT
      ) WITH (
          'connector' = 'starrocks',
          'jdbc-url' = 'jdbc:mysql://127.0.0.1:9030',
          'load-url' = '127.0.0.1:8030',
          'database-name' = 'test',
          'table-name' = 'page_uv',
          'username' = 'root',
          'password' = '',
          'sink.properties.columns' = 'page_id,visit_date,visit_user_id,visit_users=to_bitmap(visit_user_id)'
      );
      ```

3. Flink SQLクライアントでテーブルにデータを挿入します。

      ```SQL
      INSERT INTO `page_uv` VALUES
         (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 13),
         (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 23),
         (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 33),
         (1, CAST('2020-06-23 02:30:30' AS TIMESTAMP), 13),
         (2, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 23);
      ```

4. MySQLクライアントでStarRocksテーブルをクエリし、ページのUV数を計算します。

      ```SQL
      MySQL [test]> SELECT page_id, COUNT(DISTINCT visit_users) FROM page_uv GROUP BY page_id;
      +---------+-----------------------------+
      +---------+-----------------------------+
      +---------+-----------------------------+
      2行がセットされました (0.05秒)
      ```

### HLL列へのインポート

`HLL`は近似的な重複排除カウントに使用できます。詳細については、[HLLを使用した近似的な重複排除](../using_starrocks/Using_HLL.md)を参照してください。

この例では、ユニークビジター数(UV)の計算を例に、StarRocksテーブルの `HLL` 列にデータをインポートする方法を示します。

1. MySQLクライアントでStarRocksの集約テーブルを作成します。

   `test` データベースに `hll_uv` 集約テーブルを作成し、`visit_users` 列を `HLL` 型として定義し、集約関数 `HLL_UNION` を設定します。

    ```SQL
    CREATE TABLE hll_uv (
    page_id INT NOT NULL COMMENT 'ページID',
    visit_date datetime NOT NULL COMMENT '訪問日時',
    visit_users HLL HLL_UNION NOT NULL COMMENT 'ユーザーID'
    ) ENGINE=OLAP
    AGGREGATE KEY(page_id, visit_date)
    DISTRIBUTED BY HASH(page_id);
    ```

2. Flink SQLクライアントでテーブルを作成します。

   `visit_user_id` 列が `BIGINT` 型であり、この列のデータを `HLL` 型の `visit_users` 列にインポートしたい場合、DDLの定義時に以下の点に注意する必要があります:

    - Flinkは `HLL` 型をサポートしていないため、`visit_user_id` 列を `BIGINT` 型として定義し、StarRocksテーブルの `visit_users` 列を代表させます。
    - `sink.properties.columns` オプションを `page_id,visit_date,visit_user_id,visit_users=hll_hash(visit_user_id)` に設定し、Flinkコネクタにテーブルの列とStarRocksテーブルの列のマッピング方法を指示します。また、`BIGINT` 型の `visit_user_id` 列のデータを `HLL` 型に変換するために `hll_hash` 関数を使用する必要があります。

    ```SQL
    CREATE TABLE hll_uv (
        page_id INT,
        visit_date TIMESTAMP,
        visit_user_id BIGINT
    ) WITH (
        'connector' = 'starrocks',
        'jdbc-url' = 'jdbc:mysql://127.0.0.1:9030',
        'load-url' = '127.0.0.1:8030',
        'database-name' = 'test',
        'table-name' = 'hll_uv',
        'username' = 'root',
        'password' = '',
        'sink.properties.columns' = 'page_id,visit_date,visit_user_id,visit_users=hll_hash(visit_user_id)'
    );
    ```

3. Flink SQLクライアントでテーブルにデータを挿入します。

    ```SQL
    INSERT INTO hll_uv VALUES
    ```SQL
    INSERT INTO hll_uv VALUES
    (3, CAST('2023-07-24 12:00:00' AS TIMESTAMP), 78),
    (4, CAST('2023-07-24 13:20:10' AS TIMESTAMP), 2),
    (3, CAST('2023-07-24 12:30:00' AS TIMESTAMP), 674);
    ```

4. MySQLクライアントでStarRocksテーブルをクエリして、ページのUV数を計算します。

    ```SQL
    mysql> SELECT `page_id`, COUNT(DISTINCT `visit_users`) FROM `hll_uv` GROUP BY `page_id`;
    **+---------+-----------------------------+
    | page_id | count(DISTINCT visit_users) |
    +---------+-----------------------------+
    |       3 |                           2 |
    |       4 |                           1 |
    +---------+-----------------------------+
    2 rows in set (0.04 sec)
    ```

