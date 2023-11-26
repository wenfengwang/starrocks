---
displayed_sidebar: "日本語"
---

# Apache Flink®からデータを連続的にロードする

StarRocksは、Apache Flink®（以下、Flinkコネクタ）用の自社開発コネクタを提供しており、Flinkを使用してデータをStarRocksテーブルにロードするためのサポートを行っています。基本的な原則は、データを蓄積し、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を介して一度にすべてのデータをStarRocksにロードすることです。

Flinkコネクタは、DataStream API、Table API＆SQL、およびPython APIをサポートしています。Apache Flink®が提供する[flink-connector-jdbc](https://nightlies.apache.org/flink/flink-docs-master/docs/connectors/table/jdbc/)よりも高い安定性とパフォーマンスを提供しています。

> **注意**
>
> Flinkコネクタを使用してStarRocksテーブルにデータをロードするには、SELECTおよびINSERTの権限が必要です。これらの権限がない場合は、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)の手順に従って、StarRocksクラスタに接続するために使用するユーザーにこれらの権限を付与してください。

## バージョン要件

| コネクタ | Flink                    | StarRocks     | Java | Scala     |
|-----------|--------------------------|---------------| ---- |-----------|
| 1.2.8     | 1.13,1.14,1.15,1.16,1.17 | 2.1以降| 8    | 2.11,2.12 |
| 1.2.7     | 1.11,1.12,1.13,1.14,1.15 | 2.1以降| 8    | 2.11,2.12 |

## Flinkコネクタの取得

FlinkコネクタのJARファイルは、次の方法で取得できます。

- コンパイル済みのFlinkコネクタJARファイルを直接ダウンロードします。
- Mavenプロジェクトの依存関係としてFlinkコネクタを追加し、JARファイルをダウンロードします。
- Flinkコネクタのソースコードを自分でコンパイルしてJARファイルにします。

FlinkコネクタのJARファイルの命名形式は次のとおりです。

- Flink 1.15以降の場合、`flink-connector-starrocks-${connector_version}_flink-${flink_version}.jar`です。たとえば、Flink 1.15をインストールし、Flinkコネクタ1.2.7を使用したい場合は、`flink-connector-starrocks-1.2.7_flink-1.15.jar`を使用します。

- Flink 1.15より前の場合、`flink-connector-starrocks-${connector_version}_flink-${flink_version}_${scala_version}.jar`です。たとえば、環境にFlink 1.14とScala 2.12をインストールし、Flinkコネクタ1.2.7を使用したい場合は、`flink-connector-starrocks-1.2.7_flink-1.14_2.12.jar`を使用します。

> **注意**
>
> 一般的に、Flinkコネクタの最新バージョンは、Flinkの最新の3つのバージョンとの互換性を維持します。

### コンパイル済みのJARファイルをダウンロード

[Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks)から、対応するバージョンのFlinkコネクタJARファイルを直接ダウンロードします。

### Mavenの依存関係

Mavenプロジェクトの`pom.xml`ファイルに、次の形式に従ってFlinkコネクタを依存関係として追加します。`flink_version`、`scala_version`、および`connector_version`をそれぞれのバージョンに置き換えてください。

- Flink 1.15以降の場合

    ```xml
    <dependency>
        <groupId>com.starrocks</groupId>
        <artifactId>flink-connector-starrocks</artifactId>
        <version>${connector_version}_flink-${flink_version}</version>
    </dependency>
    ```

- Flink 1.15より前のバージョンの場合

    ```xml
    <dependency>
        <groupId>com.starrocks</groupId>
        <artifactId>flink-connector-starrocks</artifactId>
        <version>${connector_version}_flink-${flink_version}_${scala_version}</version>
    </dependency>
    ```

### 自分でコンパイルする

