---
displayed_sidebar: English
---

# Apache Flink® からデータを継続的にロードする

StarRocksは、Flinkを使用してStarRocksテーブルにデータをロードするための自社開発のコネクタであるFlinkコネクタ（以下、Flinkコネクタ）を提供しています。基本原理は、データを蓄積してから、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を通じて一度にStarRocksにロードすることです。

FlinkコネクタはDataStream API、Table API & SQL、Python APIをサポートしており、Apache Flink®が提供する[flink-connector-jdbc](https://nightlies.apache.org/flink/flink-docs-master/docs/connectors/table/jdbc/)よりも高性能で安定しています。

> **注意**
>
> Flinkコネクタを使用してStarRocksテーブルにデータをロードするにはSELECTとINSERTの権限が必要です。これらの権限がない場合は、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)に記載されている手順に従って、StarRocksクラスタに接続するユーザーにこれらの権限を付与してください。

## バージョン要件

| コネクタ | Flink                    | StarRocks     | Java | Scala     |
|-----------|--------------------------|---------------| ---- |-----------|
| 1.2.9 | 1.15, 1.16, 1.17, 1.18 | 2.1 以降 | 8 | 2.11, 2.12 |
| 1.2.8     | 1.13, 1.14, 1.15, 1.16, 1.17 | 2.1 以降 | 8    | 2.11, 2.12 |
| 1.2.7     | 1.11, 1.12, 1.13, 1.14, 1.15 | 2.1 以降 | 8    | 2.11, 2.12 |

## Flinkコネクタの取得

FlinkコネクタのJARファイルは、以下の方法で取得できます：

- コンパイル済みのFlinkコネクタJARファイルを直接ダウンロードする。
- MavenプロジェクトにFlinkコネクタを依存関係として追加し、JARファイルをダウンロードする。
- Flinkコネクタのソースコードを自分でJARファイルにコンパイルする。

FlinkコネクタJARファイルの命名フォーマットは以下の通りです：

- Flink 1.15以降は、`flink-connector-starrocks-${connector_version}_flink-${flink_version}.jar`です。例えば、Flink 1.15をインストールし、Flinkコネクタ1.2.7を使用したい場合は、`flink-connector-starrocks-1.2.7_flink-1.15.jar`を使用できます。

- Flink 1.15より前では、`flink-connector-starrocks-${connector_version}_flink-${flink_version}_${scala_version}.jar`です。例えば、環境にFlink 1.14とScala 2.12をインストールし、Flinkコネクタ1.2.7を使用したい場合は、`flink-connector-starrocks-1.2.7_flink-1.14_2.12.jar`を使用できます。

> **注意**
>
> 一般的に、Flinkコネクタの最新バージョンは、Flinkの最新の3つのバージョンのみと互換性を維持します。

### コンパイル済みJarファイルをダウンロードする

[Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks)から対応するバージョンのFlinkコネクタJarファイルを直接ダウンロードします。

### Maven依存関係

Mavenプロジェクトの`pom.xml`ファイルに、以下のフォーマットに従ってFlinkコネクタを依存関係として追加します。`flink_version`、`scala_version`、`connector_version`をそれぞれのバージョンに置き換えてください。

- Flink 1.15以降の場合

    ```xml
    <dependency>
        <groupId>com.starrocks</groupId>
        <artifactId>flink-connector-starrocks</artifactId>
        <version>${connector_version}_flink-${flink_version}</version>
    </dependency>
    ```

- Flink 1.15より前の場合

    ```xml
    <dependency>
        <groupId>com.starrocks</groupId>
        <artifactId>flink-connector-starrocks</artifactId>
        <version>${connector_version}_flink-${flink_version}_${scala_version}</version>
    </dependency>
    ```

### 自分でコンパイルする

