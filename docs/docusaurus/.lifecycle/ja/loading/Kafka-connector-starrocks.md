---
displayed_sidebar: "Japanese"
---

# Kafkaコネクタを使用してデータをロードする

StarRocksでは、Apache Kafka®コネクタ（StarRocks Connector for Apache Kafka®）という自己開発のコネクタが提供されており、これによりKafkaからのメッセージを継続的に消費し、StarRocksにロードすることができます。Kafkaコネクタは少なくとも一度のセマンティクスを保証します。

KafkaコネクタはKafka Connectとシームレスに統合でき、これによりStarRocksはKafkaエコシステムとより良く統合されます。StarRocksにリアルタイムデータをロードしたい場合は、Kafkaコネクタを使用することをお勧めします。以下のシナリオでKafkaコネクタを使用することをお勧めします。

- CSV、JSON、およびAvro形式でのデータの読み込みのみをサポートするRoutine Loadと比較して、KafkaコネクタはProtobufなどのより多くの形式でデータを読み込むことができます。Kafka Connectのコンバータを使用してデータをJSONおよびCSV形式に変換できる限り、データはKafkaコネクタを介してStarRocksにロードすることができます。
- Debezium形式のCDCデータなどのデータ変換をカスタマイズする。
- 複数のKafkaトピックからデータをロードする。
- Confluent cloudからデータをロードする。
- ロード速度とリソース利用のバランスを実現するために、ロードバッチサイズ、並列性、およびその他のパラメータをより細かく制御する必要がある。

## 事前準備

### Kafka環境のセットアップ

自己運用型のApache KafkaクラスタとConfluent cloudの両方がサポートされています。

- 自己運用型Apache Kafkaクラスタの場合、Apache KafkaクラスタとKafka Connectクラスタを展開し、トピックを作成してください。
- Confluent cloudの場合、Confluentアカウントを取得し、クラスタとトピックを作成してください。

### Kafkaコネクタのインストール

KafkaコネクタをKafka Connectに提出してください。