1. [Flinkコネクタパッケージ](https://github.com/StarRocks/starrocks-connector-for-apache-flink)をダウンロードします。
2. 次のコマンドを実行して、FlinkコネクタのソースコードをコンパイルしてJARファイルにします。`flink_version`は対応するFlinkのバージョンに置き換えてください。

      ```bash
      sh build.sh <flink_version>
      ```

   たとえば、環境のFlinkバージョンが1.15の場合、次のコマンドを実行します。

      ```bash
      sh build.sh 1.15
      ```

3. `target/`ディレクトリに移動して、コンパイルされたFlinkコネクタJARファイル（例: `flink-connector-starrocks-1.2.7_flink-1.15-SNAPSHOT.jar`）を見つけます。

> **注意**
>
> 正式にリリースされていないFlinkコネクタの名前には、`SNAPSHOT`の接尾辞が含まれています。

## オプション

| **オプション**                        | **必須** | **デフォルト値** | **説明**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|-----------------------------------|--------------|-------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| connector                         | Yes          | NONE              | 使用するコネクタ。値は「starrocks」である必要があります。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| jdbc-url                          | Yes          | NONE              | FEのMySQLサーバーに接続するために使用されるアドレス。複数のアドレスを指定することができます。アドレスはカンマ（,）で区切られている必要があります。形式: `jdbc:mysql://<fe_host1>:<fe_query_port1>,<fe_host2>:<fe_query_port2>,<fe_host3>:<fe_query_port3>`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| load-url                          | Yes          | NONE              | StarRocksクラスタのFEのHTTP URL。複数のURLを指定することができます。URLはセミコロン（;）で区切られている必要があります。形式: `<fe_host1>:<fe_http_port1>;<fe_host2>:<fe_http_port2>`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| database-name                     | Yes          | NONE              | データをロードするStarRocksデータベースの名前。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| table-name                        | Yes          | NONE              | StarRocksにデータをロードするテーブルの名前。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| username                          | Yes          | NONE              | StarRocksにデータをロードするために使用するアカウントのユーザー名。アカウントには[SELECTおよびINSERTの権限](../sql-reference/sql-statements/account-management/GRANT.md)が必要です。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| password                          | Yes          | NONE              | 前述のアカウントのパスワード。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| sink.semantic                     | No           | at-least-once     | Sinkが保証するセマンティクス。有効な値: **at-least-once**および**exactly-once**。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| sink.version                      | No           | AUTO              | データをロードするために使用するインターフェース。このパラメータは、Flinkコネクタバージョン1.2.4以降でサポートされています。 <ul><li>`V1`: [Stream Load](../loading/StreamLoad.md)インターフェースを使用してデータをロードします。1.2.4より前のコネクタは、このモードのみをサポートしています。</li> <li>`V2`: [Stream Loadトランザクション](../loading/Stream_Load_transaction_interface.md)インターフェースを使用してデータをロードします。StarRocksのバージョンは少なくとも2.4である必要があります。メモリ使用量を最適化し、より安定したexactly-onceの実装を提供します。</li> <li>`AUTO`: StarRocksのバージョンがトランザクションStream Loadをサポートしている場合、自動的に`V2`を選択し、それ以外の場合は`V1`を選択します。</li></ul> |
| sink.label-prefix | No | NONE | Stream Loadで使用するラベルの接頭辞。コネクタ1.2.8以降を使用している場合は、設定することをお勧めします。ラベルの接頭辞は、Flinkジョブ、Routine Load、およびBroker Loadなどのすべてのタイプのロード間で一意である必要があります。詳細については、[exactly-onceの使用方法の注意事項](#exactly-once)を参照してください。 |
| sink.buffer-flush.max-bytes       | No           | 94371840(90M)     | 一度にStarRocksに送信される前にメモリに蓄積できるデータの最大サイズ。最大値は64MBから10GBまでです。このパラメータを大きな値に設定すると、ロードパフォーマンスが向上する一方で、ロードの待機時間が増加する可能性があります。                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| sink.buffer-flush.max-rows        | No           | 500000            | 一度にStarRocksに送信される前にメモリに蓄積できる行の最大数。このパラメータは、`sink.version`を`V1`に設定し、`sink.semantic`を`at-least-once`に設定した場合にのみ使用できます。有効な値: 64000から5000000まで。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| sink.buffer-flush.interval-ms     | No           | 300000            | データがフラッシュされる間隔。このパラメータは、`sink.semantic`を`at-least-once`に設定した場合にのみ使用できます。有効な値: 1000から3600000まで。単位: ms。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| sink.max-retries                  | No           | 3                 | システムがStream Loadジョブを実行しようとする回数。このパラメータは、`sink.version`を`V1`に設定した場合にのみ使用できます。有効な値: 0から10まで。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| sink.connect.timeout-ms           | No           | 1000              | HTTP接続を確立するためのタイムアウト。有効な値: 100から60000。単位: ms。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| sink.wait-for-continue.timeout-ms | No           | 10000             | 1.2.7以降でサポートされています。FEからのHTTP 100-continueの応答を待機するタイムアウト。有効な値: `3000`から`600000`。単位: ms                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| sink.ignore.update-before         | No           | true              | 1.2.8以降でサポートされています。StarRocksテーブルにデータをロードする際に、Flinkからの`UPDATE_BEFORE`レコードを無視するかどうかを指定します。このパラメータをfalseに設定すると、レコードはStarRocksテーブルへの削除操作として扱われます。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| sink.properties.*                 | No           | NONE              | Stream Loadの動作を制御するために使用されるパラメータ。たとえば、パラメータ`sink.properties.format`は、Stream Loadで使用される形式（CSVまたはJSONなど）を指定します。サポートされるパラメータとその説明のリストについては、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。                                                                                                                                                                                                                                                                                                                                 |
| sink.properties.format            | No           | csv               | Stream Loadで使用される形式。Flinkコネクタは、データのバッチごとにStarRocksに送信する前に、データをこの形式に変換します。有効な値: `csv`および`json`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| sink.properties.row_delimiter     | No           | \n                | CSV形式のデータの行区切り文字。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| sink.properties.column_separator  | No           | \t                | CSV形式のデータの列区切り文字。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| sink.properties.max_filter_ratio  | No           | 0                 | Stream Loadの最大エラートレランス。データ品質が不十分なためにフィルタリングされることができるデータレコードの最大割合です。有効な値: `0`から`1`。デフォルト値: `0`。詳細については、[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。                                                                                                                                                                                                                                                                                                                                                                      |
| sink.parallelism                  | No           | NONE              | コネクタの並列性。Flink SQLのみで使用できます。設定しない場合、Flinkプランナーが並列性を決定します。複数の並列性のシナリオでは、データが正しい順序で書き込まれることを保証する必要があります。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| sink.properties.strict_mode | No | false | Stream Loadの厳密モードを有効にするかどうかを指定します。不適格な行（一貫性のない列値など）がある場合のロード動作に影響を与えます。有効な値: `true`および`false`。デフォルト値: `false`。詳細については、[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。 |

## FlinkとStarRocksのデータ型のマッピング

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

## 使用方法の注意事項

### Exactly Once

- Sinkがexactly-onceセマンティクスを保証する場合は、StarRocksを2.5以降にアップグレードし、Flinkコネクタを1.2.4以降にアップグレードすることをお勧めします。
  - Flinkコネクタ1.2.4以降では、exactly-onceは、StarRocks 2.4以降で提供される[Stream Loadトランザクションインターフェース](https://docs.starrocks.io/en-us/latest/loading/Stream_Load_transaction_interface)に基づいて再設計されています。非トランザクションのStream Loadインターフェースに基づく以前の実装と比較して、新しい実装はメモリ使用量とチェックポイントのオーバーヘッドを削減し、リアルタイムパフォーマンスとロードの安定性を向上させます。

  - StarRocksのバージョンが2.4より前であるか、Flinkコネクタのバージョンが1.2.4より前である場合、Sinkは自動的に非トランザクションのStream Loadインターフェースに基づく実装を選択します。

- exactly-onceを保証するための設定

  - `sink.semantic`の値を`exactly-once`に設定する必要があります。

  - Flinkコネクタのバージョンが1.2.8以降の場合は、`sink.label-prefix`の値を指定することをお勧めします。ただし、ラベルの接頭辞は、Flinkのジョブ、Routine Load、およびBroker Loadなどのすべてのタイプのロード間で一意である必要があります。

    - ラベルの接頭辞が指定されている場合、Flinkコネクタは、いくつかのFlinkの障害シナリオで生成される可能性のある未処理のトランザクションをクリーンアップするために、ラベルの接頭辞を使用します。たとえば、Flinkジョブがチェックポイントが進行中の状態で失敗した場合などです。これらの未処理のトランザクションは、一般的には`PREPARED`の状態にあります。StarRocksで`SHOW PROC '/transactions/<db_id>/running';`を使用して表示すると、これらのトランザクションを確認できます。Flinkジョブがチェックポイントから復元されると、Flinkコネクタは、ラベルの接頭辞とチェックポイントのいくつかの情報に基づいて、これらの未処理のトランザクションを見つけて中止します。Flinkジョブが終了すると、Flinkコネクタは、exactly-onceを実現するための2フェーズコミットメカニズムのため、これらのトランザクションを中止することはできません。Flinkジョブが終了すると、Flinkコネクタは、Flinkのチェックポイントコーディネーターからの通知を受け取って、これらのトランザクションを成功したチェックポイントに含める必要があるかどうかを判断することができず、これらのトランザクションが無条件に中止される可能性があります。これにより、データの損失が発生する可能性があります。Flinkでエンドツーエンドのexactly-onceを実現する方法については、この[ブログ記事](https://flink.apache.org/2018/02/28/an-overview-of-end-to-end-exactly-once-processing-in-apache-flink-with-apache-kafka-too/)を参照してください。

    - ラベルの接頭辞が指定されていない場合、未処理のトランザクションはタイムアウト後にStarRocksによってクリーンアップされます。ただし、Flinkジョブが頻繁に失敗する場合、未処理のトランザクションの数がStarRocksの`max_running_txn_num_per_db`の制限に達する可能性があります。タイムアウトの長さは、StarRocks FEの設定`prepared_transaction_default_timeout_second`で制御されます。デフォルト値は`86400`（1日）です。ラベルの接頭辞が指定されていない場合、タイムアウトまでの時間が長い場合は、この設定に小さい値を設定してトランザクションの有効期限を短くすると、トランザクションが早く期限切れになります。

- 長時間のダウンタイムの後でFlinkジョブがチェックポイントまたはセーブポイントから復旧することが確実である場合は、次のStarRocksの設定を調整して、データの損失を回避してください。

  - `prepared_transaction_default_timeout_second`：StarRocks FEの設定で、デフォルト値は`86400`です。この設定の値は、Flinkジョブのダウンタイムよりも大きくする必要があります。そうでない場合、チェックポイントに含まれる未処理のトランザクションは、Flinkジョブを再起動する前にタイムアウトのために中止される可能性があり、データの損失につながります。

    この設定の値を大きくする場合は、ラベルの接頭辞を指定しておくことをお勧めします。ラベルの接頭辞が指定されている場合、Flinkコネクタは、Flinkのセーブポイントまたはチェックポイントに保存されたトランザクションのラベルを使用して、StarRocksのトランザクションの状態を確認し、これらのトランザクションがコミットされたかどうかを判断し、データの損失を防ぐことができます。

  - `label_keep_max_second`および`label_keep_max_num`：StarRocks FEの設定で、デフォルト値はそれぞれ`259200`および`1000`です。詳細については、[FE configurations](../loading/Loading_intro.md#fe-configurations)を参照してください。`label_keep_max_second`の値は、Flinkジョブのダウンタイムよりも大きくする必要があります。そうでない場合、Flinkコネクタは、Flinkのセーブポイントまたはチェックポイントに保存されたトランザクションのラベルを使用して、StarRocksのトランザクションの状態を確認し、これらのトランザクションがコミットされたかどうかを判断することができず、最終的にはデータの損失につながる可能性があります。

  これらの設定は変更可能であり、`ADMIN SET FRONTEND CONFIG`を使用して変更できます。

  ```SQL
    ADMIN SET FRONTEND CONFIG ("prepared_transaction_default_timeout_second" = "3600");
    ADMIN SET FRONTEND CONFIG ("label_keep_max_second" = "259200");
    ADMIN SET FRONTEND CONFIG ("label_keep_max_num" = "1000");
  ```

### フラッシュポリシー

Flinkコネクタはデータをメモリにバッファリングし、Stream Loadを介して一括でStarRocksにフラッシュします。フラッシュがトリガーされる方法は、at-least-onceとexactly-onceで異なります。

at-least-onceの場合、フラッシュは次のいずれかの条件が満たされるとトリガーされます。

- バッファリングされた行のバイト数が制限`sink.buffer-flush.max-bytes`に達した場合
- バッファリングされた行の数が制限`sink.buffer-flush.max-rows`に達した場合（sinkバージョンV1のみ有効）
- 最後のフラッシュから経過した時間が制限`sink.buffer-flush.interval-ms`に達した場合
- チェックポイントがトリガーされた場合

exactly-onceの場合、フラッシュはチェックポイントがトリガーされた場合にのみ発生します。

### ロードメトリクスの監視

Flinkコネクタは、以下のメトリクスを提供してロードを監視します。

| メトリクス                     | タイプ    | 説明                                                     |
|--------------------------|---------|-----------------------------------------------------------------|
| totalFlushBytes          | counter | 正常にフラッシュされたバイト数。                                     |
| totalFlushRows           | counter | 正常にフラッシュされた行数。                                      |
| totalFlushSucceededTimes | counter | データが正常にフラッシュされた回数。  |
| totalFlushFailedTimes    | counter | データのフラッシュに失敗した回数。                  |
| totalFilteredRows        | counter | フィルタリングされた行数。totalFlushRowsにも含まれます。    |

## 例

以下の例では、Flinkコネクタを使用してFlink SQLまたはFlink DataStreamを使用してデータをStarRocksテーブルにロードする方法を示しています。

### 準備

#### StarRocksテーブルの作成

データベース`test`を作成し、Primary Keyテーブル`score_board`を作成します。

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

- Flinkバイナリ[Flink 1.15.2](https://archive.apache.org/dist/flink/flink-1.15.2/flink-1.15.2-bin-scala_2.12.tgz)をダウンロードし、ディレクトリ`flink-1.15.2`に解凍します。
- [Flinkコネクタ1.2.7](https://repo1.maven.org/maven2/com/starrocks/flink-connector-starrocks/1.2.7_flink-1.15/flink-connector-starrocks-1.2.7_flink-1.15.jar)をダウンロードし、ディレクトリ`flink-1.15.2/lib`に配置します。
- 次のコマンドを実行して、Flinkクラスタを起動します。

    ```shell
    cd flink-1.15.2
    ./bin/start-cluster.sh
    ```

### Flink SQLで実行

- 次のコマンドを実行して、Flink SQLクライアントを起動します。

    ```shell
    ./bin/sql-client.sh
    ```

- Flinkテーブル`score_board`を作成し、Flink SQLクライアントを介してテーブルに値を挿入します。
StarRocksのPrimary Keyテーブルにデータをロードする場合は、Flink DDLで主キーを定義する必要があります。他のタイプのStarRocksテーブルの場合はオプションです。

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

入力レコードのタイプに応じて、CSV形式のJava `String`、JSON形式のJava `String`、またはカスタムのJavaオブジェクトなど、いくつかの方法でFlink DataStreamジョブを実装することができます。

- 入力レコードがCSV形式の`String`の場合。完全な例については、[LoadCsvRecords](https://github.com/StarRocks/starrocks-connector-for-apache-flink/tree/main/examples/src/main/java/com/starrocks/connector/flink/examples/datastream/LoadCsvRecords.java)を参照してください。

    ```java
    /**
     * CSV形式のレコードを生成します。各レコードは"\t"で区切られた3つの値を持ちます。
     * これらの値は、StarRocksテーブルの`id`、`name`、`score`の列にロードされます。
     */
    String[] records = new String[]{
            "1\tstarrocks-csv\t100",
            "2\tflink-csv\t100"
    };
    DataStream<String> source = env.fromElements(records);

    /**
     * 必要なプロパティでコネクタを設定します。
     * プロパティ "sink.properties.format" と "sink.properties.column_separator" を追加する必要もあります。
     * これにより、コネクタに入力レコードがCSV形式であること、列の区切り文字が"\t"であることを伝えることができます。
     * CSV形式のレコードでは他の列区切り文字も使用できますが、"sink.properties.column_separator" を対応する値に変更する必要があります。
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
    // オプションを使用してSinkを作成します。
    SinkFunction<String> starRockSink = StarRocksSink.sink(options);
    source.addSink(starRockSink);
    ```

- 入力レコードがJSON形式の`String`の場合。完全な例については、[LoadJsonRecords](https://github.com/StarRocks/starrocks-connector-for-apache-flink/tree/main/examples/src/main/java/com/starrocks/connector/flink/examples/datastream/LoadJsonRecords.java)を参照してください。

    ```java
    /**
     * JSON形式のレコードを生成します。
     * 各レコードは、StarRocksテーブルの`id`、`name`、`score`の列に対応する3つのキーと値のペアを持ちます。
     */
    String[] records = new String[]{
            "{\"id\":1, \"name\":\"starrocks-json\", \"score\":100}",
            "{\"id\":2, \"name\":\"flink-json\", \"score\":100}",
    };
    DataStream<String> source = env.fromElements(records);

    /** 
     * 必要なプロパティでコネクタを設定します。
     * プロパティ "sink.properties.format" と "sink.properties.strip_outer_array" を追加する必要もあります。
     * これにより、コネクタに入力レコードがJSON形式であること、最外部の配列構造を削除することを伝えることができます。 
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
    // オプションを使用してSinkを作成します。
    SinkFunction<String> starRockSink = StarRocksSink.sink(options);
    source.addSink(starRockSink);
    ```

- 入力レコードがカスタムのJavaオブジェクトの場合。完全な例については、[LoadCustomJavaRecords](https://github.com/StarRocks/starrocks-connector-for-apache-flink/tree/main/examples/src/main/java/com/starrocks/connector/flink/examples/datastream/LoadCustomJavaRecords.java)を参照してください。

  - この例では、入力レコードは単純なPOJO `RowData`です。

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

  - メインプログラムは次のようになります。

    ```java
    // RowDataを使用してレコードを生成します。
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
     * Flinkコネクタは、ロードする行を表すためにJavaオブジェクト配列（Object[]）を使用し、各要素は列の値です。
     * 配列のスキーマは、StarRocksテーブルのスキーマと一致する必要があります。
     */
    TableSchema schema = TableSchema.builder()
            .field("id", DataTypes.INT().notNull())
            .field("name", DataTypes.STRING())
            .field("score", DataTypes.INT())
            // StarRocksテーブルがPrimary Keyテーブルの場合、主キー`id`にはnotNull()（たとえば、DataTypes.INT().notNull()）を指定する必要があります。
            .primaryKey("id")
            .build();
    // スキーマに基づいてRowDataをObject[]に変換します。
    RowDataTransformer transformer = new RowDataTransformer();
    // スキーマ、オプション、およびトランスフォーマーを使用してSinkを作成します。
    SinkFunction<RowData> starRockSink = StarRocksSink.sink(schema, options, transformer);
    source.addSink(starRockSink);
    ```

  - メインプログラムの中の`RowDataTransformer`は次のように定義されます。

    ```java
    private static class RowDataTransformer implements StarRocksSinkRowBuilder<RowData> {
    
        /**
         * 入力のRowDataに基づいてオブジェクト配列の各要素を設定します。
         * 配列のスキーマは、StarRocksテーブルのスキーマと一致する必要があります。
         */
        @Override
        public void accept(Object[] internalRow, RowData rowData) {
            internalRow[0] = rowData.id;
            internalRow[1] = rowData.name;
            internalRow[2] = rowData.score;
            // StarRocksテーブルがPrimary Keyテーブルの場合、最後の要素を設定してデータのロードがUPSERTまたはDELETE操作であるかどうかを示す必要があります。
            internalRow[internalRow.length - 1] = StarRocksSinkOP.UPSERT.ordinal();
        }
    }  
    ```

## ベストプラクティス

### プライマリキーテーブルへのデータのロード

このセクションでは、部分更新と条件付き更新を実現するために、StarRocksのプライマリキーテーブルにデータをロードする方法を示します。
これらの機能の紹介については、[データのロードによるデータの変更](https://docs.starrocks.io/en-us/latest/loading/Load_to_Primary_Key_tables)を参照してください。
これらの例では、Flink SQLを使用します。

#### 準備

StarRocksでデータベース`test`を作成し、プライマリキーテーブル`score_board`を作成します。

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

この例では、`id`と`name`の列にのみデータをロードする方法を示します。

1. MySQLクライアントでStarRocksテーブル`score_board`に2つのデータ行を挿入します。

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

   - `id`と`name`の列のみを含むDDLを定義します。
   - オプション`sink.properties.partial_update`を`true`に設定します。これにより、Flinkコネクタが部分更新を実行するようになります。
   - Flinkコネクタのバージョンが`<=` 1.2.7の場合、オプション`sink.properties.columns`を`id,name,__op`に設定する必要もあります。これにより、Flinkコネクタに更新する必要のある列を指示します。`__op`フィールドを末尾に追加する必要があることに注意してください。`__op`フィールドは、データのロードがUPSERTまたはDELETE操作であることを示し、その値はコネクタによって自動的に設定されます。

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
        -- Flinkコネクタのバージョンが1.2.7以下の場合のみ
        'sink.properties.columns' = 'id,name,__op'
    ); 
    ```

3. Flinkテーブルに2つのデータ行を挿入します。データ行のプライマリキーは、StarRocksテーブルの行と同じですが、`name`列の値が変更されています。

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

    `name`の値のみが変更され、`score`の値は変更されていないことがわかります。

#### 条件付き更新

この例では、列`score`の値に応じて条件付きで更新する方法を示します。`id`の更新は、新しい`score`の値が古い値以上の場合にのみ有効になります。

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

2. Flinkテーブル`score_board`を以下の方法で作成します:
  
    - すべての列を含むDDLを定義します。
    - オプション`sink.properties.merge_condition`を`score`に設定します。これにより、コネクタが条件として列`score`を使用するようになります。
    - オプション`sink.version`を`V1`に設定します。これにより、コネクタがStream Loadを使用するようになります。

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

3. Flinkテーブルに2つのデータ行を挿入します。データ行のプライマリキーは、StarRocksテーブルの行と同じです。最初のデータ行は`score`列の値が小さく、2番目のデータ行は`score`列の値が大きいです。

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

   2番目のデータ行の値のみが変更され、1番目のデータ行の値は変更されていないことがわかります。

### BITMAP型の列にデータをロードする

[`BITMAP`](https://docs.starrocks.io/en-us/latest/sql-reference/sql-statements/data-types/BITMAP)は、UVのカウントなどの高速なカウントを実現するためによく使用されます。詳細については、[ビットマップを使用した正確なカウントのディスティンクト](https://docs.starrocks.io/en-us/latest/using_starrocks/Using_bitmap)を参照してください。
ここでは、UVのカウントを例にとり、`BITMAP`型の列にデータをロードする方法を示します。

1. MySQLクライアントでStarRocksの集計テーブルを作成します。

   データベース`test`で、列`visit_users`を`BITMAP`型として定義し、集計関数`BITMAP_UNION`で構成された集計テーブル`page_uv`を作成します。

    ```SQL
    CREATE TABLE `test`.`page_uv` (
      `page_id` INT NOT NULL COMMENT 'ページID',
      `visit_date` datetime NOT NULL COMMENT 'アクセス時間',
      `visit_users` BITMAP BITMAP_UNION NOT NULL COMMENT 'ユーザーID'
    ) ENGINE=OLAP
    AGGREGATE KEY(`page_id`, `visit_date`)
    DISTRIBUTED BY HASH(`page_id`);
    ```

2. Flink SQLクライアントでFlinkテーブルを作成します。

    Flinkテーブルの`visit_user_id`列は`BIGINT`型であり、この列をStarRocksテーブルの`BITMAP`型の`visit_users`列にロードしたいとします。したがって、FlinkテーブルのDDLを定義する際には、次の点に注意してください。
    - Flinkは`BITMAP`をサポートしていないため、StarRocksテーブルの`BITMAP`型の`visit_users`列を表すために、`visit_user_id`列を`BIGINT`型として定義する必要があります。
    - オプション`sink.properties.columns`を`page_id,visit_date,user_id,visit_users=to_bitmap(visit_user_id)`に設定する必要があります。これにより、コネクタにFlinkテーブルとStarRocksテーブルの列のマッピングを伝えることができます。また、`to_bitmap`関数を使用して、`BIGINT`型のデータを`BITMAP`型に変換するようにコネクタに伝える必要があります。

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

3. Flinkテーブルにデータをロードします。

    ```SQL
    INSERT INTO `page_uv` VALUES
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 13),
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 23),
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 33),
       (1, CAST('2020-06-23 02:30:30' AS TIMESTAMP), 13),
       (2, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 23);
    ```

4. MySQLクライアントでStarRocksテーブルからページのUVを計算します。

    ```SQL
    MySQL [test]> SELECT `page_id`, COUNT(DISTINCT `visit_users`) FROM `page_uv` GROUP BY `page_id`;
    +---------+-----------------------------+
    | page_id | count(DISTINCT visit_users) |
    +---------+-----------------------------+
    |       2 |                           1 |
    |       1 |                           3 |
    +---------+-----------------------------+
    2 rows in set (0.05 sec)
    ```

### HLL型の列にデータをロードする

[`HLL`](../sql-reference/sql-statements/data-types/HLL.md)は、近似的なカウントのディスティンクトを実現するために使用されます。詳細については、[HLLを使用した近似的なカウントのディスティンクト](../using_starrocks/Using_HLL.md)を参照してください。

ここでは、UVのカウントを例にとり、`HLL`型の列にデータをロードする方法を示します。

1. StarRocksの集計テーブルを作成します。

   データベース`test`で、列`visit_users`を`HLL`型として定義し、集計関数`HLL_UNION`で構成された集計テーブル`hll_uv`を作成します。

    ```SQL
    CREATE TABLE `hll_uv` (
      `page_id` INT NOT NULL COMMENT 'ページID',
      `visit_date` datetime NOT NULL COMMENT 'アクセス時間',
      `visit_users` HLL HLL_UNION NOT NULL COMMENT 'ユーザーID'
    ) ENGINE=OLAP
    AGGREGATE KEY(`page_id`, `visit_date`)
    DISTRIBUTED BY HASH(`page_id`);
    ```

2. Flink SQLクライアントでFlinkテーブルを作成します。

   Flinkテーブルの`visit_user_id`列は`BIGINT`型であり、この列をStarRocksテーブルの`HLL`型の`visit_users`列にロードしたいとします。したがって、FlinkテーブルのDDLを定義する際には、次の点に注意してください。
    - Flinkは`HLL`をサポートしていないため、StarRocksテーブルの`HLL`型の`visit_users`列を表すために、`visit_user_id`列を`BIGINT`型として定義する必要があります。
    - オプション`sink.properties.columns`を`page_id,visit_date,user_id,visit_users=hll_hash(visit_user_id)`に設定する必要があります。これにより、コネクタにFlinkテーブルとStarRocksテーブルの列のマッピングを伝えることができます。また、`hll_hash`関数を使用して、`BIGINT`型のデータを`HLL`型に変換するようにコネクタに伝える必要があります。

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

3. Flinkテーブルにデータをロードします。

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
    | page_id | count(DISTINCT visit_users) |
    +---------+-----------------------------+
    |       3 |                           2 |
    |       4 |                           1 |
    +---------+-----------------------------+
    2 rows in set (0.04 sec)
    ```
