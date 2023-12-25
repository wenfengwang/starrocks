---
displayed_sidebar: English
---

# Kafka コネクタを使用してデータを読み込む

StarRocksは、Kafkaからのメッセージを継続的に消費し、StarRocksにロードする自社開発のコネクタであるApache Kafka® コネクタ（StarRocks Connector for Apache Kafka®）を提供しています。Kafka コネクタは、少なくとも一度の配信（at-least-once semantics）を保証します。

KafkaコネクタはKafka Connectとシームレスに統合でき、StarRocksをKafkaエコシステムとより良く統合させることができます。リアルタイムデータをStarRocksにロードしたい場合には、Kafkaコネクタの使用が賢明な選択です。Routine Loadと比較して、以下のシナリオでKafkaコネクタの使用を推奨します：

- Routine LoadはCSV、JSON、Avro形式のデータのロードのみをサポートしていますが、KafkaコネクタはProtobufなどのより多様な形式のデータをロードできます。Kafka Connectのコンバーターを使用してデータをJSONやCSV形式に変換できる限り、Kafkaコネクタを介してStarRocksにデータをロードできます。
- Debezium形式のCDCデータなど、データ変換をカスタマイズする。
- 複数のKafkaトピックからデータをロードする。
- Confluent Cloudからデータをロードする。
- ロードバッチサイズ、並列処理、その他のパラメーターについてより細かい制御が必要な場合、ロード速度とリソース利用のバランスを取るため。

## 準備

### Kafka環境のセットアップ

自己管理型のApache KafkaクラスターとConfluent Cloudの両方がサポートされています。

- 自己管理型のApache Kafkaクラスターの場合、Apache KafkaクラスターとKafka Connectクラスターをデプロイし、トピックを作成してください。
- Confluent Cloudの場合、Confluentアカウントを持っており、クラスターとトピックを作成していることを確認してください。

### Kafkaコネクタのインストール

KafkaコネクタをKafka Connectに登録します：

