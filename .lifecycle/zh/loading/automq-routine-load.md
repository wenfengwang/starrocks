
# 持续从AutoMQ Kafka加载数据

[AutoMQ Kafka](https://docs.automq.com/docs/automq-s3kafka/YUzOwI7AgiNIgDk1GJAcu6Uanog)是为云环境重新设计的Kafka云原生版本。
AutoMQ Kafka是[开源](https://github.com/AutoMQ/automq-for-kafka)且与Kafka协议完全兼容，充分利用了云计算的优势。
与自托管的Apache Kafka相比，AutoMQ Kafka凭借其云原生架构，提供了容量自动伸缩、网络流量自我平衡、秒级分区迁移等特性。这些特性有助于显著降低用户的总拥有成本（TCO）。

本文将指导您如何使用StarRocks的定期加载（Routine Load）功能，将数据导入到AutoMQ Kafka中。要了解定期加载的基础原理，请参见定期加载基础知识部分。

## 准备环境

### 准备StarRocks和测试数据

确保您的StarRocks集群正在运行。为了演示，本文将跟随[部署指南](../quick_start/deploy_with_docker.md)的步骤，在Linux机器上通过Docker安装StarRocks集群。

创建具有主键模型的数据库和测试表：

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

按照AutoMQ [快速开始](https://docs.automq.com/docs/automq-s3kafka/VKpxwOPvciZmjGkHk5hcTz43nde) 指南，部署您的AutoMQ Kafka集群，以准备AutoMQ Kafka环境和测试数据。确保StarRocks能够直接连接到您的AutoMQ Kafka服务器。

要在AutoMQ Kafka中快速创建一个名为example_topic的主题，并向其写入测试JSON数据，请按照以下步骤操作：

### 创建主题

使用Kafka的命令行工具来创建主题。确保您可以访问Kafka环境，并且Kafka服务已经启动。以下是创建主题的命令：

```shell
./kafka-topics.sh --create --topic example_topic --bootstrap-server 10.0.96.4:9092 --partitions 1 --replication-factor 1
```

> 注意：请将topic和bootstrap-server替换成您的Kafka服务器地址。

要检查主题创建的结果，使用以下命令：

```shell
./kafka-topics.sh --describe example_topic --bootstrap-server 10.0.96.4:9092
```

### 生成测试数据

生成一个简单的JSON格式测试数据

```json
{
  "id": 1,
  "name": "testuser",
  "timestamp": "2023-11-10T12:00:00",
  "status": "active"
}
```

### 写入测试数据

使用Kafka的命令行工具或编程方式，将测试数据写入example_topic。这是一个使用命令行工具的示例：

```shell
echo '{"id": 1, "name": "testuser", "timestamp": "2023-11-10T12:00:00", "status": "active"}' | sh kafka-console-producer.sh --broker-list 10.0.96.4:9092 --topic example_topic
```

> 注意：请将topic和bootstrap-server替换成您的Kafka服务器地址。

要查看最近写入的主题数据，使用以下命令：

```shell
sh kafka-console-consumer.sh --bootstrap-server 10.0.96.4:9092 --topic example_topic --from-beginning
```

## 创建定期加载任务

在StarRocks命令行中，创建一个定期加载任务，以持续从AutoMQ Kafka主题导入数据：

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

> 注意：请将kafka_broker_list替换成您的Kafka服务器地址。

### 参数说明

#### 数据格式

在PROPERTIES子句中，使用"format" = "json"指定数据格式为JSON。

#### 数据提取和转换

为了指定源数据与目标表之间的映射和转换关系，配置**COLUMNS**和**jsonpaths**参数。**COLUMNS**中的列名应与目标表的列名相对应，其顺序应与源数据中的列顺序相匹配。**jsonpaths**参数用于从JSON数据中提取需要的字段数据，这类似于新生成的CSV数据。然后，**COLUMNS**参数会按顺序临时命名**jsonpaths**中的字段。有关数据转换的详细说明，请参阅[导入过程中的数据转换](./Etl_in_loading.md)。
> 注意：如果每行的每个JSON对象都有与目标表的列相对应的键名和数量（顺序无关紧要），则无需配置COLUMNS。

## 验证数据导入

首先，我们检查定期加载作业，并确认定期加载任务的状态为RUNNING。

```sql
show routine load\G;
```

然后，查询StarRocks数据库中相应的表，我们可以观察到数据已经成功导入。

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