1. [Flinkコネクタパッケージ](https://github.com/StarRocks/starrocks-connector-for-apache-flink)をダウンロードします。
2. 次のコマンドを実行してFlinkコネクタのソースコードをJARファイルにコンパイルします。`flink_version`は対応するFlinkバージョンに置き換えてください。

      ```bash
      sh build.sh <flink_version>
      ```

   例えば、環境のFlinkバージョンが1.15の場合、次のコマンドを実行します：

      ```bash
      sh build.sh 1.15
      ```

3. `target/`ディレクトリに移動して、コンパイルによって生成されたFlinkコネクタJARファイル（例：`flink-connector-starrocks-1.2.7_flink-1.15-SNAPSHOT.jar`）を探します。

> **注記**
>
> 正式にリリースされていないFlinkコネクタの名前には`SNAPSHOT`接尾辞が含まれます。

## オプション

| **オプション**                        | **必須** | **デフォルト値** | **説明**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|-----------------------------------|--------------|-------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| connector                         | はい          | なし              | 使用するコネクタ。値は"starrocks"である必要があります。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| jdbc-url                          | はい          | なし              | FEのMySQLサーバーに接続するためのアドレス。複数のアドレスを指定することができ、コンマ(,)で区切る必要があります。フォーマット：`jdbc:mysql://<fe_host1>:<fe_query_port1>,<fe_host2>:<fe_query_port2>,<fe_host3>:<fe_query_port3>`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| load-url                          | はい          | なし              | StarRocksクラスタのFEのHTTP URL。複数のURLを指定することができ、セミコロン(;)で区切る必要があります。フォーマット：`<fe_host1>:<fe_http_port1>;<fe_host2>:<fe_http_port2>`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| database-name                     | はい          | なし              | データをロードするStarRocksデータベースの名前。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| table-name                        | はい          | なし              | StarRocksにデータをロードするために使用するテーブルの名前。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| username                          | はい          | なし              | StarRocksにデータをロードするために使用するアカウントのユーザー名。アカウントには[SELECTとINSERTの権限](../sql-reference/sql-statements/account-management/GRANT.md)が必要です。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| password                          | はい          | なし              | 上記アカウントのパスワード。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| sink.semantic                     | いいえ           | at-least-once     | sinkが保証するセマンティクス。有効な値は**at-least-once**と**exactly-once**です。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| sink.version                      | いいえ           | AUTO              | データをロードするために使用されるインターフェース。このパラメータはFlinkコネクタバージョン1.2.4以降でサポートされています。<ul><li>`V1`：[Stream Load](../loading/StreamLoad.md)インターフェースを使用してデータをロードします。1.2.4以前のコネクタはこのモードのみをサポートします。</li><li>`V2`：[Stream Loadトランザクション](../loading/Stream_Load_transaction_interface.md)インターフェースを使用してデータをロードします。StarRocksが少なくともバージョン2.4であることが必要です。メモリ使用量を最適化し、より安定したexactly-once実装を提供するために`V2`を推奨します。</li><li>`AUTO`：StarRocksのバージョンがトランザクションStream Loadをサポートしている場合は`V2`を自動的に選択し、そうでない場合は`V1`を選択します。</li></ul> |
| sink.label-prefix                 | いいえ           | なし              | Stream Loadに使用されるラベルの接頭辞。コネクタ1.2.8以降でexactly-onceを使用する場合は、設定することを推奨します。[exactly-onceの使用上の注意](#exactly-once)を参照してください。 |

| sink.buffer-flush.max-bytes       | いいえ           | 94371840(90M)     | 一度にStarRocksに送信される前にメモリに蓄積できるデータの最大サイズ。最大値の範囲は64MBから10GBです。このパラメータを大きな値に設定すると、ロードパフォーマンスは向上しますが、ロードレイテンシーが増加する可能性があります。                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| sink.buffer-flush.max-rows        | いいえ           | 500000            | 一度にStarRocksに送信される前にメモリに蓄積できる最大行数。このパラメータは、`sink.version`を`V1`に設定し、`sink.semantic`を`at-least-once`に設定した場合にのみ使用可能です。有効な値: 64000から5000000。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| sink.buffer-flush.interval-ms     | いいえ           | 300000            | データがフラッシュされる間隔。このパラメータは、`sink.semantic`を`at-least-once`に設定した場合にのみ使用可能です。有効な値: 1000から3600000。単位: ms。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| sink.max-retries                  | いいえ           | 3                 | システムがストリームロードジョブを実行するための再試行回数。このパラメータは、`sink.version`を`V1`に設定した場合にのみ使用可能です。有効な値: 0から10。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| sink.connect.timeout-ms           | いいえ           | 30000             | HTTP接続の確立タイムアウト。有効な値: 100から60000。単位: ms。Flinkコネクタv1.2.9以前のバージョンでは、デフォルト値は`1000`です。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| sink.wait-for-continue.timeout-ms | いいえ           | 10000             | 1.2.7以降でサポート。FEからのHTTP 100-continueの応答を待つタイムアウト。有効な値: `3000`から`60000`。単位: ms。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| sink.ignore.update-before         | いいえ           | true              | バージョン1.2.8以降でサポート。Flinkからの`UPDATE_BEFORE`レコードをプライマリキーテーブルへのデータロード時に無視するかどうか。このパラメータをfalseに設定すると、レコードはStarRocksテーブルへの削除操作として扱われます。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| sink.properties.*                 | いいえ           | NONE              | ストリームロード動作を制御するために使用されるパラメータ。例えば、`sink.properties.format`パラメータはストリームロードに使用される形式を指定します。サポートされるパラメータとその説明のリストについては、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。                                                                                                                                                                                                                                                                                                                                 |
| sink.properties.format            | いいえ           | csv               | ストリームロードに使用される形式。Flinkコネクタは、データの各バッチをStarRocksに送信する前に、指定された形式に変換します。有効な値: `csv`と`json`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| sink.properties.row_delimiter     | いいえ           | \n                | CSV形式のデータの行区切り文字。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| sink.properties.column_separator  | いいえ           | \t                | CSV形式のデータの列区切り文字。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| sink.properties.max_filter_ratio  | いいえ           | 0                 | ストリームロードの最大エラー許容率。データ品質が不十分によりフィルタリングされるデータレコードの最大割合です。有効な値: `0`から`1`。デフォルト値: `0`。詳細は[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照。                                                                                                                                                                                                                                                                                                                                                                      |
| sink.parallelism                  | いいえ           | NONE              | コネクタの並列度。Flink SQLでのみ利用可能です。設定されていない場合、Flinkプランナーが並列度を決定します。複数の並列度のシナリオでは、ユーザーはデータが正しい順序で書き込まれることを保証する必要があります。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| sink.properties.strict_mode       | いいえ           | false             | ストリームロードで厳格モードを有効にするかどうかを指定します。列値の不一致など、資格のない行がある場合のロード動作に影響します。有効な値: `true`と`false`。デフォルト値: `false`。詳細は[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照。 |

## FlinkとStarRocks間のデータ型マッピング

| Flinkデータ型                     | StarRocksデータ型   |
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
| ARRAY&lt;T&gt;                    | ARRAY&lt;T&gt;        |
| MAP&lt;KT,VT&gt;                  | JSON STRING           |
| ROW&lt;arg T...&gt;               | JSON STRING           |

## 使用上の注意

### Exactly Once

- sinkでexactly-onceセマンティクスを保証したい場合は、StarRocksを2.5以降に、Flinkコネクタを1.2.4以降にアップグレードすることを推奨します
  - Flinkコネクタ1.2.4以降、exactly-onceは[Stream Loadトランザクションインターフェース](https://docs.starrocks.io/en-us/latest/loading/Stream_Load_transaction_interface)に基づいて再設計されました
    これは2.4以降のStarRocksによって提供されます。非トランザクショナルなStream Loadインターフェースに基づく以前の実装と比較して、
    新しい実装はメモリ使用量とチェックポイントのオーバーヘッドを削減し、リアルタイムパフォーマンスと
    ロードの安定性を向上させます。

  - StarRocksのバージョンが2.4より前の場合、またはFlinkコネクタのバージョンが1.2.4より前の場合、sinkは
    非トランザクショナルなStream Loadインターフェースに基づいた実装を自動的に選択します。

- exactly-onceを保証するための設定

  - `sink.semantic`の値は`exactly-once`である必要があります。

  - Flinkコネクタのバージョンが1.2.8以降の場合、`sink.label-prefix`の値を指定することを推奨します。ラベルプレフィックスは、Flinkジョブ、ルーチンロード、ブローカーロードなど、StarRocksのすべてのロードタイプでユニークである必要があります。

    - ラベルプレフィックスが指定されている場合、Flinkコネクタはラベルプレフィックスを使用して、Flinkの障害シナリオで生成される可能性のある残留トランザクションをクリーンアップします
      例えば、チェックポイントが進行中のときにFlinkジョブが失敗した場合です。これらの残留トランザクションは、
      StarRocksで`SHOW PROC '/transactions/<db_id>/running';`を使用して表示すると、通常`PREPARED`ステータスになります。Flinkジョブがチェックポイントから復元されると、
      Flinkコネクタはラベルプレフィックスとチェックポイントの情報を基にこれらの残留トランザクションを見つけて中止します。Flinkコネクタは、2フェーズコミットメカニズムを使用してexactly-onceを実装するため、Flinkジョブが終了したときにそれらを中止することはできません。Flinkジョブが終了すると、Flinkコネクタは通知を受け取っていません
      Flinkのチェックポイントコーディネータは、トランザクションが成功したチェックポイントに含まれるべきかどうかを判断しますが、これらのトランザクションが中止された場合、データ損失が発生する可能性があります。Flinkでエンドツーエンドの完全一致を実現する方法については、この[ブログ投稿](https://flink.apache.org/2018/02/28/an-overview-of-end-to-end-exactly-once-processing-in-apache-flink-with-apache-kafka-too/)をご覧ください。

    - ラベルプレフィックスが指定されていない場合、トランザクションはStarRocksによってタイムアウト後にのみクリーンアップされます。しかし、Flinkジョブがトランザクションがタイムアウトする前に頻繁に失敗すると、StarRocksの`max_running_txn_num_per_db`の制限に達する可能性があります。タイムアウトの長さはStarRocks FEの設定`prepared_transaction_default_timeout_second`によって制御され、デフォルト値は`86400`（1日）です。ラベルプレフィックスが指定されていない場合、トランザクションがより早く期限切れになるように、より小さい値を設定することができます。

- Flinkジョブが長いダウンタイムの後にチェックポイントまたはセーブポイントから回復することが確実である場合（停止または連続的なフェイルオーバーによる）、データ損失を避けるために以下のStarRocks設定を適切に調整してください。

  - `prepared_transaction_default_timeout_second`: StarRocks FEの設定で、デフォルト値は`86400`です。この設定の値はFlinkジョブのダウンタイムよりも長くする必要があります。そうでないと、成功したチェックポイントに含まれている保留中のトランザクションが、Flinkジョブを再開する前にタイムアウトによって中止され、データ損失につながる可能性があります。

    この設定に大きな値を設定する場合は、`sink.label-prefix`の値を指定することが望ましいです。そうすることで、保留中のトランザクションはラベルプレフィックスとチェックポイントの情報に基づいてクリーンアップされ、タイムアウトによるデータ損失のリスクを減らすことができます。

  - `label_keep_max_second`と`label_keep_max_num`: StarRocks FEの設定で、それぞれのデフォルト値は`259200`と`1000`です。詳細は[FE設定](../loading/Loading_intro.md#fe-configurations)を参照してください。`label_keep_max_second`の値はFlinkジョブのダウンタイムよりも長くする必要があります。そうでないと、FlinkコネクタはFlinkのセーブポイントまたはチェックポイントに保存されたトランザクションラベルを使用してStarRocksのトランザクションの状態を確認できず、これらのトランザクションがコミットされたかどうかを判断できなくなり、結果的にデータ損失につながる可能性があります。

  これらの設定は変更可能で、`ADMIN SET FRONTEND CONFIG`を使用して変更できます。

  ```SQL
    ADMIN SET FRONTEND CONFIG ("prepared_transaction_default_timeout_second" = "3600");
    ADMIN SET FRONTEND CONFIG ("label_keep_max_second" = "259200");
    ADMIN SET FRONTEND CONFIG ("label_keep_max_num" = "1000");
  ```

### フラッシュポリシー

Flinkコネクタはデータをメモリにバッファリングし、Stream Loadを介してStarRocksにバッチでフラッシュします。フラッシュがトリガーされる条件は、at-least-onceとexactly-onceで異なります。

at-least-onceの場合、以下のいずれかの条件が満たされた場合にフラッシュがトリガーされます：

- バッファリングされた行のバイト数が`sink.buffer-flush.max-bytes`の制限に達した場合
- バッファリングされた行数が`sink.buffer-flush.max-rows`の制限に達した場合（シンクバージョンV1のみ有効）
- 最後のフラッシュからの経過時間が`sink.buffer-flush.interval-ms`の制限に達した場合
- チェックポイントがトリガーされた場合

exactly-onceの場合、フラッシュはチェックポイントがトリガーされたときにのみ発生します。

### ロードメトリクスの監視

Flinkコネクタは、ロードを監視するための以下のメトリクスを提供します。

| メトリック                     | タイプ    | 説明                                                     |
|--------------------------|---------|-----------------------------------------------------------------|
| totalFlushBytes          | カウンタ | 正常にフラッシュされたバイト数。                                     |
| totalFlushRows           | カウンタ | 正常にフラッシュされた行数。                                      |
| totalFlushSucceededTimes | カウンタ | データが正常にフラッシュされた回数。  |
| totalFlushFailedTimes    | カウンタ | データのフラッシュに失敗した回数。                  |
| totalFilteredRows        | カウンタ | フィルタリングされた行数（totalFlushRowsにも含まれます）。    |

### Flink CDC同期（スキーマ変更対応）

[Flink CDC 3.0](https://github.com/ververica/flink-cdc-connectors/releases)フレームワークを使用して、MySQLやKafkaなどのCDCソースからStarRocksへのストリーミングELTパイプラインを簡単に[構築できます](https://ververica.github.io/flink-cdc-connectors/master/content/overview/cdc-pipeline.html)。このパイプラインは、ソースからStarRocksへのデータベース全体、マージされたシャーディングテーブル、およびスキーマ変更を同期できます。

v1.2.9以降、StarRocks用のFlinkコネクタは、[StarRocks Pipeline Connector](https://ververica.github.io/flink-cdc-connectors/master/content/pipelines/starrocks-pipeline.html)としてこのフレームワークに統合されています。StarRocks Pipeline Connectorは以下をサポートしています：

- データベースとテーブルの自動作成
- スキーマ変更の同期
- フルおよびインクリメンタルデータ同期

クイックスタートは、[Flink CDC 3.0とStarRocks Pipeline Connectorを使用したMySQLからStarRocksへのストリーミングELT](https://ververica.github.io/flink-cdc-connectors/master/content/quickstart/mysql-starrocks-pipeline-tutorial.html)を参照してください。

## 例

以下の例は、Flink SQLまたはFlink DataStreamを使用してStarRocksテーブルにデータをロードする方法を示しています。

### 準備

#### StarRocksテーブルの作成

`test`データベースを作成し、`score_board`というプライマリキーテーブルを作成します。

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

- Flinkバイナリ[Flink 1.15.2](https://archive.apache.org/dist/flink/flink-1.15.2/flink-1.15.2-bin-scala_2.12.tgz)をダウンロードし、`flink-1.15.2`ディレクトリに解凍します。
- [Flinkコネクタ1.2.7](https://repo1.maven.org/maven2/com/starrocks/flink-connector-starrocks/1.2.7_flink-1.15/flink-connector-starrocks-1.2.7_flink-1.15.jar)をダウンロードし、`flink-1.15.2/lib`ディレクトリに配置します。
- 以下のコマンドを実行してFlinkクラスターを起動します。

    ```shell
    cd flink-1.15.2
    ./bin/start-cluster.sh
    ```

### Flink SQLで実行

- 以下のコマンドを実行してFlink SQLクライアントを起動します。

    ```shell
    ./bin/sql-client.sh
    ```

- Flinkテーブル`score_board`を作成し、Flink SQLクライアントを使用してテーブルに値を挿入します。
StarRocksのプライマリキーテーブルにデータをロードする場合は、Flink DDLでプライマリキーを定義する必要があります。他のタイプのStarRocksテーブルでは省略可能です。

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

### Flink DataStreamでの実行

入力レコードのタイプに応じてFlink DataStreamジョブを実装する方法はいくつかあります。例えば、CSV形式のJava `String`、JSON形式のJava `String`、またはカスタムJavaオブジェクトなどです。

- 入力レコードがCSV形式の`String`です。完全な例については、[LoadCsvRecords](https://github.com/StarRocks/starrocks-connector-for-apache-flink/tree/main/examples/src/main/java/com/starrocks/connector/flink/examples/datastream/LoadCsvRecords.java)を参照してください。

    ```java
    /**
     * CSV形式のレコードを生成します。各レコードは"\t"で区切られた3つの値を持っています。
     * これらの値はStarRocksテーブルの`id`、`name`、`score`の各カラムにロードされます。
     */
    String[] records = new String[]{
            "1\tstarrocks-csv\t100",
            "2\tflink-csv\t100"
    };
    DataStream<String> source = env.fromElements(records);

    /**
     * 必要なプロパティでコネクタを設定します。
     * また、「sink.properties.format」と「sink.properties.column_separator」のプロパティを追加して、
     * 入力レコードがCSV形式であり、カラムの区切り文字が"\t"であることをコネクタに伝える必要があります。
     * CSV形式のレコードで他のカラム区切り文字を使用することもできますが、
     * 「sink.properties.column_separator」を対応するように変更することを忘れないでください。
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

- 入力レコードがJSON形式の`String`です。完全な例については、[LoadJsonRecords](https://github.com/StarRocks/starrocks-connector-for-apache-flink/tree/main/examples/src/main/java/com/starrocks/connector/flink/examples/datastream/LoadJsonRecords.java)を参照してください。

    ```java
    /**
     * JSON形式のレコードを生成します。
     * 各レコードはStarRocksテーブルの`id`、`name`、`score`の各カラムに対応する3つのキー値ペアを持っています。
     */
    String[] records = new String[]{
            "{\"id\":1, \"name\":\"starrocks-json\", \"score\":100}",
            "{\"id\":2, \"name\":\"flink-json\", \"score\":100}",
    };
    DataStream<String> source = env.fromElements(records);

    /** 
     * 必要なプロパティでコネクタを設定します。
     * また、「sink.properties.format」と「sink.properties.strip_outer_array」のプロパティを追加して、
     * 入力レコードがJSON形式であり、最も外側の配列構造を取り除くことをコネクタに伝える必要があります。
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

- 入力レコードがカスタムJavaオブジェクトです。完全な例については、[LoadCustomJavaRecords](https://github.com/StarRocks/starrocks-connector-for-apache-flink/tree/main/examples/src/main/java/com/starrocks/connector/flink/examples/datastream/LoadCustomJavaRecords.java)を参照してください。

  - この例では、入力レコードはシンプルなPOJO `RowData`です。

      ```java
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

  - メインプログラムは以下の通りです：

    ```java
    // RowDataをコンテナとして使用するレコードを生成します。
    RowData[] records = new RowData[]{
            new RowData(1, "starrocks-rowdata", 100),
            new RowData(2, "flink-rowdata", 100),
        };
    DataStream<RowData> source = env.fromElements(records);

    // 必要なプロパティでコネクタを設定します。
    StarRocksSinkOptions options = StarRocksSinkOptions.builder()
            .withProperty("jdbc-url", jdbcUrl)
            .withProperty("load-url", loadUrl)
            .withProperty("database-name", "test")
            .withProperty("table-name", "score_board")
            .withProperty("username", "root")
            .withProperty("password", "")
            .build();

    /**
     * FlinkコネクタはJavaオブジェクト配列(Object[])を使用して、StarRocksテーブルにロードされる行を表します。
     * そして、各要素はカラムの値です。
     * StarRocksテーブルのスキーマに一致するObject[]のスキーマを定義する必要があります。
     */
    TableSchema schema = TableSchema.builder()
            .field("id", DataTypes.INT().notNull())
            .field("name", DataTypes.STRING())
            .field("score", DataTypes.INT())
            // StarRocksテーブルがプライマリキーテーブルの場合、プライマリキー`id`にはnotNull()を指定する必要があります（例：DataTypes.INT().notNull()）。
            .primaryKey("id")
            .build();
    // スキーマに従ってRowDataをObject[]に変換します。
    RowDataTransformer transformer = new RowDataTransformer();
    // スキーマ、オプション、トランスフォーマーを使用してシンクを作成します。
    SinkFunction<RowData> starRockSink = StarRocksSink.sink(schema, options, transformer);
    source.addSink(starRockSink);
    ```

  - メインプログラムで定義された`RowDataTransformer`は以下の通りです：

    ```java
    private static class RowDataTransformer implements StarRocksSinkRowBuilder<RowData> {
    
        /**
         * 入力されたRowDataに従ってオブジェクト配列の各要素を設定します。
         * 配列のスキーマはStarRocksテーブルのそれと一致します。
         */
        @Override
        public void accept(Object[] internalRow, RowData rowData) {
            internalRow[0] = rowData.id;
            internalRow[1] = rowData.name;
            internalRow[2] = rowData.score;
            // StarRocksテーブルがプライマリキーテーブルの場合、データロードがUPSERTまたはDELETE操作であるかを示すために、最後の要素を設定する必要があります。
            internalRow[internalRow.length - 1] = StarRocksSinkOP.UPSERT.ordinal();
        }
    }  
    ```

## ベストプラクティス

### プライマリキーテーブルへのデータロード

このセクションでは、StarRocksのプライマリキーテーブルにデータをロードして、部分更新と条件付き更新を実現する方法について説明します。
これらの機能の紹介については、[変更データのロード](https://docs.starrocks.io/en-us/latest/loading/Load_to_Primary_Key_tables)をご覧ください。
これらの例ではFlink SQLを使用します。

#### 準備

StarRocksにデータベース`test`を作成し、プライマリキーテーブル`score_board`を作成します。

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

この例では、`id`と`name`のカラムのみにデータをロードする方法を示します。

1. MySQLクライアントでStarRocksテーブル`score_board`に2行のデータを挿入します。

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

2. Flink SQLクライアントでFlinkテーブル`score_board`を作成します。

   - `id`と`name`のカラムのみを含むDDLを定義します。
   - Flinkコネクタが部分更新を行うように指示するために、オプション`sink.properties.partial_update`を`true`に設定します。
   - Flinkコネクタのバージョンが1.2.7以下の場合、更新する必要があるカラムをFlinkコネクタに指示するために、オプション`sink.properties.columns`を`id,name,__op`に設定する必要があります。フィールド`__op`を最後に追加する必要があることに注意してください。このフィールド`__op`は、データロードがUPSERTまたはDELETE操作であることを示し、その値はコネクタによって自動的に設定されます。

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

3. Flinkテーブルに2つのデータ行を挿入します。データ行の主キーはStarRocksテーブルの行の主キーと同じですが、`name`列の値は変更されます。

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
    2 rows in set (0.02 sec)
    ```

   `name`の値のみが変更され、`score`の値は変更されないことがわかります。

#### 条件付き更新

この例では、`score`列の値に応じた条件付き更新を行う方法を示します。`id`の更新は、新しい`score`の値が古い値以上の場合にのみ有効になります。

1. MySQLクライアントでStarRocksテーブルに2つのデータ行を挿入します。

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

2. 次の方法でFlinkテーブル`score_board`を作成します。
  
    - すべての列を含むDDLを定義します。
    - `sink.properties.merge_condition`オプションを`score`に設定して、コネクタに条件として`score`列を使用するよう指示します。
    - `sink.version`オプションを`V1`に設定して、コネクタにStream Loadを使用するよう指示します。

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

3. Flinkテーブルに2つのデータ行を挿入します。データ行の主キーはStarRocksテーブルの行の主キーと同じです。最初のデータ行の`score`列の値は小さく、2番目のデータ行の`score`列の値は大きくなります。

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
    2 rows in set (0.03 sec)
    ```

   2番目のデータ行のみの値が変更され、最初のデータ行の値は変更されません。

### BITMAP型のカラムにデータをロードする

[`BITMAP`](https://docs.starrocks.io/en-us/latest/sql-reference/sql-statements/data-types/BITMAP)は、UVカウントなどの個別カウントを高速化するためによく使用されます。詳細は[ビットマップを使用して正確なCount Distinctを行う](https://docs.starrocks.io/en-us/latest/using_starrocks/Using_bitmap)を参照してください。
ここでは、UVカウントを例にして、`BITMAP`型のカラムにデータをロードする方法を示します。

1. MySQLクライアントでStarRocks集計テーブルを作成します。

   `test`データベースで、`visit_users`列が`BITMAP`型であり、集計関数`BITMAP_UNION`で構成される集計テーブル`page_uv`を作成します。

    ```SQL
    CREATE TABLE `test`.`page_uv` (
      `page_id` INT NOT NULL COMMENT 'ページID',
      `visit_date` datetime NOT NULL COMMENT '訪問日時',
      `visit_users` BITMAP BITMAP_UNION NOT NULL COMMENT '訪問ユーザーID'
    ) ENGINE=OLAP
    AGGREGATE KEY(`page_id`, `visit_date`)
    DISTRIBUTED BY HASH(`page_id`);
    ```

2. Flink SQLクライアントでFlinkテーブルを作成します。

    Flinkテーブルの`visit_user_id`列は`BIGINT`型で、この列をStarRocksテーブルの`BITMAP`型の`visit_users`列にロードしたい場合、FlinkテーブルのDDLを定義する際に以下の点に注意してください：
    - Flinkは`BITMAP`型をサポートしていないため、`visit_user_id`列を`BIGINT`型として定義し、StarRocksテーブルの`BITMAP`型の`visit_users`列を表す必要があります。
    - `sink.properties.columns`オプションを`page_id,visit_date,visit_user_id,visit_users=to_bitmap(visit_user_id)`に設定し、FlinkテーブルとStarRocksテーブル間のカラムマッピングをコネクタに指示します。また、[`to_bitmap`](https://docs.starrocks.io/en-us/latest/sql-reference/sql-functions/bitmap-functions/to_bitmap)関数を使用して、`BIGINT`型のデータを`BITMAP`型に変換するようコネクタに指示します。

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

3. Flink SQLクライアントでFlinkテーブルにデータをロードします。

    ```SQL
    INSERT INTO `page_uv` VALUES
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 13),
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 23),
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 33),
       (1, CAST('2020-06-23 02:30:30' AS TIMESTAMP), 13),
       (2, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 23);
    ```

4. MySQLクライアントでStarRocksテーブルからページUVを計算します。

    ```SQL
    MySQL [test]> SELECT `page_id`, COUNT(DISTINCT `visit_users`) FROM `page_uv` GROUP BY `page_id`;
    +---------+-----------------------------+
    | page_id | COUNT(DISTINCT `visit_users`) |
    +---------+-----------------------------+
    |       2 |                           1 |
    |       1 |                           3 |
    +---------+-----------------------------+
    2 rows in set (0.05 sec)
    ```

### HLL型のカラムにデータをロードする

[`HLL`](../sql-reference/sql-statements/data-types/HLL.md)は、近似的な個別カウントに使用できます。詳細は[HLLを使用した近似的なCount Distinct](../using_starrocks/Using_HLL.md)を参照してください。

ここでは、UVカウントを例にして、`HLL`型のカラムにデータをロードする方法を示します。

1. StarRocks集計テーブルを作成します

   `test`データベースで、`visit_users`列が`HLL`型であり、集計関数`HLL_UNION`で構成される集計テーブル`hll_uv`を作成します。

    ```SQL
    CREATE TABLE `hll_uv` (
      `page_id` INT NOT NULL COMMENT 'ページID',
      `visit_date` datetime NOT NULL COMMENT '訪問日時',
      `visit_users` HLL HLL_UNION NOT NULL COMMENT '訪問ユーザーID'
    ) ENGINE=OLAP
    AGGREGATE KEY(`page_id`, `visit_date`)
    DISTRIBUTED BY HASH(`page_id`);
    ```

2. Flink SQLクライアントでFlinkテーブルを作成します。

    Flinkテーブルの`visit_user_id`列は`BIGINT`型で、この列をStarRocksテーブルの`HLL`型の`visit_users`列にロードしたい場合、FlinkテーブルのDDLを定義する際に以下の点に注意してください：
    - Flinkは`HLL`型をサポートしていないため、`visit_user_id`列を`BIGINT`型として定義し、StarRocksテーブルの`HLL`型の`visit_users`列を表す必要があります。
    - `sink.properties.columns`オプションを`page_id,visit_date,user_id,visit_users=hll_hash(visit_user_id)`に設定し、FlinkテーブルとStarRocksテーブル間のカラムマッピングをコネクタに指示します。また、[`hll_hash`](../sql-reference/sql-functions/aggregate-functions/hll_hash.md)関数を使用して、`BIGINT`型のデータを`HLL`型に変換するようコネクタに指示します。

    ```SQL
    CREATE TABLE `hll_uv` (
        `page_id` INT,
        `visit_date` TIMESTAMP,
        `visit_user_id` BIGINT
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

3. Flink SQLクライアントでFlinkテーブルにデータをロードします。

    ```SQL
    INSERT INTO `hll_uv` VALUES
       (3, CAST('2023-07-24 12:00:00' AS TIMESTAMP), 78),
       (4, CAST('2023-07-24 13:20:10' AS TIMESTAMP), 2),
       (3, CAST('2023-07-24 12:30:00' AS TIMESTAMP), 674);
    ```

4. MySQLクライアントでStarRocksテーブルからページのUVを計算します。

    ```SQL
    mysql> SELECT `page_id`, COUNT(DISTINCT `visit_users`) FROM `hll_uv` GROUP BY `page_id`;
    **+---------+-----------------------------+
    | page_id | COUNT(DISTINCT visit_users) |
    +---------+-----------------------------+
    |       3 |                           2 |
    |       4 |                           1 |
    +---------+-----------------------------+
    2 rows in set (0.04 sec)
    ```

