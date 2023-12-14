# 从AutoMQ Kafka连续加载数据

[AutoMQ for Kafka](https://docs.automq.com/docs/automq-s3kafka/YUzOwI7AgiNIgDk1GJAcu6Uanog) 是专为云环境重新设计的Kafka的云原生版本。
AutoMQ Kafka是[开源的](https://github.com/AutoMQ/automq-for-kafka)，完全兼容Kafka协议，充分发挥云端的优势。
与自我管理的Apache Kafka相比，AutoMQ Kafka具有云原生架构，提供诸如容量自动扩展、网络流量自平衡、秒级分区迁移等功能。这些功能有助于大幅降低用户的总体拥有成本（TCO）。

本文将指导您使用StarRocks Routine Load将数据导入AutoMQ Kafka。
如需了解Routine Load的基本原理，请参考Routine Load基础部分。

## 准备环境

### 准备StarRocks和测试数据

确保您有一个正在运行的StarRocks集群。为了演示目的，本文将按照[部署指南](../quick_start/deploy_with_docker.md)在Linux机器上通过Docker安装StarRocks集群。

创建数据库和具有主键模型的测试表：

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

## 准备AutoMQ Kafka和测试数据

要准备您的AutoMQ Kafka环境和测试数据，请按照AutoMQ [快速入门](https://docs.automq.com/docs/automq-s3kafka/VKpxwOPvciZmjGkHk5hcTz43nde)指南部署您的AutoMQ Kafka集群。确保StarRocks可以直接连接至您的AutoMQ Kafka服务器。

要在AutoMQ Kafka中快速创建名为`example_topic`的主题，并向其中写入测试JSON数据，按照以下步骤操作：

### 创建主题

使用Kafka的命令行工具来创建一个主题。确保您可以访问Kafka环境并且Kafka服务正在运行。
以下是创建主题的命令：

```shell
./kafka-topics.sh --create --topic example_topic --bootstrap-server 10.0.96.4:9092 --partitions 1 --replication-factor 1
```

> 注意：将`topic`和`bootstrap-server`替换为您的Kafka服务器地址。

要检查主题创建的结果，请使用以下命令：

```shell
./kafka-topics.sh --describe example_topic --bootstrap-server 10.0.96.4:9092
```

### 生成测试数据

生成简单的JSON格式测试数据

```json
{
  "id": 1,
  "name": "testuser",
  "timestamp": "2023-11-10T12:00:00",
  "status": "active"
}
```

### 写入测试数据

使用Kafka的命令行工具或编程方法将测试数据写入`example_topic`。以下是使用命令行工具的示例：

```shell
echo '{"id": 1, "name": "testuser", "timestamp": "2023-11-10T12:00:00", "status": "active"}' | sh kafka-console-producer.sh --broker-list 10.0.96.4:9092 --topic example_topic
```

> 注意：将`topic`和`bootstrap-server`替换为您的Kafka服务器地址。

要查看最近写入的主题数据，请使用以下命令：

```shell
sh kafka-console-consumer.sh --bootstrap-server 10.0.96.4:9092 --topic example_topic --from-beginning
```

## 创建一个Routine Load任务

在StarRocks命令行中，创建一个Routine Load任务以连续从AutoMQ Kafka主题中导入数据：

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

> 注意：将`kafka_broker_list`替换为您的Kafka服务器地址。

### 参数解释

#### 数据格式

在PROPERTIES子句的"format" = "json"中指定数据格式为JSON。

#### 数据提取和转换

要指定源数据和目标表之间的映射和转换关系，配置COLUMNS和jsonpaths参数。COLUMNS中的列名对应于目标表的列名，并且它们的顺序对应于源数据的列顺序。jsonpaths参数用于从JSON数据中提取所需的字段数据，类似于新生成的CSV数据。然后COLUMNS参数暂时以顺序命名jsonpaths中的字段。有关数据转换的更多说明，请参阅[导入期间的数据转换](./Etl_in_loading.md)。
> 注意：如果每行每个JSON对象具有与目标表的列对应的键名称和数量（不需要顺序），则无需配置COLUMNS。

## 验证数据导入

首先，检查Routine Load导入任务，并确认Routine Load导入任务状态为RUNNING状态。

```sql
show routine load\G;
```

然后，查询StarRocks数据库中的相应表，我们可以观察到数据已成功导入。

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