---
displayed_sidebar: "Chinese"
---

# 使用Kafka连接器加载数据

StarRocks提供了一个自研的连接器，名为Apache Kafka®连接器（StarRocks Connector for Apache Kafka®），可连续消费Kafka中的消息并将其加载到StarRocks中。Kafka连接器保证至少一次的语义。

Kafka连接器可以无缝集成Kafka Connect，使StarRocks能更好地与Kafka生态系统集成。如果您想将实时数据加载到StarRocks中，使用Kafka连接器是明智的选择。与常规加载相比，在以下情况下建议使用Kafka连接器：

- 与仅支持CSV、JSON和Avro格式加载数据的常规加载相比，Kafka连接器可以加载更多格式的数据，如Protobuf。只要数据能使用Kafka Connect的转换器转换为JSON和CSV格式，就可以通过Kafka连接器加载数据到StarRocks中。
- 自定义数据转换，例如将Debezium格式的CDC数据。
- 从多个Kafka主题加载数据。
- 从Confluent云加载数据。
- 需要更精细地控制加载批量大小、并行度和其他参数，以在加载速度和资源利用之间实现平衡。

## 准备工作

### 设置Kafka环境

支持自管理的Apache Kafka集群和Confluent云。

- 对于自管理的Apache Kafka集群，请确保部署了Apache Kafka集群和Kafka Connect集群并创建了主题。
- 对于Confluent云，请确保您拥有Confluent帐号并创建了集群和主题。

### 安装Kafka连接器

将Kafka连接器提交到Kafka Connect中：