- 自己運用型Kafkaクラスタ：

  - [starrocks-kafka-connector-1.0.0.tar.gz](https://releases.starrocks.io/starrocks/starrocks-kafka-connector-1.0.0.tar.gz)をダウンロードし、解凍してください。
  - 解凍したディレクトリを`plugin.path`プロパティで指定されたパスにコピーしてください。`plugin.path`プロパティはKafka Connectクラスタ内のワーカーノードの構成ファイルで見つけることができます。

- Confluent cloud：

  > **注意**
  >
  > Kafkaコネクタは現在Confluent Hubにはアップロードされていません。圧縮ファイルをConfluent cloudにアップロードする必要があります。

### StarRocksテーブルの作成

Kafkaトピックとデータに対応するStarRocksのテーブルを作成します。

## 例

次の手順は自己運用型Kafkaクラスタを例に取り、Kafkaコネクタの構成とKafka Connectの開始（Kafkaサービスを再起動する必要はありません）の方法を示しています。

1. **connect-StarRocks-sink.properties**という名前のKafkaコネクタの構成ファイルを作成し、パラメータを構成してください。パラメータの詳細については[Parameters](#parameters)を参照してください。

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
    > もしソースデータがDebezium形式のようなCDCデータであり、StarRocksテーブルがプライマリキーとなる場合、ソースデータの変更をプライマリキーテーブルに同期するために、[transformを構成](#load-debezium-formatted-cdc-data)する必要があります。

2. Kafkaコネクタを実行してください（Kafkaサービスを再起動する必要はありません）。次のコマンドでパラメータと説明を確認してください。[Kafka Documentation](https://kafka.apache.org/documentation.html#connect_running)を参照してください。

    - スタンドアロンモード

        ```shell
        bin/connect-standalone worker.properties connect-StarRocks-sink.properties [connector2.properties connector3.properties ...]
        ```

    - 分散モード

      > **注意**
      >
      > 本番環境では分散モードを使用することをお勧めします。

    - ワーカーを起動してください。

        ```shell
        bin/connect-distributed worker.properties
        ```

    - 分散モードではコネクタの設定はコマンドラインで渡されるわけではないことに注意してください。代わりに、Kafkaコネクタを構成し、Kafkaコネクトを実行するために以下のREST APIを使用してください。

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

| パラメータ                           | 必須     | デフォルト値                                                | 説明                                                    |
| ----------------------------------- | -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| name                                | YES      |                                                              | このKafkaコネクタの名前です。この名前は、このKafka Connectクラスタ内のすべてのKafkaコネクタ間でグローバルに一意である必要があります。たとえば、starrocks-kafka-connectorです。 |
| connector.class                     | YES      | com.starrocks.connector.kafka.SinkConnector                  | このKafkaコネクタのシンクで使用されるクラスです。                   |
| topics                              | YES      |                                                              | StarRocksのテーブルに対応する、1つ以上の購読するトピックです。デフォルトでは、StarRocksはトピック名がStarRocksテーブルの名前と一致すると想定しています。そのため、StarRocksはトピック名を使用してターゲットのStarRocksテーブルを決定します。`topics`または`topics.regex`（以下）のどちらか一方を記入するか、両方の記入を選択してください。ただし、StarRocksのテーブル名がトピック名と異なる場合は、オプションの`starrocks.topic2table.map`パラメータ（以下）を使用してトピック名からテーブル名へのマッピングを指定してください。 |
| topics.regex                        |          | 購読する1つ以上のトピックに一致する正規表現です。詳細については、`topics`を参照してください。`topics.regex`または`topics`（以上）のどちらか一方を記入するか、両方の記入を選択ください。 |                                                              |
| starrocks.topic2table.map           | NO       |                                                              | トピック名がStarRocksテーブル名と異なる場合のStarRocksテーブル名とトピック名のマッピングです。形式は`<topic-1>:<table-1>,<topic-2>:<table-2>,...`です。 |
| starrocks.http.url                  | YES      |                                                              | StarRocksクラスタのFEのHTTP URLです。形式は`<fe_host1>:<fe_http_port1>,<fe_host2>:<fe_http_port2>,...`です。複数のアドレスはカンマ（,）で区切られます。例えば、`192.168.xxx.xxx:8030,192.168.xxx.xxx:8030`です。 |
| starrocks.database.name             | YES      |                                                              | StarRocksデータベースの名前です。                              |
| starrocks.username                  | YES      |                                                      | StarRocksクラスタアカウントのユーザー名です。ユーザーはStarRocksテーブルに[INSERT](../sql-reference/sql-statements/account-management/GRANT.md)特権を持つ必要があります。 |
| starrocks.password                  | YES      |                                                              | StarRocksクラスタアカウントのパスワードです。              |
| key.converter                       | NO      |           Kafka Connectクラスタで使用されるキーコンバータ                           | このパラメータは、シンクコネクタ（Kafkaコネクタ-starrocks）のキーを逆シリアル化するために使用されるキーコンバータを指定します。デフォルトのキーコンバータは、Kafka Connectクラスタで使用されるキーコンバータです。|
| value.converter                     | NO      |    Kafka Connectクラスタで使用される値コンバータ                             | このパラメータは、シンクコネクタ（Kafkaコネクタ-starrocks）の値を逆シリアル化するために使用される値コンバータを指定します。デフォルトの値コンバータは、Kafka Connectクラスタで使用される値コンバータです。|
| key.converter.schema.registry.url   | NO       |                                                              | キーコンバータのスキーマレジストリURLです。                   |
| value.converter.schema.registry.url | NO       |                                                              | 値コンバータのスキーマレジストリURLです。                   |
| tasks.max                           | NO       | 1                                                            | Kafkaコネクタが作成できるタスクスレッドの上限です。通常、この値はKafka ConnectクラスタのワーカーノードのCPUコア数と同じです。このパラメータを調整してロードのパフォーマンスを制御することができます。 |
| bufferflush.maxbytes                | NO       | 94371840(90M)                                                | 一度にStarRocksに送信される前にメモリに蓄積できるデータの最大サイズです。最大値は64MBから10GBまでです。Stream Load SDKのバッファは、複数のStream Loadジョブを作成してデータをバッファに入れることができます。したがって、ここで言及されている閾値は、総データサイズを指します。 |
| bufferflush.intervalms              | NO       | 300000                                                       | ロードの遅延を制御するためにデータのバッチを送信する間隔です。範囲は[1000, 3600000]です。 |
| connect.timeoutms                   | NO       | 1000                                                         | HTTP URLへの接続のタイムアウト。範囲：[100, 60000]。 |
| sink.properties.*                   |          |                                                              | ストリームロードのパラメータを制御するためのパラメータ。 たとえば、パラメータ`sink.properties.format`はCSVやJSONなどのストリームロードに使用される形式を指定します。 サポートされているパラメータとその説明のリストについては、[ストリームロード](../sql-reference/sql-statements/data-manipulation/STREAM LOAD.md)を参照してください。 |
| sink.properties.format              | NO       | json                                                         | ストリームロードに使用される形式。Kafkaコネクタは、データのバッチごとにこれらをStarRocksに送信する前にその形式に変換します。 有効な値：`csv` および `json`。詳細は、[CSVパラメータ**](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md#csv-parameters)および** [JSONパラメータ](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md#json-parameters)を参照してください。 |

## 制限

- Kafkaトピックから単一のメッセージを複数のデータ行に展開し、StarRocksにロードすることはサポートされていません。
- KafkaコネクタのSinkは、少なくとも一度のセマンティクスを保証します。

## ベストプラクティス

### Debezium形式のCDCデータのロード

KafkaデータがDebezium CDC形式であり、StarRocksテーブルがプライマリキーテーブルである場合、`transforms`パラメータとその他関連パラメータを設定する必要があります。

```Properties
transforms=addfield,unwrap
transforms.addfield.type=com.starrocks.connector.kafka.transforms.AddOpFieldForDebeziumRecord
transforms.unwrap.type=io.debezium.transforms.ExtractNewRecordState
transforms.unwrap.drop.tombstones=true
transforms.unwrap.delete.handling.mode
```

上記の設定において、`transforms=addfield,unwrap`を指定しています。

- addfield変換は、Debezium CDC形式のデータの各レコードに__opフィールドを追加して、StarRocksプライマリキーモデルテーブルをサポートするために使用されます。 StarRocksテーブルがプライマリキーテーブルでない場合、addfield変換を指定する必要はありません。 addfield変換クラスはcom.Starrocks.Kafka.Transforms.AddOpFieldForDebeziumRecordです。 この変換はKafkaコネクタのJARファイルに含まれているため、手動でインストールする必要はありません。
- unwrap変換はDebeziumによって提供され、操作タイプに基づいてDebeziumの複雑なデータ構造を展開するために使用されます。 詳細については、[新しいレコード状態の抽出](https://debezium.io/documentation/reference/stable/transformations/event-flattening.html)を参照してください。