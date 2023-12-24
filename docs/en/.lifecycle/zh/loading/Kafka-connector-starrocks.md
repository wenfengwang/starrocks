---
displayed_sidebar: English
---

# 使用 Kafka 连接器加载数据

StarRocks 提供了一个自研的连接器，名为 Apache Kafka® 连接器（StarRocks Connector for Apache Kafka®），可以持续消费来自 Kafka 的消息，并将其加载到 StarRocks 中。Kafka 连接器保证至少一次语义。

Kafka 连接器可以与 Kafka Connect 无缝集成，从而使 StarRocks 与 Kafka 生态系统更好地集成。如果您想要将实时数据加载到 StarRocks 中，使用 Kafka 连接器是一个明智的选择。与常规加载相比，在以下情况下建议使用 Kafka 连接器：

- 与仅支持加载 CSV、JSON 和 Avro 格式数据的常规加载相比，Kafka 连接器可以加载更多格式的数据，例如 Protobuf 等。只要使用 Kafka Connect 的转换器将数据转换为 JSON 和 CSV 格式，就可以通过 Kafka 连接器将数据加载到 StarRocks 中。
- 自定义数据转换，例如 Debezium 格式的 CDC 数据。
- 从多个 Kafka 主题加载数据。
- 从 Confluent 云加载数据。
- 需要对负载批处理大小、并行度和其他参数进行更精细的控制，以实现加载速度和资源利用率之间的平衡。

## 准备工作

### 设置 Kafka 环境

支持自管理的 Apache Kafka 集群和 Confluent 云。

- 对于自管理的 Apache Kafka 集群，请确保部署了 Apache Kafka 集群和 Kafka Connect 集群，并创建了主题。
- 对于 Confluent 云，请确保您拥有 Confluent 帐户，并创建了集群和主题。

### 安装 Kafka 连接器

将 Kafka 连接器提交到 Kafka Connect：

