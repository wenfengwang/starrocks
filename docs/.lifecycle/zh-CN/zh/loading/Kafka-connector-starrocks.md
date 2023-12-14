```markdown
---
displayed_sidebar: "Chinese"

---

# 使用 Kafka connector 导入数据

StarRocks 提供  Apache Kafka®  连接器 (StarRocks Connector for Apache Kafka®)，持续消费 Kafka 的消息并导入至 StarRocks 中。

使用 Kafka connector 可以更好的融入 Kafka 生态，StarRocks 可以与 Kafka Connect 无缝对接。为 StarRocks 准实时接入链路提供了更多的选择。相比于 Routine Load，您可以在以下场景中优先考虑使用 Kafka connector 导入数据：

- 相比于 Routine Load 仅支持导入 CSV、JSON、Avro 格式的数据，Kafka connector 支持导入更丰富的数据格式。只要数据能通过 Kafka connect 的 converters 转换成 JSON 和 CSV 格式，就可以通过 Kafka connector 导入，例如 Protobuf 格式的数据。
- 需要对数据做自定义的 transform 操作，例如 Debezium CDC 格式的数据。
- 从多个 Kafka Topic 导入数据。

- 从 Confluent cloud 导入数据。

- 需要更精细化的控制导入的批次大小，并行度等参数，以求达到导入速率和资源使用之间的平衡。

## 环境准备

### 准备 Kafka 环境


支持自建 Apache Kafka 集群和 Confluent cloud：

- 如果使用自建 Apache Kafka 集群，请确保已部署 Apache Kafka 集群和 Kafka Connect 集群，并创建 Topic。

- 如果使用 Confluent cloud，请确保已拥有 Confluent 账号，并已经创建集群和 Topic。

### 安装 Kafka connector

安装 Kafka connector 至 Kafka connect。

- 自建 Kafka 集群

  - 下载并解压压缩包 [starrocks-kafka-connector-1.0.0.tar.gz](https://releases.starrocks.io/starrocks/starrocks-kafka-connector-1.0.0.tar.gz)。

  - 将解压后的目录复制到 `plugin.path` 属性所指的路径中。`plugin.path` 属性包含在 Kafka Connect 集群 worker 节点配置文件中。

- Confluent cloud

  > **说明**

  >

  > Kafka connector 目前尚未上传到 Confluent Hub，您需要将其压缩包上传到 Confluent cloud。

### 创建 StarRocks 表

您需要根据 Kafka Topic 以及数据在 StarRocks 中创建对应的表。

## 使用示例

本文以自建 Kafka 集群为例，介绍如何配置 Kafka connector 并启动 Kafka Connect 服务（无需重启 Kafka 服务），导入数据至 StarRocks。

1. 创建 Kafka connector 配置文件 **connect-StarRocks-sink.properties**，并配置对应参数。参数和相关说明，参见[参数说明](#参数说明)。

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

    > 如果源端数据为 CDC 数据，例如 Debezium CDC 格式的数据，并且 StarRocks 表为主键模型的表，为了将源端的数据变更同步至主键模型的表，则您还需要[配置 `transforms` 以及相关参数](#导入-debezium-cdc-格式数据)。

2. 启动 Kafka Connect 服务（无需重启 Kafka 服务）。命令中的参数解释，参见 [Running Kafka Connect](https://kafka.apache.org/documentation.html#connect_running)。

   1. Standalone 模式

      ```Shell
      bin/connect-standalone worker.properties connect-StarRocks-sink.properties [connector2.properties connector3.properties ...]
      ```

   2. 分布式模式

      > **说明**
      >
      > 生产环境中建议您使用分布式模式。

      1. 启动 worker：

          ```Shell
          bin/connect-distributed worker.properties
          ```

      2. 注意，分布式模式不支持启动时在命令行配置 Kafka connector。您需要通过调用 REST API 来配置 Kafka connector 和启动 Kafka connect:

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

| tasks.max                           | 否       | 1                                                            | Kafka 连接器需要创建的任务线程的最大数量，通常与 Kafka Connect 集群中的 worker 节点上的 CPU 核数量相同。如果需要提高导入性能，可以调整此参数。 |
| bufferflush.maxbytes                | 否       | 94371840(90M)                                                | 数据缓冲刷新的大小，达到此阈值后，数据将通过 Stream Load 批量写入 StarRocks。取值范围：[64MB, 10GB]。 Stream Load SDK 可能会启动多个 Stream Load 来缓冲数据，因此此处的阈值是指总数据量大小。 |
| bufferflush.intervalms              | 否       | 300000                                                       | 数据缓冲刷新的时间间隔，用于控制数据写入 StarRocks的延迟，取值范围：[1000, 3600000]。 |
| connect.timeoutms                   | 否       | 1000                                                         | 连接 http-url 的超时时间。取值范围：[100, 60000]。           |
| sink.properties.*                   |          |                                                              | 指定 Stream Load 的参数，用于控制导入行为，例如使用 `sink.properties.format` 指定导入数据的格式为 CSV 或者 JSON。更多参数和说明，请参见 [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。 |
| sink.properties.format              | 否       | json                                                         | Stream Load 导入时的数据格式。取值为 CSV 或者 JSON。默认为JSON。更多参数说明，参见 [CSV 适用参数](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md#csv-适用参数)和 [JSON 适用参数](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md#json-适用参数)。 |

## 使用限制

- 不支持将 Kafka topic 里的一条消息展开成多条导入到 StarRocks。
- Kafka连接器的Sink保证至少一次语义。

## 最佳实践

### 导入Debezium CDC格式数据

如果Kafka数据为Debezium CDC格式，并且StarRocks表为主键模型表，则在Kafka连接器配置文件**connect-StarRocks-sink.properties**中除了[配置基础参数](#使用示例)外，还需要配置`transforms`以及相关参数。

```Properties
transforms=addfield,unwrap
transforms.addfield.type=com.starrocks.connector.kafka.transforms.AddOpFieldForDebeziumRecord
transforms.unwrap.type=io.debezium.transforms.ExtractNewRecordState
transforms.unwrap.drop.tombstones=true
transforms.unwrap.delete.handling.mode=rewrite
```

在上述配置中，我们指定`transforms=addfield,unwrap`。

- addfield transform 用于向 Debezium CDC格式数据的每个记录添加一个__op字段，以支持[主键模型表](../table_design/table_types/primary_key_table.md)。如果StarRocks表不是主键模型表，则无需指定addfield转换。addfield transform的类是com.Starrocks.Kafka.Transforms.AddOpFieldForDebeziumRecord，已经包含在Kafka连接器的JAR文件中，您无需手动安装。
- unwrap transform 是指由Debezium提供的unwrap，可以根据操作类型unwrap Debezium复杂的数据结构。更多信息，请参见[New Record State Extraction](https://debezium.io/documentation/reference/stable/transformations/event-flattening.html)。