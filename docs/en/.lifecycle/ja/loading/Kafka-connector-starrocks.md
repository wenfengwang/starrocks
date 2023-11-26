---
displayed_sidebar: "Japanese"
---

# Kafkaコネクタを使用してデータをロードする

StarRocksは、Apache Kafka®コネクタ（StarRocks Connector for Apache Kafka®）という独自のコネクタを提供しており、Kafkaからメッセージを連続的に消費し、StarRocksにロードします。Kafkaコネクタは、少なくとも一度のセマンティクスを保証します。

Kafkaコネクタは、Kafka Connectとシームレスに統合することができます。これにより、StarRocksはKafkaエコシステムとより良く統合されます。リアルタイムデータをStarRocksにロードする場合は、Kafkaコネクタを使用することをお勧めします。以下のシナリオでは、ルーチンロードではなく、Kafkaコネクタを使用することをお勧めします。

- ソースデータの形式が、例えばProtobufであり、JSON、CSV、またはAvroではない場合。
- Debezium形式のCDCデータなど、データ変換をカスタマイズする場合。
- 複数のKafkaトピックからデータをロードする場合。
- Confluent Cloudからデータをロードする場合。
- ロード速度とリソース利用のバランスを実現するために、ロードバッチサイズ、並列度、およびその他のパラメータをより細かく制御する必要がある場合。

## 準備

### Kafka環境のセットアップ

セルフマネージドのApache KafkaクラスタとConfluent Cloudの両方がサポートされています。

- セルフマネージドのApache Kafkaクラスタの場合、Apache KafkaクラスタとKafka Connectクラスタをデプロイし、トピックを作成してください。
- Confluent Cloudの場合、Confluentアカウントを持っており、クラスタとトピックを作成してください。

### Kafkaコネクタのインストール

KafkaコネクタをKafka Connectに提出してください。

