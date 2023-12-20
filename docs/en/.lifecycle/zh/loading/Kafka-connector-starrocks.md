---
displayed_sidebar: English
---

# 使用 Kafka 连接器加载数据

StarRocks 提供了一个名为 Apache Kafka® 连接器（StarRocks Connector for Apache Kafka®）的自研连接器，该连接器持续消费来自 Kafka 的消息并将其加载到 StarRocks 中。Kafka 连接器保证至少一次语义。

Kafka 连接器可以与 Kafka Connect 无缝集成，这使得 StarRocks 能够更好地与 Kafka 生态系统集成。如果您想将实时数据加载到 StarRocks 中，使用 Kafka 连接器是一个明智的选择。与 Routine Load 相比，以下场景推荐使用 Kafka 连接器：

- 与 Routine Load 仅支持加载 CSV、JSON 和 Avro 格式的数据相比，Kafka 连接器可以加载更多格式的数据，例如 Protobuf。只要数据可以使用 Kafka Connect 的转换器转换为 JSON 和 CSV 格式，就可以通过 Kafka 连接器将数据加载到 StarRocks 中。
- 自定义数据转换，例如 Debezium 格式的 CDC 数据。
- 从多个 Kafka 主题加载数据。
- 从 Confluent Cloud 加载数据。
- 需要对加载批量大小、并行度和其他参数进行更精细的控制，以实现加载速度和资源利用率之间的平衡。

## 准备工作

### 设置 Kafka 环境

支持自托管的 Apache Kafka 集群和 Confluent Cloud。

- 对于自托管的 Apache Kafka 集群，请确保部署了 Apache Kafka 集群和 Kafka Connect 集群并创建了主题。
- 对于 Confluent Cloud，请确保您拥有 Confluent 账户并创建了集群和主题。

### 安装 Kafka 连接器

将 Kafka 连接器提交到 Kafka Connect：