- 自管理的 Kafka 集群：

  - 下载并解压 [starrocks-kafka-connector-1.0.0.tar.gz](https://releases.starrocks.io/starrocks/starrocks-kafka-connector-1.0.0.tar.gz)。
  - 将提取的目录复制到 `plugin.path` 属性指定的路径。您可以在 Kafka Connect 集群中工作节点的配置文件中找到 `plugin.path` 属性。

- Confluent 云：

  > **注意**
  >
  > Kafka 连接器目前尚未上传到 Confluent Hub。您需要将压缩文件上传到 Confluent 云。

### 创建 StarRocks 表

根据 Kafka 主题和数据在 StarRocks 中创建一个或多个表。

## 示例

以下步骤以自管理的 Kafka 集群为例，演示了如何配置 Kafka 连接器并启动 Kafka Connect（无需重新启动 Kafka 服务），以便将数据加载到 StarRocks 中。

1. 创建名为 **connect-StarRocks-sink.properties** 的 Kafka 连接器配置文件，并配置相关参数。有关参数的详细信息，请参见[参数](#参数)。

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
    > 如果源数据是 CDC 数据，例如 Debezium 格式的数据，并且 StarRocks 表是主键表，则还需要[配置 `transform`](#加载-debezium格式的cdc数据)以将源数据更改同步到主键表中。

2. 运行 Kafka 连接器（无需重新启动 Kafka 服务）。有关以下命令中的参数和说明，请参见[Kafka 文档](https://kafka.apache.org/documentation.html#connect_running)。

    - 独立模式

        ```shell
        bin/connect-standalone worker.properties connect-StarRocks-sink.properties [connector2.properties connector3.properties ...]
        ```

    - 分布式模式

      > **注意**
      >
      > 建议在生产环境中使用分布式模式。

    - 启动工作节点。

        ```shell
        bin/connect-distributed worker.properties
        ```

    - 请注意，在分布式模式下，连接器配置不会通过命令行传递。而是使用下面描述的 REST API 来配置 Kafka 连接器并运行 Kafka 连接。

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

3. 查询 StarRocks 表中的数据。

## 参数

| 参数                           | 必填 | 默认值                                                | 描述                                                  |
| ----------------------------------- | -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 名字                                | 是      |                                                              | 此 Kafka 连接器的名称。它必须在此 Kafka Connect 群集中的所有 Kafka 连接器中是全局唯一的。例如，starrocks-kafka-connector。 |
| 连接器.class                     | 是      | com.starrocks.connector.kafka.SinkConnector                  | 此 Kafka 连接器的接收器使用的类。                   |
| 主题                              | 是      |                                                              | 要订阅的一个或多个主题，其中每个主题对应一个 StarRocks 表。默认情况下，StarRocks 假定主题名称与 StarRocks 表名称匹配。因此，StarRocks 使用主题名称来确定目标 StarRocks 表。请选择填写 `topics` 或 `topics.regex` （下面），但不能同时填写两者。但是，如果 StarRocks 表名称与主题名称不同，则使用可选的 `starrocks.topic2table.map` 参数（下面）来指定主题名称到表名称的映射关系。 |
| 主题.regex                        |          | 用于匹配要订阅的一个或多个主题的正则表达式。有关更多说明，请参见 `topics`。请选择填写 `topics.regex` 或 `topics` （上面），但不能同时填写两者。 |                                                              |
| starrocks.topic2table.map           | 不       |                                                              | 当主题名称与 StarRocks 表名称不同时，主题名称与表名称的映射关系。格式为 `<topic-1>:<table-1>,<topic-2>:<table-2>,...`。 |
| starrocks.http.url                  | 是      |                                                              | StarRocks 集群中 FE 的 HTTP URL。格式为 `<fe_host1>:<fe_http_port1>,<fe_host2>:<fe_http_port2>,...`。多个地址用逗号（,）分隔。例如，`192.168.xxx.xxx:8030,192.168.xxx.xxx:8030`。 |
| starrocks.database.name             | 是      |                                                              | StarRocks 数据库的名称。                              |
| starrocks.username                  | 是      |                                                              | 您的 StarRocks 集群账号的用户名。用户需要在 StarRocks 表上拥有 [INSERT](../sql-reference/sql-statements/account-management/GRANT.md) 权限。 |
| starrocks.password                  | 是      |                                                              | 您的 StarRocks 集群账号的密码。              |
| key.converter                       | 不       | Kafka Connect 集群使用的键转换器                                                           | 该参数指定 sink 连接器（Kafka-connector-starrocks）使用的键转换器，用于反序列化 Kafka 数据的键。默认键转换器是 Kafka Connect 集群使用的键转换器。 |
| value.converter                     | 不       | Kafka Connect 集群使用的值转换器                             | 该参数指定 sink 连接器（Kafka-connector-starrocks）使用的值转换器，用于反序列化 Kafka 数据的值。默认值转换器是 Kafka Connect 集群使用的转换器。 |
| key.converter.schema.registry.url   | 不       |                                                              | 键转换器的模式注册表 URL。                   |
| value.converter.schema.registry.url | 不       |                                                              | 值转换器的模式注册表 URL。                 |
| tasks.max                           | 不       | 1                                                            | Kafka 连接器可以创建的任务线程数的上限，通常与 Kafka Connect 集群中工作节点的 CPU 内核数相同。您可以调整此参数以控制负载性能。 |
| bufferflush.maxbytes                | 不       | 94371840（90M）                                                | 一次性发送到 StarRocks 的数据在内存中可以累积的最大大小。最大值范围从 64 MB 到 10 GB。请注意，流加载 SDK 缓冲区可能会创建多个流加载作业来缓冲数据。因此，这里提到的阈值是指总数据大小。 |
| bufferflush.intervalms              | 不       | 300000                                                       | 发送一批数据的时间间隔，用于控制加载延迟。范围：[1000, 3600000]。 |
| connect.timeoutms                   | 不       | 1000                                                         | 连接到 HTTP URL 的超时。范围：[100, 60000]。 |
| sink.properties.*                   |          |                                                              | 控制加载行为的流加载参数。例如，参数 `sink.properties.format` 指定了流加载使用的格式，例如 CSV 或 JSON。有关支持的参数及其说明的列表，请参见 [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM LOAD.md)。 |
| sink.properties.format              | 不       | json                                                         | 用于流加载的格式。Kafka 连接器会将每批数据转换为此格式，然后再发送到 StarRocks。有效值：`csv` 和 `json`。有关详细信息，请参见 [CSV 参数**](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md#csv-parameters)和** [JSON 参数](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md#json-parameters)。 |

## 限制

- 不支持将 Kafka 主题中的单条消息扁平化为多个数据行并加载到 StarRocks 中。
- Kafka 连接器的 Sink 保证至少一次语义。

## 最佳实践

### 加载 Debezium 格式的 CDC 数据

如果 Kafka 数据采用 Debezium CDC 格式，并且 StarRocks 表是主键表，则还需要配置 `transforms` 参数和其他相关参数。

```Properties
transforms=addfield,unwrap
transforms.addfield.type=com.starrocks.connector.kafka.transforms.AddOpFieldForDebeziumRecord
transforms.unwrap.type=io.debezium.transforms.ExtractNewRecordState
transforms.unwrap.drop.tombstones=true
transforms.unwrap.delete.handling.mode
```

在上述配置中，我们指定了 `transforms=addfield,unwrap`。

- addfield 变换用于向每条 Debezium CDC 格式数据的记录中添加 __op 字段，以支持 StarRocks 主键模型表。如果 StarRocks 表不是主键表，则无需指定 addfield 变换。addfield 变换类为 com.starrocks.connector.kafka.transforms.AddOpFieldForDebeziumRecord。它包含在 Kafka 连接器的 JAR 文件中，因此无需手动安装。
- unwrap 变换由 Debezium 提供，用于根据操作类型解包 Debezium 的复杂数据结构。有关详细信息，请参见[新记录状态提取](https://debezium.io/documentation/reference/stable/transformations/event-flattening.html)。```