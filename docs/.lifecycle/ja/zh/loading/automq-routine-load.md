# AutoMQ Kafka からの継続的なデータインポート

[AutoMQ for Kafka](https://docs.automq.com/zh/docs/automq-s3kafka/YUzOwI7AgiNIgDk1GJAcu6Uanog)(以下、AutoMQ Kafka)は、クラウドネイティブな設計に基づいたKafkaです。
AutoMQ Kafkaは[オープンソースのコア](https://github.com/AutoMQ/automq-for-kafka)を持ち、Kafkaプロトコルと100%互換性があり、クラウドのメリットを最大限に活用できます。
自己管理型のApache Kafkaと比較して、AutoMQ Kafkaはクラウドネイティブアーキテクチャに基づいて自動スケーリング、トラフィックの自動バランス、秒単位のパーティション移動などの機能を実現し、ユーザーにとってより低い総所有コスト（TCO）をもたらします。
この記事では、StarRocksのRoutine Loadを使用してAutoMQ Kafkaにデータをインポートする方法について説明します。Routine Loadの基本原則については、[Routine Loadの基本原則](https://docs.starrocks.io/zh/docs/loading/load_concept/strict_mode/#routine-load)を参照してください。

## 環境準備

### StarRocksおよびテストデータの準備

使用可能なStarRocksクラスタが準備されていることを確認してください。デモンストレーションを容易にするために、[Dockerを使用したStarRocksのデプロイ](../quick_start/deploy_with_docker.md)を参考にして、Linuxマシン上にデモ用のStarRocksクラスタをインストールしました。
データベースとプライマリキーモデルのテストテーブルを作成します:

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

### AutoMQ Kafka 環境とテストデータの準備

AutoMQの[クイックスタート](https://docs.automq.com/zh/docs/automq-s3kafka/VKpxwOPvciZmjGkHk5hcTz43nde)を参考にAutoMQ Kafkaクラスタをデプロイし、AutoMQ KafkaがStarRocksとネットワーク接続を維持していることを確認します。
AutoMQ Kafkaでexample_topicというトピックを素早く作成し、テスト用のJSONデータを書き込むには、以下の手順に従ってください:

#### **トピックの作成**

Kafkaのコマンドラインツールを使用してトピックを作成します。Kafka環境へのアクセス権があり、Kafkaサービスが実行中であることを確認してください。以下はトピックを作成するコマンドです：

```bash
./kafka-topics.sh --create --topic example_topic --bootstrap-server 10.0.96.4:9092  --partitions 1 --replication-factor 1
```

> 注意：topicとbootstrap-serverをあなたのKafkaサーバーのアドレスに置き換えてください。

トピックを作成した後、以下のコマンドでトピックの作成結果を確認できます

```bash
./kafka-topics.sh --describe --topic example_topic --bootstrap-server 10.0.96.4:9092
```

#### **テストデータの生成**

以下は、前述のテーブルに対応するシンプルなJSON形式のテストデータを生成する例です:

```json
{
  "id": 1,
  "name": "testuser",
  "timestamp": "2023-11-10T12:00:00",
  "status": "active"
}
```

#### **テストデータの書き込み**

Kafkaのコマンドラインツールまたはプログラムを使用して、テストデータをexample_topicに書き込みます。以下はコマンドラインツールを使用した例です：

```bash
echo '{"id": 1, "name": "testuser", "timestamp": "2023-11-10T12:00:00", "status": "active"}' | ./kafka-console-producer.sh --broker-list 10.0.96.4:9092 --topic example_topic
```

> 注意：topicとbootstrap-serverをあなたのKafkaサーバーのアドレスに置き換えてください。

以下のコマンドを使用して、書き込まれたばかりのトピックデータを確認できます:

```bash
./kafka-console-consumer.sh --bootstrap-server 10.0.96.4:9092 --topic example_topic --from-beginning
```

## Routine Load インポートジョブの作成

StarRocksのコマンドラインでRoutine Loadインポートジョブを作成し、AutoMQ Kafkaトピック内のデータを継続的にインポートできます:

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

> 注意：kafka_broker_listをあなたのKafkaサーバーのアドレスに置き換えてください。

### パラメータ説明

#### **データフォーマット**

PROPERTIES節の"format" = "json"でデータフォーマットをJSONとして指定する必要があります。

#### **データ抽出と変換**

ソースデータとターゲットテーブル間の列のマッピングと変換関係を指定する必要がある場合、COLUMNSとjsonpathsパラメータを設定できます。
COLUMNS内の列名は**ターゲットテーブル**の列名に対応し、列の順序は**ソースデータ**内の列の順序に対応します。jsonpathsパラメータは、必要なフィールドデータをJSONデータから抽出するために使用されます。それは新しく生成されたCSVデータのようなものです。
その後、COLUMNSパラメータはjsonpaths内のフィールドに**順番に**一時的な名前を付けます。
データ変換の詳細については、[データ変換の実装](./Etl_in_loading.md)を参照してください。
> 注意：JSONオブジェクトごとに1行で、キーの名前と数（順序は必要ありません）がターゲットテーブルの列に対応している場合、COLUMNSを設定する必要はありません。

## データインポートの検証

まず、Routine Loadインポートジョブの状態を確認し、Routine Loadインポートタスクの状態がRUNNINGであることを確認します：

```sql
SHOW ROUTINE LOAD\G;
```

次に、StarRocksデータベース内の対応するテーブルをクエリすると、データが正常にインポートされていることがわかります：

```sql
SELECT * FROM users;
+------+--------------+---------------------+--------+
| id   | name         | timestamp           | status |
+------+--------------+---------------------+--------+
|    1 | testuser     | 2023-11-10T12:00:00 | active |
|    2 | testuser     | 2023-11-10T12:00:00 | active |
+------+--------------+---------------------+--------+
2 rows in set (0.01 sec)
```
