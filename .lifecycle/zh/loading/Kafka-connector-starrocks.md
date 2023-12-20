---
displayed_sidebar: English
---

# 使用 Kafka 连接器加载数据

StarRocks 提供了一个自研的连接器，称为 Apache Kafka® 连接器（StarRocks Connector for Apache Kafka®），它能够持续地从 Kafka 中消费消息并将它们加载到 StarRocks。Kafka 连接器保证至少一次处理语义。

Kafka 连接器能够与 Kafka Connect 无缝集成，使 StarRocks 能更好地融入 Kafka 生态系统。如果您想实时地将数据加载到 StarRocks，选择 Kafka 连接器是明智的。与常规加载相比，推荐在以下场景中使用 Kafka 连接器：

- 与只支持加载 CSV、JSON 和 Avro 格式数据的常规加载相比，Kafka 连接器能够加载更多格式的数据，如 Protobuf。只要数据能够通过 Kafka Connect 的转换器转换成 JSON 或 CSV 格式，就能通过 Kafka 连接器加载到 StarRocks。
- 自定义数据转换，比如 Debezium 格式化的 CDC 数据。
- 从多个 Kafka 主题加载数据。
- 从 Confluent Cloud 加载数据。
- 需要更细致地控制加载批次大小、并行度和其他参数，以达成加载速度和资源使用之间的平衡。

## 准备工作

### 搭建 Kafka 环境

支持自托管的 Apache Kafka 集群和 Confluent Cloud。

- 对于自托管的 Apache Kafka 集群，请确保已部署 Apache Kafka 集群和 Kafka Connect 集群并已创建主题。
- 对于 Confluent Cloud，请确保您已有 Confluent 账户，并已创建集群和主题。

### 安装 Kafka 连接器

将 Kafka 连接器提交至 Kafka Connect：

