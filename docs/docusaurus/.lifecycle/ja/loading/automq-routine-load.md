# AutoMQ Kafkaからデータを連続して読み込む

[AutoMQ for Kafka](https://docs.automq.com/docs/automq-s3kafka/YUzOwI7AgiNIgDk1GJAcu6Uanog)は、クラウド環境向けに再設計されたKafkaのクラウドネイティブバージョンです。
AutoMQ Kafkaは[オープンソース](https://github.com/AutoMQ/automq-for-kafka)であり、Kafkaプロトコルと完全に互換性があり、クラウドの利点を十分に活用しています。
自己管理型のApache Kafkaと比較して、AutoMQ Kafkaはクラウドネイティブアーキテクチャを持ち、容量の自動スケーリング、ネットワークトラフィックの自己バランシング、パーティションの瞬時の移動などの機能を提供しています。これらの機能により、ユーザーの総所有コスト（TCO）が大幅に低下します。

本記事では、StarRocks Routine Loadを使用してデータをAutoMQ Kafkaにインポートする方法について説明します。
Routine Loadの基本原則については、Routine Load Fundamentalsのセクションを参照してください。

## 環境の準備

### StarRocksとテストデータの準備

稼働中のStarRocksクラスターがあることを確認してください。デモンストレーション目的で、この記事ではLinuxマシン上にDockerを使用してStarRocksクラスターをインストールするための[デプロイガイド](../quick_start/deploy_with_docker.md)に従います。

以下のプライマリキーモデルを持つデータベースとテストテーブルを作成してください：

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

AutoMQ Kafka環境とテストデータを準備するには、AutoMQ[クイックスタート](https://docs.automq.com/docs/automq-s3kafka/VKpxwOPvciZmjGkHk5hcTz43nde)ガイドに従ってAutoMQ Kafkaクラスターをデプロイしてください。StarRocksが直接AutoMQ Kafkaサーバーに接続できることを確認してください。

AutoMQ Kafkaで`example_topic`というトピックを素早く作成し、テストJSONデータを書き込むために、以下の手順に従ってください：

### トピックの作成

Kafkaのコマンドラインツールを使用してトピックを作成してください。Kafka環境にアクセス権があり、Kafkaサービスが実行されていることを確認してください。
以下はトピックを作成するためのコマンドです：

```shell
./kafka-topics.sh --create --topic example_topic --bootstrap-server 10.0.96.4:9092 --partitions 1 --replication-factor 1
```

> 注：`topic`と`bootstrap-server`をKafkaサーバーアドレスに置き換えてください。

トピック作成の結果をチェックするには、次のコマンドを使用してください：

```shell
./kafka-topics.sh --describe example_topic --bootstrap-server 10.0.96.4:9092
```

### テストデータの生成

シンプルなJSON形式のテストデータを生成してください

```json
{
  "id": 1,
  "name": "testuser",
  "timestamp": "2023-11-10T12:00:00",
  "status": "active"
}
```

### テストデータの書き込み

Kafkaのコマンドラインツールまたはプログラミング方法を使用して、`example_topic`にテストデータを書き込んでください。以下はコマンドラインツールを使用した例です：

```shell
echo '{"id": 1, "name": "testuser", "timestamp": "2023-11-10T12:00:00", "status": "active"}' | sh kafka-console-producer.sh --broker-list 10.0.96.4:9092 --topic example_topic
```

> 注：`topic`と`bootstrap-server`をKafkaサーバーアドレスに置き換えてください。

最近書き込まれたトピックデータを表示するには、以下のコマンドを使用してください：

```shell
sh kafka-console-consumer.sh --bootstrap-server 10.0.96.4:9092 --topic example_topic --from-beginning
```

## ルーチンロードタスクの作成

StarRocksコマンドラインで、AutoMQ Kafkaトピックからデータを連続してインポートするために、ルーチンロードタスクを作成してください：

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

> 注：`kafka_broker_list`をKafkaサーバーアドレスに置き換えてください。

### パラメータの説明

#### データの形式

PROPERTIES句の"format" = "json"でデータ形式をJSONとして指定してください。

#### データの抽出と変換

ソースデータとターゲットテーブルの間のマッピングおよび変換関係を指定するには、COLUMNSおよびjsonpathsパラメータを構成してください。COLUMNS内の列名は、ターゲットテーブルの列名に対応し、その順序はソースデータの列の順序に対応します。jsonpathsパラメータは、JSONデータから必要なフィールドデータを抽出するために使用され、新しく生成されたCSVデータに類似します。次に、COLUMNSパラメータは、jsonpaths内のフィールドに一時的に名前を付けます。データ変換の詳細については、[Import中のデータ変換](./Etl_in_loading.md)を参照してください。
> 注：各JSONオブジェクトごとに1行に1つのキー名と数量（順序は必要ありません）が対応し、これがターゲットテーブルの列に対応している場合は、COLUMNSを構成する必要はありません。

## データインポートの検証

まず、Routine Loadのインポートジョブを確認し、Routine Loadのインポートタスクのステータスが実行中であることを確認してください。

```sql
show routine load\G;
```

その後、StarRocksデータベース内の対応するテーブルをクエリし、データが正常にインポートされていることを確認できます。

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