- 自管理的Kafka集群：

  - 下载并解压缩[starrocks-kafka-connector-1.0.0.tar.gz](https://releases.starrocks.io/starrocks/starrocks-kafka-connector-1.0.0.tar.gz)。
  - 将提取的目录复制到`plugin.path`属性指定的路径中。您可以在Kafka Connect集群中的工作节点的配置文件中找到`plugin.path`属性。

- Confluent云：

  > **注意**
  >
  > Kafka连接器当前未上传到Confluent Hub。您需要将压缩文件上传到Confluent云。

### 创建StarRocks表

根据Kafka主题和数据在StarRocks中创建一个或多个表。

## 示例

以下步骤以自管理的Kafka集群为例，演示如何配置Kafka连接器并启动Kafka Connect（无需重新启动Kafka服务）以将数据加载到StarRocks中。

1. 创建名为**connect-StarRocks-sink.properties**的Kafka连接器配置文件并配置参数。有关参数的详细信息，请参见[参数](#参数)。

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
    > 如果源数据是CDC数据，例如Debezium格式的数据，并且StarRocks表是主键表，还需要[配置`transform`](#加载Debezium格式的CDC数据)以将源数据更改同步到主键表中。

2. 运行Kafka连接器（无需重新启动Kafka服务）。有关以下命令中参数和描述，请参见[Kafka文档](https://kafka.apache.org/documentation.html#connect_running)。

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

    - 请注意，在分布式模式下，不通过命令行传递连接器配置。而是使用下面描述的REST API配置Kafka连接器并运行Kafka连接。

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

3. 查询StarRocks表中的数据。

## 参数

| 参数                                | 必填    | 默认值                                                       | 描述                                                         |
| ------------------------------------- | -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| name                                 | 是       |                                                              | 此Kafka连接器的名称。在此Kafka Connect集群中的所有Kafka连接器中必须是全局唯一的。例如，starrocks-kafka-connector。 |
| connector.class                      | 是       | com.starrocks.connector.kafka.SinkConnector                  | 此Kafka连接器的sink使用的类。                              |
| topics                               | 是       |                                                              | 要订阅的一个或多个主题，每个主题对应一个StarRocks表。默认情况下，StarRocks假定主题名称与StarRocks表名相匹配。因此，StarRocks使用主题名称确定目标StarRocks表。请选择填写`topics`或`topics.regex`（下文）中的一个，而不是同时填写两者。但是，如果StarRocks表名与主题名不相同，那么使用可选的`starrocks.topic2table.map`参数（下文）来指定主题名到表名的映射。 |
| topics.regex                        |          | 用于匹配要订阅的一个或多个主题的正则表达式。有关更多描述，请参见`topics`。请选择填写`topics.regex`或`topics`（上文）中的一个。|                                                              |
| starrocks.topic2table.map           | 否       |                                                              | 当主题名与StarRocks表名不同时，主题名和表名的映射。格式为`<topic-1>:<table-1>,<topic-2>:<table-2>,...`。 |
| starrocks.http.url                  | 是       |                                                              | StarRocks集群中前端的HTTP URL。格式为`<fe_host1>:<fe_http_port1>,<fe_host2>:<fe_http_port2>,...`。多个地址以逗号（,）分隔。例如，`192.168.xxx.xxx:8030,192.168.xxx.xxx:8030`。 |
| starrocks.database.name             | 是       |                                                              | StarRocks数据库的名称。                                       |
| starrocks.username                  | 是       |                                                              | 您的StarRocks集群帐户的用户名。用户需要在StarRocks表上具有[INSERT](../sql-reference/sql-statements/account-management/GRANT.md)权限。 |
| starrocks.password                  | 是       |                                                              | 您的StarRocks集群帐户的密码。                                 |
| key.converter                       | 否       | Kafka Connect集群使用的键转换器                           | 此参数指定sink连接器（Kafka-connector-starrocks）的键转换器，用于反序列化Kafka数据的键。默认的键转换器是Kafka Connect集群使用的键转换器。 |
| value.converter                     | 否       | Kafka Connect集群使用的值转换器                           | 此参数指定sink连接器（Kafka-connector-starrocks）的值转换器，用于反序列化Kafka数据的值。默认的值转换器是Kafka Connect集群使用的值转换器。 |
| key.converter.schema.registry.url   | 否       |                                                              | 键转换器的模式注册中心URL。                                  |
| value.converter.schema.registry.url | 否       |                                                              | 值转换器的模式注册中心URL。                                  |
| tasks.max                           | 否       | 1                                                          | Kafka连接器可以创建的任务线程数的上限，通常与Kafka Connect集群中工作节点上的CPU核心数相同。您可以调整此参数以控制负载性能。 |
| bufferflush.maxbytes                | 否       | 94371840（90M）                                              | 在一次发送到StarRocks之前可以在内存中累积的数据的最大大小。最大值范围从64 MB到10 GB。请记住，流加载SDK缓冲区可能会创建多个流加载作业以缓冲数据。因此，这里提到的阈值是指总数据大小。 |
| bufferflush.intervalms              | 否       | 300000                                                     | 发送一批数据的间隔，控制加载延迟。范围：[1000, 3600000]。 |
| connect.timeoutms                   | NO       | 1000                                                         | 连接到HTTP URL的超时时间。范围：[100, 60000]。 |
| sink.properties.*                   |          |                                                              | 流加载参数以控制加载行为。例如，参数`sink.properties.format`指定用于流加载的格式，例如CSV或JSON。有关受支持参数及其描述的列表，请参阅[流加载](../sql-reference/sql-statements/data-manipulation/STREAM LOAD.md)。 |
| sink.properties.format              | NO       | json                                                         | 用于流加载的格式。Kafka连接器将在将每个数据批次发送到StarRocks之前将其转换为该格式。有效值：`csv`和`json`。有关更多信息，请参阅[CSV parameters**](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md#csv-parameters)和**[JSON parameters](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md#json-parameters)。 |

## 限制

- 不支持将Kafka主题中的单个消息展平为多个数据行并加载到StarRocks中。
- Kafka连接器的Sink保证至少一次语义。

## 最佳实践

### 加载Debezium格式的CDC数据

如果Kafka数据以Debezium CDC格式呈现且StarRocks表是主键表，则还需要配置`transforms`参数及其他相关参数。

```Properties
transforms=addfield,unwrap
transforms.addfield.type=com.starrocks.connector.kafka.transforms.AddOpFieldForDebeziumRecord
transforms.unwrap.type=io.debezium.transforms.ExtractNewRecordState
transforms.unwrap.drop.tombstones=true
transforms.unwrap.delete.handling.mode
```

在上述配置中，我们指定了`transforms=addfield,unwrap`。

- `addfield` 转换用于向每个Debezium CDC格式数据记录添加`__op`字段，以支持StarRocks主键模型表。如果StarRocks表不是主键表，则无需指定`addfield` 转换。`addfield` 转换类为`com.starrocks.connector.kafka.transforms.AddOpFieldForDebeziumRecord`。它包含在Kafka连接器JAR文件中，因此无需手动安装。
- `unwrap` 转换由Debezium提供，用于根据操作类型展开Debezium的复杂数据结构。有关更多信息，请参阅[New Record State Extraction](https://debezium.io/documentation/reference/stable/transformations/event-flattening.html)。