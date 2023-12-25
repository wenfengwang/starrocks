---
displayed_sidebar: Chinese
---

# Kafka connector を使用したデータのインポート

StarRocks は Apache Kafka® 用のコネクタ（StarRocks Connector for Apache Kafka®）を提供し、Kafka のメッセージを継続的に消費して StarRocks にインポートします。

Kafka connector を使用すると、Kafka エコシステムにより良く統合され、StarRocks は Kafka Connect とシームレスに連携できます。StarRocks の準リアルタイム接続ルートにより多くの選択肢を提供します。Routine Load と比較して、以下のシナリオで Kafka connector を使用してデータをインポートすることを優先的に検討できます：

- Routine Load は CSV、JSON、Avro 形式のデータのみをインポートするのに対し、Kafka connector はより豊富なデータ形式をインポートできます。データが Kafka Connect の converters を介して JSON および CSV 形式に変換できる場合、Kafka connector を介してインポートできます。例えば、Protobuf 形式のデータです。
- Debezium CDC 形式のデータなど、データにカスタムの transform 操作を行う必要がある場合。
- 複数の Kafka Topic からデータをインポートする場合。
- Confluent cloud からデータをインポートする場合。
- インポートのバッチサイズ、並列度などのパラメータをより細かく制御し、インポート速度とリソース使用のバランスを取る必要がある場合。

## 環境準備

### Kafka 環境の準備

自己構築の Apache Kafka クラスターと Confluent cloud に対応しています：

- 自己構築の Apache Kafka クラスターを使用する場合は、Apache Kafka クラスターと Kafka Connect クラスターがデプロイされており、Topic が作成されていることを確認してください。
- Confluent cloud を使用する場合は、Confluent アカウントを保有しており、クラスターと Topic が作成されていることを確認してください。

### Kafka connector のインストール

Kafka connector を Kafka Connect にインストールします。

