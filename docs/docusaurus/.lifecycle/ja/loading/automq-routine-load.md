# AutoMQ Kafkaからデータを連続的にロードする

[AutoMQ for Kafka](https://docs.automq.com/docs/automq-s3kafka/YUzOwI7AgiNIgDk1GJAcu6Uanog) は、クラウド環境向けに再設計されたKafkaのクラウドネイティブバージョンです。
AutoMQ Kafka は [オープンソース](https://github.com/AutoMQ/automq-for-kafka) であり、Kafkaプロトコルと完全な互換性があり、クラウドの恩恵を十分に活用しています。
セルフマネージドのApache Kafkaと比較して、クラウドネイティブなアーキテクチャを持つAutoMQ Kafkaには、容量の自動スケーリング、ネットワークトラフィックの自己バランシング、秒単位でのパーティションの移動などの機能があります。これらの機能により、ユーザーの総所有コスト（TCO）が大幅に低下します。

この記事では、StarRocks Routine Loadを使用してデータをAutoMQ Kafkaにインポートする方法について説明します。
ルーチンロードの基本原則については、「ルーチンロードの基礎」のセクションを参照してください。

## 環境の準備

### StarRocksとテストデータの準備

動作しているStarRocksクラスターがあることを確認してください。デモの目的で、この記事ではLinuxマシン上にDockerを使用してStarRocksクラスターをインストールする [展開ガイド](../quick_start/deploy_with_docker.md) に従います。

プライマリキーモデルを持つデータベースとテストテーブルの作成:

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

AutoMQ Kafka環境とテストデータを準備するために、AutoMQ [クイックスタート](https://docs.automq.com/docs/automq-s3kafka/VKpxwOPvciZmjGkHk5hcTz43nde) ガイドに従ってAutoMQ Kafkaクラスターを展開します。StarRocksがAutoMQ Kafkaサーバーに直接接続できることを確認してください。

AutoMQ Kafkaで `example_topic` というトピックをすばやく作成して、テストJSONデータを書き込むには、次の手順に従います:

### トピックの作成

Kafkaのコマンドラインツールを使用してトピックを作成してください。Kafka環境にアクセスでき、Kafkaサービスが実行されていることを確認してください。
以下はトピックを作成するためのコマンドです:

```shell
./kafka-topics.sh --create --topic example_topic --bootstrap-server 10.0.96.4:9092 --partitions 1 --replication-factor 1
```

> 注意: `topic` と `bootstrap-server` をKafkaサーバーアドレスに置き換えてください。

トピック作成の結果を確認するには、次のコマンドを使用します:

```shell
./kafka-topics.sh --describe example_topic --bootstrap-server 10.0.96.4:9092
```

### テストデータの生成

シンプルなJSON形式のテストデータを生成します

```json
{
  "id": 1,
  "name": "testuser",
  "timestamp": "2023-11-10T12:00:00",
  "status": "active"
}
```

### テストデータの書き込み

コマンドラインツールまたはプログラミング方法を使用して、 `example_topic` にテストデータを書き込みます。以下はコマンドラインツールを使用した例です:

```shell
echo '{"id": 1, "name": "testuser", "timestamp": "2023-11-10T12:00:00", "status": "active"}' | sh kafka-console-producer.sh --broker-list 10.0.96.4:9092 --topic example_topic
```

> 注意: `topic` と `bootstrap-server` をKafkaサーバーアドレスに置き換えてください。

最近書き込まれたトピックデータを表示するには、次のコマンドを使用します:

```shell
sh kafka-console-consumer.sh --bootstrap-server 10.0.96.4:9092 --topic example_topic --from-beginning
```

## ルーチンロードタスクの作成

StarRocksコマンドラインで、AutoMQ Kafkaトピックからデータを連続的にインポートするためのルーチンロードタスクを作成します:

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

> 注意: `kafka_broker_list` をKafkaサーバーアドレスに置き換えてください。

### パラメータの説明

#### データフォーマット

PROPERTIES句の "format" = "json" でデータ形式をJSONとして指定します。

#### データの抽出と変換

ソースデータとターゲットテーブルのマッピングおよび変換関係を指定するには、COLUMNSおよびjsonpathsパラメータを設定します。 COLUMNSの列名は、ターゲットテーブルの列名に対応し、その順序はソースデータの列の順序に対応します。 jsonpathsパラメータは、JSONデータから必要なフィールドデータを抽出するために使用され、新しく生成されたCSVデータと同様です。次に、COLUMNSパラメータは、jsonpaths内のフィールドに一時的に名前を付けます。 データ変換の詳細については、[インポート時のデータ変換](./Etl_in_loading.md) を参照してください。
> 注意: 各行ごとのJSONオブジェクトに、ターゲットテーブルの列に対応するキー名と数量（順序は必要ありません）がある場合は、COLUMNSを構成する必要はありません。

## データインポートの検証

まず、Routine Loadのインポートジョブをチェックし、Routine Loadのインポートタスクのステータスが実行中であることを確認します。

```sql
show routine load\G;
```

次に、StarRocksデータベースで対応するテーブルをクエリし、データが正常にインポートされたことを確認できます。

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