- 自托管 Kafka 集群：

  - 下载并解压 [starrocks-kafka-connector-1.0.0.tar.gz](https://releases.starrocks.io/starrocks/starrocks-kafka-connector-1.0.0.tar.gz)。
  - 将解压后的目录复制到 `plugin.path` 属性中指定的路径。您可以在 Kafka Connect 集群内工作节点的配置文件中找到 `plugin.path` 属性。

- Confluent Cloud：

    > **注意**
    > Kafka 连接器目前尚未上传到 Confluent Hub。您需要将压缩文件上传到 Confluent Cloud。

### 创建 StarRocks 表

根据 Kafka Topics 和数据在 StarRocks 中创建一张或多张表。

## 示例

以下步骤以自托管的 Kafka 集群为例，演示如何配置 Kafka 连接器并启动 Kafka Connect（无需重启 Kafka 服务）以将数据加载到 StarRocks 中。

1. 创建名为 **connect-StarRocks-sink.properties** 的 Kafka 连接器配置文件并配置参数。有关参数的详细信息，请参阅[参数](#parameters)。

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
      > 如果源数据是 CDC 数据，例如 Debezium 格式的数据，且 StarRocks 表是 Primary Key 表，还需要配置 [`transforms`](#load-debezium-formatted-cdc-data)，以便将源数据的变化同步到 Primary Key 表。

2. 运行 Kafka Connector（无需重启 Kafka 服务）。以下命令中的参数及说明请参见 [Kafka 文档](https://kafka.apache.org/documentation.html#connect_running)。

-     独立模式

      ```shell
      bin/connect-standalone worker.properties connect-StarRocks-sink.properties [connector2.properties connector3.properties ...]
      ```

-     分布式模式

            > **注意**
            > 在生产环境中，建议使用分布式模式。

-     启动 worker。

      ```shell
      bin/connect-distributed worker.properties
      ```

-     请注意，在分布式模式下，连接器配置不会在命令行上传递。相反，请使用下面描述的 REST API 来配置 Kafka 连接器并运行 Kafka Connect。

      ```Shell
      curl -i http://127.0.0.1:8083/connectors -H "Content-Type: application/json" -X POST -d '{
      "name":"starrocks-kafka-connector",
      "config":{
          "connector.class":"com.starrocks.connector.kafka.StarRocksSinkConnector",
          "topics":"dbserver1.inventory.customers",
          "starrocks.http.url":"192.168.xxx.xxx:8030,192.168.xxx.xxx:8030",
          "starrocks.username":"root",
          "starrocks.password":"123456",
          "starrocks.database.name":"inventory",
          "key.converter":"io.confluent.connect.json.JsonSchemaConverter",
          "value.converter":"io.confluent.connect.json.JsonSchemaConverter"
      }
      }
      ```

3. 查询 StarRocks 表中的数据。

## 参数

|参数|必填|默认值|说明|
|---|---|---|---|
|name|是||此 Kafka 连接器的名称。它在此 Kafka Connect 集群内的所有 Kafka 连接器中必须是全局唯一的。例如，starrocks-kafka-connector。|
|connector.class|是|com.starrocks.connector.kafka.StarRocksSinkConnector|此 Kafka 连接器 sink 使用的类。|
|topics|是||要订阅的一个或多个主题，其中每个主题对应一个 StarRocks 表。默认情况下，StarRocks 假定主题名称与 StarRocks 表的名称匹配。因此 StarRocks 通过主题名称来确定目标 StarRocks 表。请选择填写 `topics` 或 `topics.regex`（如下），但不能两者都填写。但是，如果 StarRocks 表名称与主题名称不同，则使用可选的 `starrocks.topic2table.map` 参数（如下）来指定从主题名称到表名称的映射。|
|topics.regex|||匹配要订阅的一个或多个主题的正则表达式。有关更多说明，请参阅 `topics`。请选择填写 `topics.regex` 或 `topics`（上方），但不能同时填写两者。|
|starrocks.topic2table.map|否||当主题名称与 StarRocks 表名称不同时，StarRocks 表名称和主题名称的映射。格式为 `<topic-1>:<table-1>,<topic-2>:<table-2>,...`。|
|starrocks.http.url|是||StarRocks 集群中 FE 的 HTTP URL。格式为 `<fe_host1>:<fe_http_port1>,<fe_host2>:<fe_http_port2>,...`。多个地址之间用逗号（,）分隔。例如，`192.168.xxx.xxx:8030,192.168.xxx.xxx:8030`。|
|starrocks.database.name|是||StarRocks 数据库的名称。|
|starrocks.username|是||您的 StarRocks 集群账户的用户名。用户需要在 StarRocks 表上具有 [INSERT](../sql-reference/sql-statements/account-management/GRANT.md) 权限。|
|starrocks.password|是||您的 StarRocks 集群账户的密码。|
|key.converter|否||Kafka Connect 集群使用的 key 转换器。此参数指定 sink 连接器（Kafka-connector-starrocks）的 key 转换器，用于反序列化 Kafka 数据的 key。默认 key 转换器是 Kafka Connect 集群使用的转换器。|
|value.converter|否||Kafka Connect 集群使用的 value 转换器。此参数指定 sink 连接器（Kafka-connector-starrocks）的 value 转换器，用于反序列化 Kafka 数据的 value。默认 value 转换器是 Kafka Connect 集群使用的转换器。|
|key.converter.schema.registry.url|否||key 转换器的 schema registry URL。|
|value.converter.schema.registry.url|否||value 转换器的 schema registry URL。|
|tasks.max|否|1|Kafka 连接器可以创建的任务线程数的上限，通常与 Kafka Connect 集群中 worker 节点上的 CPU 核心数相同。您可以调整此参数来控制负载性能。|
|bufferflush.maxbytes|否|94371840(90M)|一次发送到 StarRocks 之前可以在内存中累积的最大数据大小。最大值范围为 64 MB 至 10 GB。请记住，Stream Load SDK 缓冲区可能会创建多个 Stream Load 作业来缓冲数据。因此，这里所说的阈值是指总数据大小。|
|bufferflush.intervalms|否|300000|发送一批数据的时间间隔，控制加载延迟。范围：[1000, 3600000]。|
|connect.timeoutms|否|1000|连接到 HTTP URL 的超时时间。范围：[100, 60000]。|
|sink.properties.*|||Stream Load 参数用于控制负载行为。例如，参数 `sink.properties.format` 指定用于 Stream Load 的格式，如 CSV 或 JSON。有关支持的参数及其描述的列表，请参阅 [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。|
|sink.properties.format|否|json|用于 Stream Load 的格式。Kafka 连接器会将每批数据转换为该格式，然后发送到 StarRocks。有效值：`csv` 和 `json`。有关详细信息，请参阅 [CSV 参数](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md#csv-parameters) 和 [JSON 参数](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md#json-parameters)。|

## 限制

- 不支持将 Kafka 主题中的单条消息扁平化为多个数据行并加载到 StarRocks 中。
- Kafka 连接器的 Sink 保证至少一次语义。

## 最佳实践

### 加载 Debezium 格式的 CDC 数据

如果 Kafka 数据为 Debezium CDC 格式，且 StarRocks 表为 Primary Key 表，还需要配置 `transforms` 参数和其他相关参数。

```Properties
transforms=addfield,unwrap
transforms.addfield.type=com.starrocks.connector.kafka.transforms.AddOpFieldForDebeziumRecord
transforms.unwrap.type=io.debezium.transforms.ExtractNewRecordState
transforms.unwrap.drop.tombstones=true
transforms.unwrap.delete.handling.mode=rewrite
```

在上面的配置中，我们指定 `transforms=addfield,unwrap`。

- `addfield` 转换用于将 `__op` 字段添加到 Debezium CDC 格式数据的每条记录中，以支持 StarRocks 主键模型表。如果 StarRocks 表不是主键表，则不需要指定 `addfield` 转换。`addfield` 转换类是 `com.starrocks.connector.kafka.transforms.AddOpFieldForDebeziumRecord`。它包含在 Kafka 连接器 JAR 文件中，因此您不需要手动安装它。
- `unwrap` 转换由 Debezium 提供，用于根据操作类型展开 Debezium 的复杂数据结构。有关更多信息，请参阅 [新记录状态提取](https://debezium.io/documentation/reference/stable/transformations/event-flattening.html)。