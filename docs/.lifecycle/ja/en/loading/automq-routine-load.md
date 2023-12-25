# AutoMQ Kafkaからデータを継続的にロードする

[AutoMQ for Kafka](https://docs.automq.com/docs/automq-s3kafka/YUzOwI7AgiNIgDk1GJAcu6Uanog)は、クラウド環境向けに再設計されたKafkaのクラウドネイティブバージョンです。
AutoMQ Kafkaは[オープンソース](https://github.com/AutoMQ/automq-for-kafka)であり、Kafkaプロトコルと完全に互換性があり、クラウドの利点を十分に活用しています。
自己管理型のApache Kafkaと比較して、クラウドネイティブアーキテクチャのAutoMQ Kafkaは、容量の自動スケーリング、ネットワークトラフィックの自己バランシング、数秒でのパーティションの移動などの機能を提供します。これらの機能は、ユーザーの総所有コスト（TCO）を大幅に削減します。

この記事では、StarRocks Routine Loadを使用してAutoMQ Kafkaにデータをインポートする方法について説明します。
Routine Loadの基本原則については、Routine Loadの基本に関するセクションを参照してください。

## 環境の準備

### StarRocksとテストデータの準備

実行中のStarRocksクラスタがあることを確認します。デモンストレーションの目的で、この記事では[デプロイガイド](../quick_start/deploy_with_docker.md)に従って、Dockerを介してLinuxマシンにStarRocksクラスタをインストールします。

主キーモデルを使用したデータベースとテストテーブルの作成:

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

AutoMQ Kafka環境とテストデータを準備するには、AutoMQ [クイックスタート](https://docs.automq.com/docs/automq-s3kafka/VKpxwOPvciZmjGkHk5hcTz43nde)ガイドに従ってAutoMQ Kafkaクラスタをデプロイします。StarRocksがAutoMQ Kafkaサーバに直接接続できることを確認します。

AutoMQ Kafkaで`example_topic`というトピックをすばやく作成し、テストJSONデータを書き込むには、次の手順に従います。

### トピックの作成

Kafkaのコマンドラインツールを使用してトピックを作成します。Kafka環境にアクセスでき、Kafkaサービスが実行されていることを確認します。
トピックを作成するコマンドは次のとおりです。

```shell
./kafka-topics.sh --create --topic example_topic --bootstrap-server 10.0.96.4:9092 --partitions 1 --replication-factor 1
```

> 注: `topic`と`bootstrap-server`をあなたのKafkaサーバアドレスに置き換えてください。

トピック作成の結果を確認するには、このコマンドを使用します：

```shell
./kafka-topics.sh --describe --topic example_topic --bootstrap-server 10.0.96.4:9092
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

Kafkaのコマンドラインツールまたはプログラミングメソッドを使用して、テストデータを`example_topic`に書き込みます。こちらはコマンドラインツールを使用した例です：

```shell
echo '{"id": 1, "name": "testuser", "timestamp": "2023-11-10T12:00:00", "status": "active"}' | ./kafka-console-producer.sh --broker-list 10.0.96.4:9092 --topic example_topic
```

> 注: `topic`と`bootstrap-server`をあなたのKafkaサーバアドレスに置き換えてください。

最近書き込まれたトピックデータを表示するには、以下のコマンドを使用します：

```shell
./kafka-console-consumer.sh --bootstrap-server 10.0.96.4:9092 --topic example_topic --from-beginning
```

## Routine Loadタスクの作成

StarRocksコマンドラインで、AutoMQ Kafkaトピックからデータを継続的にインポートするRoutine Loadタスクを作成します。

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

> 注: `kafka_broker_list`を実際のKafkaサーバアドレスに置き換えてください。

### パラメータの説明

#### データ形式

PROPERTIES句の`"format" = "json"`でデータ形式をJSONとして指定します。

#### データの抽出と変換

ソースデータとターゲットテーブルの間のマッピングおよび変換関係を指定するには、COLUMNSとjsonpathsパラメータを設定します。COLUMNSの列名はターゲットテーブルの列名に対応し、その順序はソースデータの列の順序に対応します。jsonpathsパラメータは、新しく生成されたCSVデータと同様に、JSONデータから必要なフィールドデータを抽出するために使用されます。次に、COLUMNSパラメータはjsonpaths内のフィールドに順番に一時的に名前を付けます。データ変換の詳細については、[インポート時のデータ変換](./Etl_in_loading.md)を参照してください。
> 注: 行ごとに1つのJSONオブジェクトがあり、ターゲットテーブルの列に対応するキー名と数量（順序は必要ありません）がある場合、COLUMNSを設定する必要はありません。

## データインポートの検証

まず、Routine Loadインポートジョブをチェックし、Routine LoadインポートタスクのステータスがRUNNING状態であることを確認します。

```sql
SHOW ROUTINE LOAD\G;
```

次に、StarRocksデータベース内の対応するテーブルをクエリすると、データが正常にインポートされていることが確認できます。

```sql
StarRocks> SELECT * FROM users;
+------+--------------+---------------------+--------+
| id   | name         | timestamp           | status |
+------+--------------+---------------------+--------+
|    1 | testuser     | 2023-11-10T12:00:00 | active |
|    2 | testuser     | 2023-11-10T12:00:00 | active |
+------+--------------+---------------------+--------+
2 rows in set (0.01 sec)
```
