# 从 AutoMQ Kafka 持续加载数据

[AutoMQ for Kafka](https://docs.automq.com/docs/automq-s3kafka/YUzOwI7AgiNIgDk1GJAcu6Uanog) 是专为云环境重新设计的 Kafka 云原生版本。
AutoMQ Kafka 是[开源](https://github.com/AutoMQ/automq-for-kafka)的，并且完全兼容 Kafka 协议，充分利用云的优势。
与自行管理的 Apache Kafka 相比，AutoMQ Kafka 采用云原生架构，提供诸如容量自动扩展、网络流量自平衡、秒级分区移动等功能。这些功能有助于显著降低用户的总拥有成本（TCO）。

本文将指导您使用 StarRocks Routine Load 将数据导入 AutoMQ Kafka。
要了解例程负载的基本原理，请参阅例程负载基础知识部分。

## 准备环境

### 准备 StarRocks 和测试数据

确保您有一个正在运行的 StarRocks 集群。出于演示目的，本文按照 [部署指南](../quick_start/deploy_with_docker.md) ，通过 Docker 在 Linux 机器上安装 StarRocks 集群。

使用主键模型创建数据库和测试表：

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

要准备 AutoMQ Kafka 环境和测试数据，请按照 AutoMQ [快速入门](https://docs.automq.com/docs/automq-s3kafka/VKpxwOPvciZmjGkHk5hcTz43nde) 指南部署您的 AutoMQ Kafka 集群。确保 StarRocks 可以直接连接到您的 AutoMQ Kafka 服务器。

要快速创建在 AutoMQ Kafka 中命名为 `example_topic` 的主题，并向其中写入测试 JSON 数据，请执行以下步骤：

### 创建主题

使用 Kafka 的命令行工具创建主题。确保您有权限访问 Kafka 环境，并且 Kafka 服务正在运行。
以下是创建主题的命令：

```shell
./kafka-topics.sh --create --topic example_topic --bootstrap-server 10.0.96.4:9092 --partitions 1 --replication-factor 1
```

> 注意：请将 `topic` 和 `bootstrap-server` 替换为您的 Kafka 服务器地址。

要检查主题创建的结果，请使用以下命令：

```shell
./kafka-topics.sh --describe example_topic --bootstrap-server 10.0.96.4:9092
```

### 生成测试数据

生成简单的 JSON 格式测试数据

```json
{
  "id": 1,
  "name": "testuser",
  "timestamp": "2023-11-10T12:00:00",
  "status": "active"
}
```

### 写入测试数据

使用 Kafka 的命令行工具或编程方法将测试数据写入 `example_topic`。以下是使用命令行工具的示例：

```shell
echo '{"id": 1, "name": "testuser", "timestamp": "2023-11-10T12:00:00", "status": "active"}' | sh kafka-console-producer.sh --broker-list 10.0.96.4:9092 --topic example_topic
```

> 注意：请将 `topic` 和 `bootstrap-server` 替换为您的 Kafka 服务器地址。

要查看最近写入的主题数据，请使用以下命令：

```shell
sh kafka-console-consumer.sh --bootstrap-server 10.0.96.4:9092 --topic example_topic --from-beginning
```

## 创建例程加载任务

在 StarRocks 命令行中，创建一个 Routine Load 任务，以持续从 AutoMQ Kafka 主题中导入数据：

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

### 参数说明

#### 数据格式

在 PROPERTIES 子句中将数据格式指定为 JSON，格式为 "format" = "json"。

#### 数据提取和转换

要指定源数据与目标表之间的映射和转换关系，请配置 COLUMNS 和 jsonpaths 参数。COLUMNS 中的列名与目标表的列名对应，并且它们的顺序与源数据中的列顺序相对应。jsonpaths 参数用于从 JSON 数据中提取所需的字段数据，类似于新生成的 CSV 数据。然后，COLUMNS 参数按顺序临时命名 jsonpaths 中的字段。有关数据转换的更多说明，请参见[导入过程中的数据转换](./Etl_in_loading.md)。
> 注意：如果每行每个 JSON 对象都具有与目标表的列相对应的键名称和数量（无需顺序），则无需配置 COLUMNS。

## 验证数据导入

首先，我们检查例程加载导入作业，并确认例程加载导入任务的状态为运行状态。

```sql
show routine load\G;
```

然后，在 StarRocks 数据库中查询相应的表，我们可以观察到数据已经成功导入。

```sql
StarRocks > select * from users;
+------+--------------+---------------------+--------+
| id   | name         | timestamp           | status |
+------+--------------+---------------------+--------+
|    1 | testuser     | 2023-11-10T12:00:00 | active |
|    2 | testuser     | 2023-11-10T12:00:00 | active |
+------+--------------+---------------------+--------+
2 rows in set (0.01 sec)