# AutoMQ Kafkaからデータを連続的に読み込む.md

[AutoMQ for Kafka](https://docs.automq.com/zh/docs/automq-s3kafka/YUzOwI7AgiNIgDk1GJAcu6Uanog)は、クラウド環境向けに再設計されたKafkaのクラウドネイティブバージョンです。
AutoMQ Kafkaは[オープンソース](https://github.com/AutoMQ/automq-for-kafka)であり、Kafkaプロトコルと完全に互換性があり、クラウドの利点を最大限に活用しています。
セルフマネージドのApache Kafkaと比較して、クラウドネイティブアーキテクチャを持つAutoMQ Kafkaは、容量の自動スケーリング、ネットワークトラフィックの自己バランシング、パーティションの秒単位での移動などの機能を提供します。これらの機能により、ユーザーの総所有コスト（TCO）が大幅に低下します。

この記事では、StarRocks Routine Loadを使用してデータをAutoMQ Kafkaにインポートする方法について説明します。
Routine Loadの基本原則については、「Routine Loadの基礎」のセクションを参照してください。

## 環境の準備

### StarRocksとテストデータの準備

実行中のStarRocksクラスタがあることを確認してください。デモンストレーション目的で、この記事では[デプロイメントガイド](https://docs.starrocks.io/zh/docs/3.0/quick_start/deploy_with_docker/)に従って、Linuxマシン上のDockerを使用してStarRocksクラスタをインストールします。

プライマリキーモデルを使用してデータベースとテストテーブルを作成します。

```sql
create database automq_db;
create table users (
  id bigint NOT NULL,
  name string NOT NULL,
  timestamp string NULL,
  status string NULL
) PRIMARY KEY (id)
DISTRIBUTED BY HASH(id)
PROPERTIES (
  "replication_num" = "1",
  "enable_persistent_index" = "true"
);
```

## AutoMQ Kafkaとテストデータの準備

AutoMQ Kafka環境とテストデータを準備するには、AutoMQ [クイックスタート](https://docs.automq.com/docs/automq-s3kafka/VKpxwOPvciZmjGkHk5hcTz43nde)ガイドに従ってAutoMQ Kafkaクラスタをデプロイします。StarRocksがAutoMQ Kafkaサーバーに直接接続できることを確認してください。

AutoMQ Kafkaで`example_topic`というトピックを作成し、テストJSONデータを書き込むには、次の手順に従ってください。

### トピックの作成

Kafkaのコマンドラインツールを使用してトピックを作成します。Kafkaの環境にアクセスでき、Kafkaサービスが実行されていることを確認してください。
以下はトピックを作成するためのコマンドです。

```shell
./kafka-topics.sh --create --topic example_topic --bootstrap-server 10.0.96.4:9092 --partitions 1 --replication-factor 1
```

> 注意: `topic`と`bootstrap-server`をKafkaサーバーアドレスに置き換えてください。

トピック作成の結果を確認するには、次のコマンドを使用します。

```shell
./kafka-topics.sh --describe example_topic --bootstrap-server 10.0.96.4:9092
```

### テストデータの生成

単純なJSON形式のテストデータを生成します。

```json
{
  "id": 1,
  "name": "testuser",
  "timestamp": "2023-11-10T12:00:00",
  "status": "active"
}
```

### テストデータの書き込み

Kafkaのコマンドラインツールまたはプログラミング方法を使用して、テストデータをexample_topicに書き込みます。以下はコマンドラインツールを使用した例です。

```shell
echo '{"id": 1, "name": "testuser", "timestamp": "2023-11-10T12:00:00", "status": "active"}' | sh kafka-console-producer.sh --broker-list 10.0.96.4:9092 --topic example_topic
```

> 注意: `topic`と`bootstrap-server`をKafkaサーバーアドレスに置き換えてください。

最近書き込まれたトピックデータを表示するには、次のコマンドを使用します。

```shell
sh kafka-console-consumer.sh --bootstrap-server 10.0.96.4:9092 --topic example_topic --from-beginning
```

## ルーチンロードタスクの作成

StarRocksコマンドラインで、AutoMQ Kafkaトピックからデータを連続的にインポートするためのルーチンロードタスクを作成します。

```sql
CREATE ROUTINE LOAD automq_example_load ON users
COLUMNS(id, name, timestamp, status)
PROPERTIES
(
  "desired_concurrent_number" = "5",
  "format" = "json",
  "jsonpaths" = "[\"$.id\",\"$.name\",\"$.timestamp\",\"$.status\"]"
)
FROM KAFKA
(
  "kafka_broker_list" = "10.0.96.4:9092",
  "kafka_topic" = "example_topic",
  "kafka_partitions" = "0",
  "property.kafka_default_offsets" = "OFFSET_BEGINNING"
);
```

> 注意: `kafka_broker_list`をKafkaサーバーアドレスに置き換えてください。

### パラメータの説明

#### データ形式

PROPERTIES句の"format" = "json"でデータ形式をJSONとして指定します。

#### データの抽出と変換

ソースデータとターゲットテーブルの間のマッピングと変換の関係を指定するには、COLUMNSおよびjsonpathsパラメータを設定します。COLUMNSの列名は、ターゲットテーブルの列名に対応し、その順序はソースデータの列順序に対応します。jsonpathsパラメータは、JSONデータから必要なフィールドデータを抽出するために使用され、新しく生成されたCSVデータに類似します。その後、COLUMNSパラメータは、jsonpaths内のフィールドに一時的に名前を付けます。データ変換の詳細については、[インポート時のデータ変換](https://docs.starrocks.io/zh/docs/3.0/loading/Etl_in_loading/)を参照してください。
> 注意: 各JSONオブジェクトごとにキー名と数量（順序は必要ありません）がターゲットテーブルの列に対応している場合、COLUMNSを設定する必要はありません。

## データインポートの検証

まず、Routine Loadのインポートジョブを確認し、Routine LoadのインポートタスクのステータスがRUNNINGであることを確認します。

```sql
show routine load\G;
```

次に、StarRocksデータベースの対応するテーブルをクエリし、データが正常にインポートされたことを確認できます。

```sql
StarRocks > select * from users;
+------+--------------+---------------------+--------+
| id   | name         | timestamp           | status |
+------+--------------+---------------------+--------+
|    1 | testuser     | 2023-11-10T12:00:00 | active |
|    2 | testuser     | 2023-11-10T12:00:00 | active |
+------+--------------+---------------------+--------+
2 rows in set (0.01 sec)
```