- 自己構築の Kafka クラスター

  - 圧縮ファイル [starrocks-kafka-connector-1.0.0.tar.gz](https://releases.starrocks.io/starrocks/starrocks-kafka-connector-1.0.0.tar.gz) をダウンロードして解凍します。
  - 解凍したディレクトリを Kafka Connect クラスターの worker ノードの設定ファイルに含まれる `plugin.path` プロパティが指すパスにコピーします。
- Confluent cloud

  > **説明**
  >
  > Kafka connector はまだ Confluent Hub にアップロードされていません。圧縮ファイルを Confluent cloud にアップロードする必要があります。

### StarRocks テーブルの作成

Kafka Topic と StarRocks 内のデータに基づいて対応するテーブルを作成する必要があります。

## 使用例

ここでは、自己構築の Kafka クラスターを例に、Kafka connector を設定し、Kafka Connect サービスを起動して（Kafka サービスを再起動する必要はありません）、StarRocks にデータをインポートする方法について説明します。

1. Kafka connector の設定ファイル **connect-StarRocks-sink.properties** を作成し、対応するパラメータを設定します。パラメータとその説明については、[パラメータ説明](#パラメータ説明)を参照してください。

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
    > ソースデータが CDC データ（例：Debezium CDC 形式）で、StarRocks のテーブルがプライマリキーモデルの場合、ソースのデータ変更をプライマリキーモデルのテーブルに同期させるために、[transforms および関連パラメータの設定](#Debezium-CDC-形式データのインポート)が必要です。

2. Kafka Connect サービスを起動します（Kafka サービスを再起動する必要はありません）。コマンドのパラメータについては、[Running Kafka Connect](https://kafka.apache.org/documentation.html#connect_running)を参照してください。

   1. スタンドアロンモード

      ```Shell
      bin/connect-standalone worker.properties connect-StarRocks-sink.properties [connector2.properties connector3.properties ...]
      ```

   2. 分散モード

      > **説明**
      >
      > 本番環境では分散モードの使用をお勧めします。

      1. worker を起動します：

          ```Shell
          bin/connect-distributed worker.properties
          ```

      2. 注意：分散モードでは、コマンドラインで Kafka connector の設定を行うことはできません。REST API を呼び出して Kafka connector を設定し、Kafka Connect を起動する必要があります：

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

3. StarRocks のテーブル内のデータをクエリします。

## **パラメータ説明**

| パラメータ                              | 必須 | デフォルト値                                                 | 説明                                                         |
| ----------------------------------- | ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| name                                | はい |                                                              | 現在の Kafka connector を表し、Kafka Connect クラスター内でグローバルに一意でなければなりません。例：starrocks-kafka-connector。 |
| connector.class                     | はい | com.starrocks.connector.kafka.SinkConnector                  | Kafka connector の sink に使用されるクラス。                 |
| topics                              | はい |                                                              | 一つまたは複数の購読対象 Topic。各 Topic は StarRocks のテーブルに対応し、デフォルトでは Topic 名が StarRocks テーブル名と一致し、インポート時に Topic 名に基づいて対象 StarRocks テーブルが決定されます。`topics` と `topics.regex`（以下参照）のどちらか一方を記入します。一致しない場合は `starrocks.topic2table.map` を設定する必要があります。 |
| topics.regex                        |      | 購読対象 Topic にマッチする正規表現。`topics` と同様の説明。`topics` と `topics.regex` のどちらか一方を記入します（上記参照）。 |                                                              |
| starrocks.topic2table.map           | いいえ |                                                              | Topic 名が StarRocks テーブル名と一致しない場合、この設定項目でマッピング関係を指定できます。形式は `<topic-1>:<table-1>,<topic-2>:<table-2>,...` です。 |
| starrocks.http.url                  | はい |                                                              | FE の HTTP Server アドレス。形式は `<fe_host1>:<fe_http_port1>,<fe_host2>:<fe_http_port2>,...`。複数のアドレスは、英語のコンマ (,) で区切ります。例：`192.168.xxx.xxx:8030,192.168.xxx.xxx:8030`。 |
| starrocks.database.name             | はい |                                                              | StarRocks の対象データベース名。                             |
| starrocks.username                  | はい |                                                              | StarRocks のユーザー名。対象テーブルに INSERT 権限が必要です。 |
| starrocks.password                  | はい |                                                              | StarRocks のユーザーパスワード。                             |
| key.converter                       | いいえ | Kafka Connect クラスターの key converter                      | このシナリオでの sink connector（Kafka-connector-starrocks）の key converter で、Kafka データの key を逆シリアライズするために使用されます。デフォルトは Kafka Connect クラスターの key converter ですが、カスタム設定も可能です。 |
| value.converter                     | いいえ | Kafka Connect クラスターの value converter                    | sink connector の value converter で、Kafka データの value を逆シリアライズするために使用されます。デフォルトは Kafka Connect クラスターの value converter ですが、カスタム設定も可能です。 |
| key.converter.schema.registry.url   | いいえ |                                                              | key converter に対応する schema registry のアドレス。        |
| value.converter.schema.registry.url | いいえ |                                                              | value converter に対応する schema registry のアドレス。      |
| tasks.max                           | いいえ | 1                                                            | 作成する Kafka connector の task スレッドの最大数。通常は Kafka Connect クラスター内の worker ノードの CPU コア数と同じです。パフォーマンスを向上させる必要がある場合は、このパラメータを調整できます。 |
| bufferflush.maxbytes                | いいえ | 94371840(90M)                                                | データをバッチ処理するサイズで、この閾値に達すると Stream Load を介して StarRocks にデータが一括書き込まれます。値の範囲は [64MB, 10GB] です。Stream Load SDK buffer は複数の Stream Load を起動してデータをバッファリングする可能性があるため、ここでの閾値は総データ量のサイズを指します。 |
| bufferflush.intervalms              | いいえ | 300000                                                       | データをバッチ処理して送信する間隔で、StarRocks へのデータ書き込みの遅延を制御するために使用されます。値の範囲は [1000, 3600000] です。 |
| connect.timeoutms                   | いいえ | 1000                                                         | HTTP-URL への接続タイムアウト時間。値の範囲は [100, 60000] です。 |
| sink.properties.*                   |      |                                                              | Stream Load のパラメータを指定してインポート動作を制御します。例えば `sink.properties.format` を使用してインポートデータの形式を CSV または JSON に指定します。詳細なパラメータと説明については、[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。 |
| sink.properties.format              | いいえ | JSON                                                         | Stream Load でのインポート時のデータ形式。CSV または JSON のいずれかを取ることができます。デフォルトは JSON です。詳細なパラメータ説明については、[CSV 适用参数](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md#csv-适用参数)と [JSON 适用参数](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md#json-适用参数)を参照してください。 |

## 使用制限

- Kafka topic 内の一つのメッセージを複数のレコードに展開して StarRocks にインポートすることはサポートされていません。
- Kafka Connector の Sink は少なくとも一度のセマンティクスを保証します。

## ベストプラクティス

### Debezium CDC 形式データのインポート

Kafka データが Debezium CDC 形式で、StarRocks のテーブルがプライマリキーモデルである場合、Kafka connector の設定ファイル **connect-StarRocks-sink.properties** に[基本パラメータを設定](#使用例)するだけでなく、`transforms` および関連パラメータも設定する必要があります。

```Properties
transforms=addfield,unwrap
transforms.addfield.type=com.starrocks.connector.kafka.transforms.AddOpFieldForDebeziumRecord
transforms.unwrap.type=io.debezium.transforms.ExtractNewRecordState
transforms.unwrap.drop.tombstones=true
transforms.unwrap.delete.handling.mode=rewrite
```

上記の設定では `transforms=addfield,unwrap` を指定しています。

- addfield transform は、Debezium CDC 形式のデータの各レコードに __op フィールドを追加して、[プライマリキーモデルのテーブル](../table_design/table_types/primary_key_table.md)をサポートするために使用されます。StarRocks のテーブルがプライマリキーモデルでない場合は、addfield 変換を指定する必要はありません。addfield transform のクラスは com.starrocks.connector.kafka.transforms.AddOpFieldForDebeziumRecord で、Kafka connector の JAR ファイルに含まれているため、手動でインストールする必要はありません。
- unwrap transform は、Debezium が提供するもので、操作タイプに基づいて Debezium の複雑なデータ構造を展開することができます。詳細については、[New Record State Extraction](https://debezium.io/documentation/reference/stable/transformations/event-flattening.html)を参照してください。
