---
displayed_sidebar: "Japanese"
---

# Kafkaコネクタを使用してデータをロードする

StarRocksは、Apache Kafka®コネクタ（StarRocks Connector for Apache Kafka®）という自社開発のコネクタを提供しており、このコネクタはKafkaからメッセージを連続的に消費し、StarRocksにロードします。Kafkaコネクタは少なくとも一度のセマンティクスを保証します。

KafkaコネクタはKafka Connectとシームレスに統合し、StarRocksをKafkaエコシステムとより良く統合できます。StarRocksにリアルタイムデータをロードしたい場合は、Kafkaコネクタを使用することをお勧めします。以下のシナリオでは、Routine LoadよりもKafkaコネクタの使用をお勧めします。

- CSV、JSON、Avro形式のデータのみをサポートするRoutine Loadに比べて、KafkaコネクタはProtobufなどのより多くの形式のデータをロードできます。Kafka Connectのコンバータを使用してデータをJSONおよびCSV形式に変換できれば、Kafkaコネクタを使用してデータをStarRocksにロードできます。
- Debezium形式のCDCデータなどのデータ変換のカスタマイズ。
- 複数のKafkaトピックからデータをロードする。
- Confluentクラウドからデータをロードする。
- ロードバッチサイズ、並列性、およびその他のパラメータを細かく制御して、ロード速度とリソース利用率のバランスを実現する必要がある。

## 準備

### Kafka環境のセットアップ

セルフマネージド型のApache KafkaクラスタやConfluentクラウドの両方をサポートしています。

- セルフマネージド型のApache Kafkaクラスタの場合、Apache KafkaクラスタとKafka Connectクラスタを展開し、トピックを作成してください。
- Confluentクラウドの場合、Confluentアカウントを取得し、クラスタとトピックを作成してください。

### Kafkaコネクタのインストール

KafkaコネクタをKafka Connectに提出します。

