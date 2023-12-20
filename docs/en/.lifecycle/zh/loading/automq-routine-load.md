
# 持续从 AutoMQ Kafka 加载数据

[AutoMQ for Kafka](https://docs.automq.com/docs/automq-s3kafka/YUzOwI7AgiNIgDk1GJAcu6Uanog) 是为云环境重新设计的 Kafka 的云原生版本。
AutoMQ Kafka 是 [open source](https://github.com/AutoMQ/automq-for-kafka) 并且与 Kafka 协议完全兼容，充分利用了云计算的优势。
与自托管的 Apache Kafka 相比，AutoMQ Kafka 以其云原生架构提供了如容量自动扩展、网络流量自平衡、秒级移动分区等特性。这些特性有助于显著降低用户的总体拥有成本（TCO）。

本文将指导您通过 StarRocks 的 Routine Load 功能将数据导入 AutoMQ Kafka。要了解 Routine Load 的基本原理，请参考 Routine Load 基础知识部分。

## 准备环境

### 准备 StarRocks 和测试数据

确保您有一个运行中的 StarRocks 集群。为了演示，本文将遵循[部署指南](../quick_start/deploy_with_docker.md)通过 Docker 在 Linux 机器上安装 StarRocks 集群。

创建一个数据库和带有主键模型的测试表：

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

## 准备 AutoMQ Kafka 和测试数据

按照 AutoMQ 的[快速入门](https://docs.automq.com/docs/automq-s3kafka/VKpxwOPvciZmjGkHk5hcTz43nde)指南准备您的 AutoMQ Kafka 环境和测试数据。确保 StarRocks 能够直接连接到您的 AutoMQ Kafka 服务器。

按照以下步骤在 AutoMQ Kafka 中快速创建一个名为 `example_topic` 的主题，并写入测试 JSON 数据：

### 创建主题

使用 Kafka 的命令行工具创建主题。请确保您可以访问 Kafka 环境且 Kafka 服务正在运行。
这是创建主题的命令：

```shell
./kafka-topics.sh --create --topic example_topic --bootstrap-server 10.0.96.4:9092 --partitions 1 --replication-factor 1
```

> 注意：请将 `topic` 和 `bootstrap-server` 替换为您的 Kafka 服务器地址。

要检查主题创建的结果，使用此命令：

```shell
./kafka-topics.sh --describe --topic example_topic --bootstrap-server 10.0.96.4:9092
```

### 生成测试数据

生成一个简单的 JSON 格式测试数据：

```json
{
  "id": 1,
  "name": "testuser",
  "timestamp": "2023-11-10T12:00:00",
  "status": "active"
}
```

### 写入测试数据

使用 Kafka 的命令行工具或编程方法将测试数据写入 `example_topic`。这是使用命令行工具的一个示例：

```shell
echo '{"id": 1, "name": "testuser", "timestamp": "2023-11-10T12:00:00", "status": "active"}' | ./kafka-console-producer.sh --broker-list 10.0.96.4:9092 --topic example_topic
```

> 注意：请将 `topic` 和 `bootstrap-server` 替换为您的 Kafka 服务器地址。

要查看最近写入的主题数据，请使用以下命令：

```shell
./kafka-console-consumer.sh --bootstrap-server 10.0.96.4:9092 --topic example_topic --from-beginning
```

## 创建 Routine Load 任务

在 StarRocks 命令行中，创建一个 Routine Load 任务以持续地从 AutoMQ Kafka 主题导入数据：

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

> 注意：请将 `kafka_broker_list` 替换为您的 Kafka 服务器地址。

### 参数解释

#### 数据格式

在 PROPERTIES 子句中指定数据格式为 JSON，即 `"format" = "json"`。

#### 数据提取和转换

要指定源数据与目标表之间的映射和转换关系，请配置 `COLUMNS` 和 `jsonpaths` 参数。`COLUMNS` 中的列名应与目标表的列名对应，其顺序应与源数据中的列顺序相对应。`jsonpaths` 参数用于从 JSON 数据中提取所需的字段数据，类似于新生成的 CSV 数据。然后 `COLUMNS` 参数按顺序临时命名 `jsonpaths` 中的字段。有关数据转换的更多解释，请参见[导入期间的数据转换](./Etl_in_loading.md)。
> 注意：如果每行 JSON 对象的键名和数量（顺序不要求）与目标表的列相对应，则无需配置 `COLUMNS`。

## 验证数据导入

首先，我们检查 Routine Load 导入作业，并确认 Routine Load 导入任务的状态为 RUNNING。

```sql
SHOW ROUTINE LOAD\G;
```

然后，在 StarRocks 数据库中查询相应的表，我们可以观察到数据已经成功导入。

```sql
StarRocks > SELECT * FROM users;
+------+--------------+---------------------+--------+
| id   | name         | timestamp           | status |
+------+--------------+---------------------+--------+
|    1 | testuser     | 2023-11-10T12:00:00 | active |
|    2 | testuser     | 2023-11-10T12:00:00 | active |
+------+--------------+---------------------+--------+
2 rows in set (0.01 sec)
```