- セルフマネージドのKafkaクラスタの場合:

  - [starrocks-kafka-connector-1.0.0.tar.gz](https://releases.starrocks.io/starrocks/starrocks-kafka-connector-1.0.0.tar.gz)をダウンロードして解凍してください。
  - 解凍したディレクトリを`plugin.path`プロパティで指定されたパスにコピーしてください。`plugin.path`プロパティは、Kafka Connectクラスタ内のワーカーノードの設定ファイルに記載されています。

- Confluent Cloudの場合:

  > **注意**
  >
  > Kafkaコネクタは現在、Confluent Hubにアップロードされていません。圧縮ファイルをConfluent Cloudにアップロードする必要があります。

### StarRocksテーブルの作成

Kafkaのトピックとデータに基づいて、StarRocksにテーブルを作成してください。

## 例

以下の手順は、セルフマネージドのKafkaクラスタを例にして、Kafkaコネクタの設定とKafka Connectの起動（Kafkaサービスの再起動は不要）を示しています。

1. **connect-StarRocks-sink.properties**という名前のKafkaコネクタの設定ファイルを作成し、パラメータを設定してください。パラメータの詳細については、[パラメータ](#parameters)を参照してください。

    ```Properties
    name=starrocks-kafka-connector
    connector.class=com.starrocks.connector.kafka.StarRocksSinkConnector
    topics=dbserver1.inventory.customers
    starrocks.http.url=192.168.xxx.xxx:8030,192.168.xxx.xxx:8030
    starrocks.username=root
    starrocks.password=123456
    starrocks.database.name=inventory
    key.converter=io.confluent.connect.json.JsonSchemaConverter
    value.converter=io.confluent.connect.json.JsonSchemaConverter
    ```

    > **注意**
    >
    > ソースデータがDebezium形式のCDCデータであり、StarRocksテーブルがプライマリキーテーブルである場合、ソースデータの変更をプライマリキーテーブルに同期するために、`transform`を[設定](#load-debezium-formatted-cdc-data)する必要があります。

2. Kafkaコネクタを実行してください（Kafkaサービスの再起動は不要です）。以下のコマンドのパラメータと説明については、[Kafkaドキュメント](https://kafka.apache.org/documentation.html#connect_running)を参照してください。

    - スタンドアロンモード

        ```shell
        bin/connect-standalone worker.properties connect-StarRocks-sink.properties [connector2.properties connector3.properties ...]
        ```

    - 分散モード

      > **注意**
      >
      > 本番環境では、分散モードを使用することをお勧めします。

    - ワーカーを起動してください。

        ```shell
        bin/connect-distributed worker.properties
        ```

    - 分散モードでは、コネクタの設定はコマンドラインで渡されません。代わりに、以下で説明するREST APIを使用してKafkaコネクタを設定し、Kafka Connectを実行してください。

        ```Shell
        curl -i http://127.0.0.1:8083/connectors -H "Content-Type: application/json" -X POST -d '{
        "name":"starrocks-kafka-connector",
        "config":{
            "connector.class":"com.starrocks.connector.kafka.SinkConnector",
            "topics":"dbserver1.inventory.customers",
            "starrocks.http.url":"192.168.xxx.xxx:8030,192.168.xxx.xxx:8030",
            "starrocks.user":"root",
            "starrocks.password":"123456",
            "starrocks.database.name":"inventory",
            "key.converter":"io.confluent.connect.json.JsonSchemaConverter",
            "value.converter":"io.confluent.connect.json.JsonSchemaConverter"
        }
        }
        ```

3. StarRocksテーブルでデータをクエリしてください。

## パラメータ

| パラメータ                           | 必須 | デフォルト値                                                | 説明                                                  |
| ----------------------------------- | ---- | ------------------------------------------------------------ | ------------------------------------------------------ |
| name                                | YES  |                                                              | このKafkaコネクタの名前です。このKafka Connectクラスタ内のすべてのKafkaコネクタ間でグローバルに一意である必要があります。例：starrocks-kafka-connector。 |
| connector.class                     | YES  | com.starrocks.connector.kafka.SinkConnector                  | このKafkaコネクタのシンクで使用されるクラスです。                   |
| topics                              | YES  |                                                              | 購読する1つ以上のトピックで、各トピックはStarRocksテーブルに対応します。デフォルトでは、StarRocksはトピック名がStarRocksテーブルの名前と一致すると見なします。したがって、StarRocksはトピック名を使用して対象のStarRocksテーブルを決定します。`topics`または`topics.regex`（以下参照）のいずれかを選択してくださいが、両方を使用しないでください。ただし、StarRocksテーブル名がトピック名と異なる場合は、オプションの`starrocks.topic2table.map`パラメータ（以下参照）を使用して、トピック名からテーブル名へのマッピングを指定してください。 |
| topics.regex                        |      | 購読する1つ以上のトピックに一致するための正規表現。詳細については、`topics`を参照してください。`topics.regex`（上記）または`topics`のいずれかを選択してください。 |                                                              |
| starrocks.topic2table.map           | NO   |                                                              | トピック名がStarRocksテーブル名と異なる場合のStarRocksテーブル名とトピック名のマッピングです。形式は`<topic-1>:<table-1>,<topic-2>:<table-2>,...`です。 |
| starrocks.http.url                  | YES  |                                                              | StarRocksクラスタのFEのHTTP URLです。形式は`<fe_host1>:<fe_http_port1>,<fe_host2>:<fe_http_port2>,...`です。複数のアドレスはカンマ（,）で区切られます。例：`192.168.xxx.xxx:8030,192.168.xxx.xxx:8030`。 |
| starrocks.database.name             | YES  |                                                              | StarRocksデータベースの名前です。                              |
| starrocks.username                  | YES  |                                                      | StarRocksクラスタのアカウントのユーザー名です。ユーザーはStarRocksテーブルに対して[INSERT](../sql-reference/sql-statements/account-management/GRANT.md)権限が必要です。 |
| starrocks.password                  | YES  |                                                              | StarRocksクラスタのアカウントのパスワードです。              |
| key.converter                       | NO   |           Kafka Connectクラスタで使用されるキーコンバータ                                                           | このパラメータは、シンクコネクタ（Kafka-connector-starrocks）のキーをデシリアライズするために使用されるキーコンバータを指定します。デフォルトのキーコンバータは、Kafka Connectクラスタで使用されるキーコンバータです。|
| value.converter                     | NO   |    Kafka Connectクラスタで使用される値コンバータ                             | このパラメータは、シンクコネクタ（Kafka-connector-starrocks）の値をデシリアライズするために使用される値コンバータを指定します。デフォルトの値コンバータは、Kafka Connectクラスタで使用される値コンバータです。 |
| key.converter.schema.registry.url   | NO   |                                                              | キーコンバータのスキーマレジストリURLです。                   |
| value.converter.schema.registry.url | NO   |                                                              | 値コンバータのスキーマレジストリURLです。                 |
| tasks.max                           | NO   | 1                                                            | Kafkaコネクタが作成できるタスクスレッドの上限です。通常、Kafka ConnectクラスタのワーカーノードのCPUコア数と同じです。このパラメータを調整してロードパフォーマンスを制御できます。 |
| bufferflush.maxbytes                | NO   | 94371840(90M)                                                | 一度にStarRocksに送信される前にメモリに蓄積できるデータの最大サイズです。最大値は64MBから10GBまでです。ここで言及されているしきい値は、Stream Load SDKバッファがデータをバッファリングするために複数のStream Loadジョブを作成する可能性があるため、合計データサイズを指します。 |
| bufferflush.intervalms              | NO   | 300000                                                       | データのバッチを送信する間隔で、ロードの遅延を制御します。範囲：[1000, 3600000]。 |
| connect.timeoutms                   | NO   | 1000                                                         | HTTP URLへの接続のタイムアウトです。範囲：[100, 60000]。 |
| sink.properties.*                   |      |                                                              | ロード動作を制御するためのStream Loadパラメータです。たとえば、`sink.properties.format`パラメータは、CSVやJSONなどのStream Loadに使用される形式を指定します。サポートされるパラメータとその説明のリストについては、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM LOAD.md)を参照してください。 |
| sink.properties.format              | NO   | json                                                         | Stream Loadに使用される形式です。Kafkaコネクタは、データの各バッチをStarRocksに送信する前に、その形式に変換します。有効な値：`csv`および`json`。詳細については、[CSVパラメータ**](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md#csv-parameters)および** [JSONパラメータ](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md#json-parameters)を参照してください。 |

## 制限事項

- Kafkaトピックから単一のメッセージを複数のデータ行にフラット化してStarRocksにロードすることはサポートされていません。
- Kafkaコネクタのシンクは、少なくとも一度のセマンティクスを保証します。

## ベストプラクティス

### Debezium形式のCDCデータのロード

KafkaデータがDebezium CDC形式であり、StarRocksテーブルがプライマリキーテーブルである場合、`transforms`パラメータとその他の関連パラメータを設定する必要があります。

```Properties
transforms=addfield,unwrap
transforms.addfield.type=com.starrocks.connector.kafka.transforms.AddOpFieldForDebeziumRecord
transforms.unwrap.type=io.debezium.transforms.ExtractNewRecordState
transforms.unwrap.drop.tombstones=true
transforms.unwrap.delete.handling.mode
```

上記の設定では、`transforms=addfield,unwrap`を指定しています。

- addfield変換は、Debezium CDC形式のデータの各レコードに__opフィールドを追加し、StarRocksプライマリキーモデルテーブルをサポートするために使用されます。StarRocksテーブルがプライマリキーテーブルでない場合は、addfield変換を指定する必要はありません。addfield変換のクラスはcom.Starrocks.Kafka.Transforms.AddOpFieldForDebeziumRecordです。これはKafkaコネクタのJARファイルに含まれているため、手動でインストールする必要はありません。
- unwrap変換はDebeziumによって提供され、操作タイプに基づいてDebeziumの複雑なデータ構造を展開するために使用されます。詳細については、[New Record State Extraction](https://debezium.io/documentation/reference/stable/transformations/event-flattening.html)を参照してください。