- 自托管 Kafka 集群：

  - 下载并解压缩 [starrocks-kafka-connector-1.0.0.tar.gz](https://releases.starrocks.io/starrocks/starrocks-kafka-connector-1.0.0.tar.gz) 文件。
  - 将解压后的目录复制到 Kafka Connect 集群工作节点配置文件中指定的 plugin.path 属性的路径下。

- Confluent Cloud：

    > **注意**
    > Kafka 连接器目前还未上传至 Confluent Hub。您需要将**压缩文件**上传至 Confluent Cloud。

### 创建 StarRocks 表

根据 Kafka 主题和数据在 StarRocks 中创建相应的表。

## 示例

以下步骤以自托管的 Kafka 集群为例，展示如何配置 Kafka 连接器并启动 Kafka Connect（无需重启 Kafka 服务）以便将数据加载到 StarRocks。

1. 创建一个名为**connect-StarRocks-sink.properties**的Kafka连接器配置文件，并配置参数。有关参数的详细信息，请参见[参数](#parameters)部分。

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
      > 如果源数据是 CDC 数据，如 Debezium 格式的数据，并且 StarRocks 表是主键表，则您还需要配置 [transform](#load-debezium-formatted-cdc-data) 参数，以便将源数据的变更同步到主键表中。

2. 运行 Kafka 连接器（无需重启 Kafka 服务）。关于以下命令中的参数及其描述，请参见[Kafka 文档](https://kafka.apache.org/documentation.html#connect_running)。

-     单机模式

      ```shell
      bin/connect-standalone worker.properties connect-StarRocks-sink.properties [connector2.properties connector3.properties ...]
      ```

-     分布式模式

            > **注意**
            > 在生产环境中，推荐使用**分布式模式**。

-     启动工作进程。

      ```shell
      bin/connect-distributed worker.properties
      ```

-     请注意，在分布式模式下，连接器的配置不是通过命令行传递的。相反，您需要使用下面描述的 REST API 来配置 Kafka 连接器并运行 Kafka Connect。

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

3. 在 StarRocks 表中查询数据。

## 参数

|参数|必填|默认值|说明|
|---|---|---|---|
|name|YES|此 Kafka 连接器的名称。它在此 Kafka Connect 集群内的所有 Kafka 连接器中必须是全局唯一的。例如，starrocks-kafka-connector。|
|connector.class|YES|com.starrocks.connector.kafka.SinkConnector|此 Kafka 连接器接收器使用的类。|
|topics|YES|要订阅的一个或多个主题，其中每个主题对应一个 StarRocks 表。默认情况下，StarRocks 假定主题名称与 StarRocks 表的名称匹配。因此StarRocks通过主题名称来确定目标StarRocks表。请选择填写topics或topics.regex（如下），但不能两者都填写。但是，如果StarRocks表名称与主题名称不同，则使用可选的starrocks.topic2table.map参数（如下）来指定从主题名称到表名称的映射。|
|topics.regex|匹配要订阅的一个或多个主题的正则表达式。有关更多说明，请参阅主题。请选择填写 topic.regex 或主题（上方），但不能同时填写两者。|
|starrocks.topic2table.map|NO|当主题名称与 StarRocks 表名称不同时，StarRocks 表名称和主题名称的映射。格式为<主题1>:<表1>,<主题2>:<表2>,....|
|starrocks.http.url|YES|StarRocks 集群中 FE 的 HTTP URL。格式为<fe_host1>:<fe_http_port1>,<fe_host2>:<fe_http_port2>,.... 多个地址之间用逗号（,）分隔。例如，192.168.xxx.xxx:8030,192.168.xxx.xxx:8030。|
|starrocks.database.name|YES|StarRocks 数据库的名称。|
|starrocks.username|YES|您的 StarRocks 集群帐户的用户名。用户需要 StarRocks 表的 INSERT 权限。|
|starrocks.password|YES|您的 StarRocks 集群帐户的密码。|
|key.converter|NO|Kafka Connect 集群使用的密钥转换器|该参数指定接收器连接器（Kafka-connector-starrocks）的密钥转换器，用于反序列化 Kafka 数据的密钥。默认密钥转换器是 Kafka Connect 集群使用的密钥转换器。|
|value.converter|NO|Kafka Connect 集群使用的值转换器|该参数指定接收器连接器（Kafka-connector-starrocks）的值转换器，用于反序列化 Kafka 数据的值。默认值转换器是 Kafka Connect 集群使用的转换器。|
|key.converter.schema.registry.url|NO|密钥转换器的架构注册表 URL。|
|value.converter.schema.registry.url|NO|值转换器的架构注册表 URL。|
|tasks.max|NO|1|Kafka连接器可以创建的任务线程数的上限，通常与Kafka Connect集群中工作节点上的CPU核心数相同。您可以调整此参数来控制负载性能。|
|bufferflush.maxbytes|NO|94371840(90M)|一次发送到 StarRocks 之前可以在内存中累积的最大数据大小。最大值范围为 64 MB 至 10 GB。请记住，流加载 SDK 缓冲区可能会创建多个流加载作业来缓冲数据。因此，这里所说的阈值是指总数据大小。|
|bufferflush.intervalms|NO|300000|发送一批数据的时间间隔，控制加载延迟。范围：[1000, 3600000]。|
|connect.timeoutms|NO|1000|连接到 HTTP URL 的超时。范围：[100, 60000]。|
|sink.properties.*|流负载参数 o 控制负载行为。例如，参数sink.properties.format指定用于Stream Load的格式，例如CSV或JSON。有关支持的参数及其描述的列表，请参阅 [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM LOAD.md)。|
|sink.properties.format|NO|json|用于流加载的格式。 Kafka 连接器会将每批数据转换为格式，然后再将其发送到 StarRocks。有效值：csv 和 json。有关详细信息，请参阅 CSV 参数**和** JSON 参数。|

## 限制

- 不支持将 Kafka 主题中的单条消息拆分成多行数据并加载到 StarRocks。
- Kafka 连接器的 Sink 保证至少一次处理语义。

## 最佳实践

### 加载 Debezium 格式的 CDC 数据

如果 Kafka 数据是 Debezium CDC 格式，并且 StarRocks 表是一个主键表，您还需要配置 transforms 参数和其他相关参数。

```Properties
transforms=addfield,unwrap
transforms.addfield.type=com.starrocks.connector.kafka.transforms.AddOpFieldForDebeziumRecord
transforms.unwrap.type=io.debezium.transforms.ExtractNewRecordState
transforms.unwrap.drop.tombstones=true
transforms.unwrap.delete.handling.mode
```

在上述配置中，我们指定了 transforms=addfield,unwrap。

- addfield 转换用于将 __op 字段添加到每条 Debezium CDC 格式数据记录中，以支持 StarRocks 主键模型表。如果 StarRocks 表不是主键表，您不需要指定 addfield 转换。addfield 转换类是 com.Starrocks.Kafka.Transforms.AddOpFieldForDebeziumRecord，已包含在 Kafka 连接器 JAR 文件中，因此无需手动安装。
- unwrap 转换由 Debezium 提供，用于解包 Debezium 的复杂数据结构，基于操作类型。更多信息，请参阅[新记录状态提取](https://debezium.io/documentation/reference/stable/transformations/event-flattening.html)。