- セルフマネージド型Kafkaクラスタの場合：

  - [starrocks-kafka-connector-1.0.0.tar.gz](https://releases.starrocks.io/starrocks/starrocks-kafka-connector-1.0.0.tar.gz)をダウンロードして解凍します。
  - 解凍されたディレクトリを`plugin.path`プロパティで指定されたパスにコピーします。`plugin.path`プロパティは、Kafka Connectクラスタ内のワーカーノードの構成ファイルにあります。

- Confluentクラウドの場合：

  > **注意**
  >
  > Kafkaコネクタは現在Confluent Hubにアップロードされていません。圧縮ファイルをConfluentクラウドにアップロードする必要があります。

### StarRocksテーブルの作成

Kafkaトピックとデータに応じて、StarRocksにテーブルを作成してください。

## 例

次の手順では、Kafkaクラスタをセルフマネージド型として取り上げ、Kafkaコネクタの構成方法およびKafka Connectの開始方法（Kafkaサービスの再起動不要）をデモンストレーションします。

1. **connect-StarRocks-sink.properties**という名前のKafkaコネクタ構成ファイルを作成し、パラメータを構成します。パラメータの詳細については、[パラメータ](#parameters)を参照してください。

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
    > もしソースデータがDebezium形式などのCDCデータで、StarRocksテーブルがプライマリキーテーブルである場合は、[transformの構成](#load-debezium-formatted-cdc-data)も必要です。これにより、ソースデータの変更がプライマリキーテーブルに同期されます。

2. Kafkaコネクタを実行します（Kafkaサービスの再起動は不要です）。次のコマンドで、パラメータと説明を確認してください。[Kafka Documentation](https://kafka.apache.org/documentation.html#connect_running)を参照してください。

    - スタンドアロンモード

        ```shell
        bin/connect-standalone worker.properties connect-StarRocks-sink.properties [connector2.properties connector3.properties ...]
        ```

    - 分散モード

      > **注意**
      >
      > 本番環境では、分散モードを使用することをお勧めします。

    - ワーカーを開始します。

        ```shell
        bin/connect-distributed worker.properties
        ```

    - 分散モードではコネクタの構成はコマンドラインで渡されません。代わりに、Kafkaコネクタを構成し、Kafkaコネクトを実行するために以下で説明するREST APIを使用してください。

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

3. StarRocksテーブルでのデータクエリ。

## パラメータ

| パラメータ                           | 必須 | デフォルト値                                                | 説明                                                  |
| ----------------------------------- | -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| name                                | YES      |                                                              | このKafkaコネクタの名前。この名前は、このKafka Connectクラスタ内のすべてのKafkaコネクタでグローバルに一意である必要があります。例えば、starrocks-kafka-connector。 |
| connector.class                     | YES      | com.starrocks.connector.kafka.SinkConnector                  | このKafkaコネクタのシンクで使用されるクラス。                   |
| topics                              | YES      |                                                              | StarRocksのテーブルに対応する1つ以上のトピック。デフォルトでは、StarRocksはトピック名がStarRocksテーブル名と一致すると仮定します。トピック名がStarRocksテーブル名と一致しない場合は、オプションの`starrocks.topic2table.map`パラメータ（以下）を使用して、トピック名からテーブル名へのマッピングを指定してください。 |
| topics.regex                        |          | トピックに対応する1つ以上のトピックをマッチさせる正規表現。詳細については、`topics`を参照してください。`topics`と`topics.regex`のどちらかを選択してくださいが、両方とも指定しないでください。 |                                                              |
| starrocks.topic2table.map           | NO       |                                                              | トピック名がStarRocksテーブル名と異なる場合は、トピック名とテーブル名のマッピングが必要です。フォーマットは`<topic-1>:<table-1>,<topic-2>:<table-2>,...`です。 |
| starrocks.http.url                  | YES      |                                                              | StarRocksクラスタのFEのHTTP URL。フォーマットは`<fe_host1>:<fe_http_port1>,<fe_host2>:<fe_http_port2>,...`です。複数のアドレスはカンマで区切られます。例：`192.168.xxx.xxx:8030,192.168.xxx.xxx:8030`。 |
| starrocks.database.name             | YES      |                                                              | StarRocksデータベースの名前。                              |
| starrocks.username                  | YES      |                                                     | StarRocksクラスタアカウントのユーザー名。ユーザーはStarRocksテーブルで[INSERT](../sql-reference/sql-statements/account-management/GRANT.md)権限を持っている必要があります。 |
| starrocks.password                  | YES      |                                                              | StarRocksクラスタアカウントのパスワード。              |
| key.converter                       | NO      |           Kafka Connectクラスタで使用されるKey変換器のデフォルト値                                                           | このパラメータは、シンクコネクタ（Kafka-connector-starrocks）のキーをデシリアライズするために使用されるキー変換器を指定します。デフォルトのキー変換器は、Kafka Connectクラスタで使用されているものです。|
| value.converter                     | NO      |     Kafka Connectクラスタで使用されるValue変換器のデフォルト値                            | このパラメータは、シンクコネクタ（Kafka-connector-starrocks）の値をデシリアライズするために使用される値変換器を指定します。デフォルトの値変換器は、Kafka Connectクラスタで使用されているものです。 |
| key.converter.schema.registry.url   | NO       |                                                              | キー変換器のスキーマレジストリURL。                   |
| value.converter.schema.registry.url | NO       |                                                              | 値変換器のスキーマレジストリURL。                 |
| tasks.max                           | NO       | 1                                                            | Kafkaコネクタが作成できるタスクスレッドの上限。通常はKafka ConnectクラスタのワーカーノードのCPUコア数と同じです。このパラメータを調整してロードのパフォーマンスを制御できます。 |
| bufferflush.maxbytes                | NO       | 94371840(90M)                                                | 一度にStarRocksに送信される前にメモリに蓄積できるデータの最大サイズ。最大値は64 MBから10 GBまでです。Stream Load SDKバッファは複数のStream Loadジョブを作成してデータをバッファリングする場合があるため、ここで言及した閾値は合計データサイズを指します。 |
| bufferflush.intervalms              | NO       | 300000                                                       | ロードの遅延を制御する、データのバッチ送信間隔。範囲：[1000, 3600000]。 |
| connect.timeoutms                   | NO       | 1000                                                         | HTTP URL への接続タイムアウトです。範囲：[100, 60000]。 |
| sink.properties.*                   |          |                                                              | ストリームロードのパラメータを制御します。たとえば、`sink.properties.format` パラメータは、CSV または JSON などのストリームロードに使用されるフォーマットを指定します。サポートされるパラメータとその説明のリストについては、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM LOAD.md) を参照してください。 |
| sink.properties.format              | NO       | json                                                         | ストリームロードに使用されるフォーマットです。Kafka コネクタは、データのバッチごとにこれらを StarRocks に送信する前に、そのフォーマットに変換します。有効な値：`csv` および `json`。詳細については、[CSV パラメータ](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md#csv-parameters) および [JSON パラメータ](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md#json-parameters) を参照してください。 |

## 制限

- Kafka トピックから単一メッセージを複数のデータ行にフラット化し、StarRocks にロードすることはサポートされていません。
- Kafka コネクタのシンクは、少なくとも一度のセマンティクスを保証します。

## ベストプラクティス

### Debezium 形式の CDC データをロードする

Kafka データが Debezium CDC フォーマットであり、StarRocks テーブルがプライマリキー テーブルである場合、`transforms` パラメータとその他の関連パラメータを構成する必要があります。

```Properties
transforms=addfield,unwrap
transforms.addfield.type=com.starrocks.connector.kafka.transforms.AddOpFieldForDebeziumRecord
transforms.unwrap.type=io.debezium.transforms.ExtractNewRecordState
transforms.unwrap.drop.tombstones=true
transforms.unwrap.delete.handling.mode
```

上記の構成では、`transforms=addfield,unwrap` を指定しています。

- addfield 変換は、Debezium CDC フォーマットデータの各レコードに __op フィールドを追加して、StarRocks プライマリキーモデルテーブルをサポートします。StarRocks テーブルがプライマリキーテーブルでない場合、addfield 変換を指定する必要はありません。addfield 変換クラスは com.Starrocks.Kafka.Transforms.AddOpFieldForDebeziumRecord です。これは Kafka コネクタ JAR ファイルに含まれているため、手動でインストールする必要はありません。
- unwrap 変換は Debezium によって提供され、操作タイプに基づいて Debezium の複雑なデータ構造を展開するために使用されます。詳細については、[New Record State Extraction](https://debezium.io/documentation/reference/stable/transformations/event-flattening.html) を参照してください。