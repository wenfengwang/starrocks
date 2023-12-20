---
displayed_sidebar: English
---

# [预览] 持续从 Apache® Pulsar™ 加载数据

从 StarRocks 版本 2.5 开始，Routine Load 支持从 Apache® Pulsar™ 持续加载数据。Pulsar 是一个分布式的、开源的发布-订阅消息传递和流媒体平台，具有存储-计算分离架构。通过 Routine Load 从 Pulsar 加载数据与从 Apache Kafka 加载数据类似。本主题以 CSV 格式的数据为例，介绍如何通过 Routine Load 从 Apache Pulsar 加载数据。

## 支持的数据文件格式

Routine Load 支持从 Pulsar 集群消费 CSV 和 JSON 格式的数据。

> 注意
> 对于 CSV 格式的数据，StarRocks 支持使用 50 字节以内的 UTF-8 编码字符串作为列分隔符。常用的列分隔符包括逗号 (,)、制表符和竖线 (|)。

## Pulsar 相关概念

**[主题](https://pulsar.apache.org/docs/2.10.x/concepts-messaging/#topics)**

Pulsar 中的主题是用于将消息从生产者传输到消费者的命名通道。Pulsar 中的主题分为分区主题和非分区主题。

- **[分区主题](https://pulsar.apache.org/docs/2.10.x/concepts-messaging/#partitioned-topics)** 是一种特殊类型的主题，由多个 broker 处理，因此允许更高的吞吐量。分区主题实际上是作为 N 个内部主题实现的，其中 N 是分区数。
- **非分区主题** 是一种普通类型的主题，只由单个 broker 服务，这限制了主题的最大吞吐量。

**[消息 ID](https://pulsar.apache.org/docs/2.10.x/concepts-messaging/#messages)**

消息的消息 ID 在消息被 [BookKeeper 实例](https://pulsar.apache.org/docs/2.10.x/concepts-architecture-overview/#apache-bookkeeper) 持久存储后立即分配。消息 ID 指示消息在账本中的特定位置，并且在 Pulsar 集群内是唯一的。

Pulsar 支持消费者通过 consumer.*seek*(*messageId*) 指定初始位置。但与 Kafka 消费者偏移量（一个长整数值）相比，消息 ID 由四部分组成：`ledgerId:entryID:partition-index:batch-index`。

因此，您无法直接从消息中获取消息 ID。结果是，目前 **Routine Load** 不支持在从 Pulsar 加载数据时指定初始位置，只支持从分区的开始或结束消费数据。

**[订阅](https://pulsar.apache.org/docs/2.10.x/concepts-messaging/#subscriptions)**

订阅是一个命名的配置规则，用于确定如何将消息传递给消费者。Pulsar 还支持消费者同时订阅多个主题。一个主题可以有多个订阅。

订阅的类型在消费者连接时定义，并且可以通过用不同配置重启所有消费者来更改。Pulsar 提供四种订阅类型：

- `exclusive`（默认）：仅允许单个消费者附加到订阅。只允许一个消费者消费消息。
- `shared`：多个消费者可以附加到同一订阅。消息以轮询方式分配给消费者，每条消息只被传递给一个消费者。
- `failover`：多个消费者可以附加到同一订阅。为非分区主题或分区主题的每个分区选出一个主消费者接收消息。当主消费者断开连接时，所有（未确认和后续的）消息都将传递给下一个消费者。
- `key_shared`：多个消费者可以附加到同一订阅。消息按消费者分配，具有相同键或相同排序键的消息只被传递给一个消费者。

> 注意：
> 目前 Routine Load 使用 exclusive 类型。

## 创建 Routine Load 作业

以下示例描述了如何消费 Pulsar 中的 CSV 格式消息，并通过创建 Routine Load 作业将数据加载到 StarRocks 中。有关详细说明和参考，请参阅 [CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。

```SQL
CREATE ROUTINE LOAD load_test.routine_wiki_edit_1 ON routine_wiki_edit
COLUMNS TERMINATED BY ",",
ROWS TERMINATED BY "\n",
COLUMNS (order_id, pay_dt, customer_name, nationality, temp_gender, price)
WHERE event_time > "2022-01-01 00:00:00",
PROPERTIES
(
    "desired_concurrent_number" = "1",
    "max_batch_interval" = "15000",
    "max_error_number" = "1000"
)
FROM PULSAR
(
    "pulsar_service_url" = "pulsar://localhost:6650",
    "pulsar_topic" = "persistent://tenant/namespace/topic-name",
    "pulsar_subscription" = "load-test",
    "pulsar_partitions" = "load-partition-0,load-partition-1",
    "pulsar_initial_positions" = "POSITION_EARLIEST,POSITION_LATEST",
    "property.auth.token" = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUJzdWIiOiJqaXV0aWFuY2hlbiJ9.lulGngOC72vE70OW54zcbyw7XdKSOxET94WT_hIqD5Y"
);
```

当创建 Routine Load 来消费来自 Pulsar 的数据时，除了 `data_source_properties` 之外的大多数输入参数与消费来自 Kafka 的数据相同。有关除 `data_source_properties` 之外的参数的说明，请参阅 [CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。

与 `data_source_properties` 相关的参数及其说明如下：

| 参数 | 必填 | 说明 |
| --- | --- | --- |
| pulsar_service_url | 是 | 用于连接 Pulsar 集群的 URL。格式为：“pulsar://ip:port”或“pulsar://service:port”。例如：“pulsar_service_url" = "pulsar://localhost:6650” |
```
|pulsar_topic|是|订阅的主题。例如：“pulsar_topic”=“persistent://tenant/namespace/topic-name”|
|pulsar_subscription|是|为主题配置的订阅。例如："pulsar_subscription" = "my_subscription"|
|pulsar_partitions, pulsar_initial_positions|否|`pulsar_partitions`：主题中订阅的分区。`pulsar_initial_positions`：由`pulsar_partitions`指定的分区的初始位置。初始位置必须对应`pulsar_partitions`中的分区。有效值：`POSITION_EARLIEST`（默认值）：订阅从分区中最早的可用消息开始。`POSITION_LATEST`：订阅从分区中最新的可用消息开始。注意：如果未指定`pulsar_partitions`，则订阅该主题的所有分区。如果同时指定了`pulsar_partitions`和`property.pulsar_default_initial_position`，则`pulsar_partitions`的值会覆盖`property.pulsar_default_initial_position`的值。如果既没有指定`pulsar_partitions`也没有指定`property.pulsar_default_initial_position`，订阅将从分区中最新的可用消息开始。示例："`pulsar_partitions`" = "my-partition-0,my-partition-1,my-partition-2,my-partition-3", "`pulsar_initial_positions`" = "POSITION_EARLIEST,POSITION_EARLIEST,POSITION_LATEST,POSITION_LATEST"|

Routine Load 支持以下针对 Pulsar 的自定义参数。

|参数|必填|说明|
|---|---|---|
|property.pulsar_default_initial_position|否|订阅主题分区时的默认初始位置。当未指定`pulsar_initial_positions`时，该参数生效。其有效值与`pulsar_initial_positions`的有效值相同。例如："`property.pulsar_default_initial_position`" = "POSITION_EARLIEST"`|
|property.auth.token|否|如果 Pulsar 启用了使用安全令牌对客户端进行身份验证，则需要令牌字符串来验证您的身份。例如："`property.auth.token`" = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUJzdWIiOiJqaXV0aWFuY2hlbiJ9.lulGngOC72vE70OW54zcbyw7XdKSOxET94WT_hIqD"`|

## 检查加载作业和任务

### 检查加载作业

执行 [SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation//SHOW_ROUTINE_LOAD.md) 语句以检查加载作业 `routine_wiki_edit_1` 的状态。StarRocks 返回执行状态 `State`、统计信息（包括消耗的总行数和加载的总行数）`Statistics` 以及加载作业的进度 `Progress`。

当您检查从 Pulsar 消费数据的 Routine Load 作业时，除了 `Progress` 之外，大多数返回参数与从 Kafka 消费数据相同。`Progress` 指的是积压，即分区中未确认的消息数。

```Plaintext
MySQL [load_test] > SHOW ROUTINE LOAD for routine_wiki_edit_1 \G
*************************** 1. row ***************************
                  Id: 10142
                Name: routine_wiki_edit_1
          CreateTime: 2022-06-29 14:52:55
           PauseTime: 2022-06-29 17:33:53
             EndTime: NULL
              DbName: default_cluster:test_pulsar
           TableName: test1
               State: PAUSED
      DataSourceType: PULSAR
      CurrentTaskNum: 0
       JobProperties: {"partitions":"*","rowDelimiter":"'\n'","partial_update":"false","columnToColumnExpr":"*","maxBatchIntervalS":"10","whereExpr":"*","timezone":"Asia/Shanghai","format":"csv","columnSeparator":"','","json_root":"","strict_mode":"false","jsonpaths":"","desireTaskConcurrentNum":"3","maxErrorNum":"10","strip_outer_array":"false","currentTaskConcurrentNum":"0","maxBatchRows":"200000"}
DataSourceProperties: {"serviceUrl":"pulsar://localhost:6650","currentPulsarPartitions":"my-partition-0,my-partition-1","topic":"persistent://tenant/namespace/topic-name","subscription":"load-test"}
    CustomProperties: {"auth.token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUJzdWIiOiJqaXV0aWFuY2hlbiJ9.lulGngOC72vE70OW54zcbyw7XdKSOxET94WT_hIqD"}
           Statistic: {"receivedBytes":5480943882,"errorRows":0,"committedTaskNum":696,"loadedRows":66243440,"loadRowsRate":29000,"abortedTaskNum":0,"totalRows":66243440,"unselectedRows":0,"receivedBytesRate":2400000,"taskExecuteTimeMs":2283166}
            Progress: {"my-partition-0(backlog): 100","my-partition-1(backlog): 0"}
ReasonOfStateChanged: 
        ErrorLogUrls: 
            OtherMsg:
1 row in set (0.00 sec)
```

### 检查加载任务

执行 [SHOW ROUTINE LOAD TASK](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD_TASK.md) 语句查看加载作业 `routine_wiki_edit_1` 的加载任务，例如正在运行的任务数、已消费的 Kafka 主题分区和消费进度 `DataSourceProperties`，以及对应的 Coordinator BE 节点 `BeId`。

```SQL
MySQL [example_db]> SHOW ROUTINE LOAD TASK WHERE JobName = "routine_wiki_edit_1" \G
```

## 更改加载作业

在更改加载作业之前，必须使用 [PAUSE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/PAUSE_ROUTINE_LOAD.md) 语句暂停它。然后您可以执行 [ALTER ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md)。更改后，可以执行 [RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) 语句来恢复它，并使用 [SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) 语句检查其状态。

当使用 Routine Load 消费 Pulsar 数据时，除了 `data_source_properties` 之外，大部分返回参数与消费 Kafka 数据相同。

**请注意以下几点**：

- 在 `data_source_properties` 相关参数中，目前仅支持修改 `pulsar_partitions`、`pulsar_initial_positions` 以及自定义 Pulsar 参数 `property.pulsar_default_initial_position` 和 `property.auth.token`。参数 `pulsar_service_url`、`pulsar_topic` 和 `pulsar_subscription` 无法修改。
- 如果需要修改要消费的分区以及匹配的初始位置，需要确保在创建 Routine Load 作业时使用 `pulsar_partitions` 指定了分区，并且只能修改指定分区的初始位置 `pulsar_initial_positions`。
- 如果在创建 Routine Load 作业时仅指定了主题 `pulsar_topic` 而没有指定分区 `pulsar_partitions`，则可以通过 `property.pulsar_default_initial_position` 修改所有分区下主题的起始位置。
```markdown
- 如果在创建 Routine Load 作业时只指定了 Topic `pulsar_topic` 而没有指定分区 `pulsar_partitions`，则可以通过 `property.pulsar_default_initial_position` 修改该 topic 下所有分区的初始位置。