- 自己管理型Kafkaクラスター：

  - [starrocks-kafka-connector-1.0.0.tar.gz](https://releases.starrocks.io/starrocks/starrocks-kafka-connector-1.0.0.tar.gz)をダウンロードして解凍します。
  - 解凍したディレクトリを、`plugin.path` プロパティで指定されたパスにコピーします。`plugin.path` プロパティは、Kafka Connectクラスター内のワーカーノードの設定ファイルで見つけることができます。

- Confluent Cloud：

  > **注記**
  >
  > 現在、KafkaコネクタはConfluent Hubにアップロードされていません。圧縮ファイルをConfluent Cloudにアップロードする必要があります。

### StarRocksテーブルの作成

Kafkaトピックとデータに基づいてStarRocksにテーブルを作成します。

## 例

以下の手順は、自己管理型Kafkaクラスターを例にして、Kafkaコネクタを設定し、Kafka Connectを起動する方法を示しています（Kafkaサービスを再起動する必要はありません）。

1. **connect-StarRocks-sink.properties** という名前のKafkaコネクタ設定ファイルを作成し、パラメータを設定します。パラメータの詳細については、[パラメータ](#parameters)を参照してください。

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
    > ソースデータがCDCデータ（Debezium形式のデータなど）で、StarRocksテーブルがプライマリキーテーブルの場合、ソースデータの変更をプライマリキーテーブルに同期するために[`transform`の設定](#load-debezium-formatted-cdc-data)も必要です。

2. Kafkaコネクタを実行します（Kafkaサービスを再起動する必要はありません）。以下のコマンドで使用されるパラメータと説明については、[Kafkaドキュメント](https://kafka.apache.org/documentation.html#connect_running)を参照してください。

    - スタンドアロンモード

        ```shell
        bin/connect-standalone worker.properties connect-StarRocks-sink.properties [connector2.properties connector3.properties ...]
        ```

    - 分散モード

      > **注記**
      >
      > 本番環境では分散モードを使用することを推奨します。

    - ワーカーを起動します。

        ```shell
        bin/connect-distributed worker.properties
        ```

    - 分散モードでは、コネクタ設定はコマンドラインで渡されません。代わりに、以下で説明するREST APIを使用してKafkaコネクタを設定し、Kafka Connectを実行します。

        ```Shell
        curl -i http://127.0.0.1:8083/connectors -H "Content-Type: application/json" -X POST -d '{
        "name":"starrocks-kafka-connector",
        "config":{
            "connector.class":"com.starrocks.connector.kafka.StarRocksSinkConnector",
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

3. StarRocksテーブルでデータをクエリします。

## パラメータ

| パラメータ                           | 必須 | 既定値                                                | 説明                                                  |
| ----------------------------------- | :---: | ------------------------------------------------------------ | ------------------------------------------------------------ |
| name                                | はい      |                                                              | このKafkaコネクタの名前です。このKafka Connectクラスタ内のすべてのKafkaコネクタ間でグローバルに一意でなければなりません。例：starrocks-kafka-connector。 |
| connector.class                     | はい      | com.starrocks.connector.kafka.StarRocksSinkConnector                  | このKafkaコネクタのシンクに使用されるクラスです。                   |

| topics                              | YES      |                                                              | サブスクライブする1つ以上のトピック。各トピックはStarRocksのテーブルに対応します。デフォルトでは、StarRocksはトピック名がStarRocksテーブル名と一致すると想定します。そのため、トピック名を使用して対象のStarRocksテーブルを決定します。`topics`か`topics.regex`（以下参照）のいずれかを選択してくださいが、両方を指定することはできません。ただし、StarRocksテーブル名がトピック名と異なる場合は、オプションの`starrocks.topic2table.map`パラメータ（以下参照）を使用してトピック名からテーブル名へのマッピングを指定します。 |
| topics.regex                        |          | 正規表現を使用してサブスクライブする1つ以上のトピックに一致させます。詳細は`topics`を参照してください。`topics.regex`か`topics`（上記）のいずれかを選択してくださいが、両方を指定することはできません。 |                                                              |
| starrocks.topic2table.map           | NO       |                                                              | トピック名がStarRocksテーブル名と異なる場合のStarRocksテーブル名とトピック名のマッピング。形式は`<topic-1>:<table-1>,<topic-2>:<table-2>,...`です。 |
| starrocks.http.url                  | YES      |                                                              | StarRocksクラスタのFEのHTTP URL。形式は`<fe_host1>:<fe_http_port1>,<fe_host2>:<fe_http_port2>,...`です。アドレスはコンマ(,)で区切ります。例：`192.168.xxx.xxx:8030,192.168.xxx.xxx:8030`。 |
| starrocks.database.name             | YES      |                                                              | StarRocksデータベースの名前です。                              |
| starrocks.username                  | YES      |                                                      | StarRocksクラスタアカウントのユーザー名。ユーザーにはStarRocksテーブルに対する[INSERT](../sql-reference/sql-statements/account-management/GRANT.md)権限が必要です。 |
| starrocks.password                  | YES      |                                                              | StarRocksクラスタアカウントのパスワードです。              |
| key.converter                       | NO       |           Kafka Connectクラスタで使用されるキーコンバーター                                                           | このパラメータは、シンクコネクタ（Kafka-connector-starrocks）で使用されるキーコンバーターを指定します。これはKafkaデータのキーを逆シリアル化するために使用されます。デフォルトのキーコンバーターはKafka Connectクラスタで使用されるものです。|
| value.converter                     | NO       |    Kafka Connectクラスタで使用される値コンバーター                             | このパラメータは、シンクコネクタ（Kafka-connector-starrocks）で使用される値コンバーターを指定します。これはKafkaデータの値を逆シリアル化するために使用されます。デフォルトの値コンバーターはKafka Connectクラスタで使用されるものです。 |
| key.converter.schema.registry.url   | NO       |                                                              | キーコンバーター用のスキーマレジストリURLです。                   |
| value.converter.schema.registry.url | NO       |                                                              | 値コンバーター用のスキーマレジストリURLです。                 |
| tasks.max                           | NO       | 1                                                            | Kafkaコネクタが作成できるタスクスレッドの最大数で、通常はKafka ConnectクラスタのワーカーノードのCPUコア数と同じです。このパラメータを調整して負荷性能を制御することができます。 |
| bufferflush.maxbytes                | NO       | 94371840(90M)                                                | 一度にStarRocksに送信する前にメモリに蓄積できるデータの最大サイズです。最大値は64MBから10GBの範囲です。Stream Load SDKバッファはデータをバッファリングするために複数のStream Loadジョブを作成する可能性があるため、ここで言及されるしきい値は合計データサイズを指します。 |
| bufferflush.intervalms              | NO       | 300000                                                       | データのバッチを送信する間隔で、ロードレイテンシを制御します。範囲：[1000, 3600000]。 |
| connect.timeoutms                   | NO       | 1000                                                         | HTTP URLへの接続タイムアウトです。範囲：[100, 60000]。 |
| sink.properties.*                   |          |                                                              | Stream Loadパラメータでロード動作を制御します。例えば、`sink.properties.format`パラメータはStream Loadに使用される形式を指定します。サポートされるパラメータとその説明の一覧については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。 |
| sink.properties.format              | NO       | json                                                         | Stream Loadに使用される形式です。Kafkaコネクタは、データの各バッチをStarRocksに送信する前にこの形式に変換します。有効な値は`csv`と`json`です。詳細は、[CSV parameters](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md#csv-parameters)と[JSON parameters](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md#json-parameters)を参照してください。 |

## 制限事項

- Kafkaトピックからの単一メッセージを複数のデータ行にフラット化してStarRocksにロードすることはサポートされていません。
- KafkaコネクタのSinkは少なくとも一度は処理される(at-least-once)セマンティクスを保証します。

## ベストプラクティス

### Debezium形式のCDCデータをロードする

KafkaデータがDebezium CDC形式であり、StarRocksテーブルがプライマリキーテーブルである場合、`transforms`パラメータおよびその他の関連パラメータも設定する必要があります。

```Properties
transforms=addfield,unwrap
transforms.addfield.type=com.starrocks.connector.kafka.transforms.AddOpFieldForDebeziumRecord
transforms.unwrap.type=io.debezium.transforms.ExtractNewRecordState
transforms.unwrap.drop.tombstones=true
transforms.unwrap.delete.handling.mode
```

上記の設定では、`transforms=addfield,unwrap`を指定します。

- `addfield` 変換は、Debezium CDC 形式のデータの各レコードに `__op` フィールドを追加し、StarRocks のプライマリキーモデルテーブルをサポートするために使用されます。StarRocks テーブルがプライマリキーテーブルでない場合、`addfield` 変換を指定する必要はありません。`addfield` 変換クラスは `com.Starrocks.Kafka.Transforms.AddOpFieldForDebeziumRecord` で、Kafka コネクタの JAR ファイルに含まれているため、手動でインストールする必要はありません。
- `unwrap` 変換は Debezium によって提供され、操作タイプに基づいて Debezium の複雑なデータ構造を展開するために使用されます。詳細については、[新しいレコード状態の抽出](https://debezium.io/documentation/reference/stable/transformations/event-flattening.html)を参照してください。
