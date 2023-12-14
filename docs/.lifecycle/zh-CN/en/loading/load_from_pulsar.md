---
displayed_sidebar: "中文"
---

# [预览] 从Apache® Pulsar™连续加载数据

自 StarRocks 版本 2.5 起，Routine Load 支持从 Apache® Pulsar™ 连续加载数据。Pulsar 是一个分布式、开源的发布-订阅消息传递和流式处理平台，具有存储-计算分离架构。通过 Routine Load 从 Pulsar 加载数据类似于从 Apache Kafka 加载数据。本主题以 CSV 格式的数据为例介绍如何通过 Routine Load 从 Apache Pulsar 加载数据。

## 支持的数据文件格式

Routine Load 支持从 Pulsar 集群消费 CSV 和 JSON 格式的数据。

> 注意
>
> 关于 CSV 格式的数据，StarRocks 支持以 UTF-8 编码的字符串作为列分隔符，且长度不超过50字节。常用的列分隔符包括逗号(,)、制表符和竖线(|)。

## Pulsar 相关概念

**[Topic](https://pulsar.apache.org/docs/2.10.x/concepts-messaging/#topics)**

Pulsar 中的主题是生产者向消费者传递消息的命名通道。Pulsar 的主题分为分区主题和非分区主题。

- **[分区主题](https://pulsar.apache.org/docs/2.10.x/concepts-messaging/#partitioned-topics)** 是一种特殊类型的主题，由多个代理处理，从而实现更高的吞吐量。分区主题实际上由 N 个内部主题实现，其中 N 是分区数。
- **非分区主题** 是一种普通类型的主题，仅由单个代理提供服务，从而限制了主题的最大吞吐量。

**[消息 ID](https://pulsar.apache.org/docs/2.10.x/concepts-messaging/#messages)**

消息的消息 ID 是由[BookKeeper 实例](https://pulsar.apache.org/docs/2.10.x/concepts-architecture-overview/#apache-bookkeeper)在消息被持久化存储后立即分配的。消息 ID 表示消息在分类帐中的特定位置，并且在 Pulsar 集群中是唯一的。

Pulsar 支持消费者通过 consumer.*seek*(*messageId*) 指定初始位置。但是与长整数值的 Kafka 消费者偏移量相比，消息 ID 由四部分组成：`ledgerId:entryID:partition-index:batch-index`。

因此，您无法直接从消息中获取消息 ID。因此，目前 **Routine Load 不支持在从 Pulsar 加载数据时指定初始位置，只支持从分区的开头或结尾消费数据。**

**[订阅](https://pulsar.apache.org/docs/2.10.x/concepts-messaging/#subscriptions)**

订阅是确定消息如何传递给消费者的命名配置规则。Pulsar 还支持消费者同时订阅多个主题。一个主题可以有多个订阅。

订阅的类型是在消费者连接到订阅时定义的，并且该类型可以通过使用不同的配置重新启动所有消费者来更改。Pulsar 提供四种订阅类型：

- `exclusive` (默认): 仅允许单个消费者连接到订阅。只允许一个消费者消费消息。
- `shared`: 多个消费者可以连接到相同的订阅。消息以循环分发的方式传递给消费者，并且任何给定的消息仅传递给一个消费者。
- `failover`: 多个消费者可以连接到相同的订阅。为非分区主题或分区主题的每个分区选择一个主消费者并接收消息。当主消费者断开连接时，所有（未确认和后续的）消息都将传递给下一个排队的消费者。
- `key_shared`: 多个消费者可以连接到相同的订阅。消息以分发方式传递给消费者，具有相同键或相同排序键的消息仅传递给一个消费者。

> 注意
>
> 当前 Routine Load 使用 exclusive 类型。

## 创建 Routine Load 作业

以下示例描述了如何从 Pulsar 消费 CSV 格式的消息，并通过创建 Routine Load 作业将数据加载到 StarRocks 中。有关详细说明和参考，请参阅 [创建 Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。

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

当创建 Routine Load 以从 Pulsar 消费数据时，除了 `data_source_properties` 之外，大多数输入参数与从 Kafka 消费数据相同。有关除 `data_source_properties`外的参数的描述，请参阅[创建 Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。

与 `data_source_properties` 相关的参数及其描述如下:

| **参数**                                    | **是否必需** | **描述**                                                     |
| ------------------------------------------- | ------------ | ------------------------------------------------------------ |
| pulsar_service_url                          | 是           | 用于连接到 Pulsar 集群的 URL。格式：`"pulsar://ip:port"` 或 `"pulsar://service:port"`。示例：`"pulsar_service_url" = "pulsar://localhost:6650"` |
| pulsar_topic                                | 是           | 订阅的主题。示例："pulsar_topic" = "persistent://tenant/namespace/topic-name" |
| pulsar_subscription                         | 是           | 为主题配置的订阅。示例：`"pulsar_subscription" = "my_subscription"` |
| pulsar_partitions, pulsar_initial_positions | 否           | `pulsar_partitions`: 主题中的已订阅分区。`pulsar_initial_positions`: 指定的分区的初始位置。 初始位置必须对应于 `pulsar_partitions` 中的分区。有效值：`POSITION_EARLIEST`（默认值）：订阅从分区中最早可用的消息开始。 `POSITION_LATEST`：订阅从分区中最新可用的消息开始。注意：如果未指定`pulsar_partitions`，则会订阅主题的所有分区。如果同时指定了 `pulsar_partitions` 和 `property.pulsar_default_initial_position`，则`pulsar_partitions` 值会覆盖 `property.pulsar_default_initial_position` 值。如果未同时指定 `pulsar_partitions` 和 `property.pulsar_default_initial_position`，则订阅将从分区中最新可用的消息开始。示例：`"pulsar_partitions" = "my-partition-0,my-partition-1,my-partition-2,my-partition-3", "pulsar_initial_positions" = "POSITION_EARLIEST,POSITION_EARLIEST,POSITION_LATEST,POSITION_LATEST"` |

Routine Load 支持 Pulsar 的以下自定义参数。

| 参数                                     | 是否必需 | 描述                                                         |
| ---------------------------------------- | -------- | ------------------------------------------------------------ |
| property.pulsar_default_initial_position | 否       | 当订阅主题的分区时，默认的初始位置。当未指定 `pulsar_initial_positions` 时，该参数生效。其有效值与 `pulsar_initial_positions` 的有效值相同。示例：`"property.pulsar_default_initial_position" = "POSITION_EARLIEST"` |
| property.auth.token                      | 否       | 如果 Pulsar 启用使用安全令牌验证客户端，您需要令牌字符串来验证您的身份。示例：`"property.auth.token" = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUJzdWIiOiJqaXV0aWFuY2hlbiJ9.lulGngOC72vE70OW54zcbyw7XdKSOxET94WT_hIqD"` |

## 检查加载作业和任务

### 检查加载作业

执行 [SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation//SHOW_ROUTINE_LOAD.md) 语句，以检查加载作业 `routine_wiki_edit_1` 的状态。StarRocks 返回执行状态 `State`、统计信息（包括总行数消费和总行数加载）`Statistics`，以及加载作业的进度 `progress`。

```plaintext
当您检查从 Pulsar 消耗数据的 Routine Load 作业时，除了 `progress` 之外，大多数返回的参数都与从 Kafka 消耗数据相同。`progress` 指的是积压，即分区中未确认消息的数量。

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

执行[SHOW ROUTINE LOAD TASK](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD_TASK.md)语句来检查加载作业 `routine_wiki_edit_1` 的加载任务，例如正在运行的任务数量，被消耗的 Kafka 主题分区和消费进度 `DataSourceProperties`，以及相应的协调器 BE 节点 `BeId`。

```SQL
MySQL [example_db]> SHOW ROUTINE LOAD TASK WHERE JobName = "routine_wiki_edit_1" \G
```

## 修改加载作业

在修改加载作业之前，您必须使用[PAUSE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/PAUSE_ROUTINE_LOAD.md)语句将其暂停。然后，您可以执行[ALTER ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md)。修改后，您可以执行[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md)语句来恢复它，并使用[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)语句来检查其状态。

当 Routine Load 用于从 Pulsar 消耗数据时，除了 `data_source_properties` 之外，大多数返回的参数都与从 Kafka 消耗数据相同。

**请注意以下几点**：

- 在与 `data_source_properties` 相关的参数中，目前仅支持修改 `pulsar_partitions`、`pulsar_initial_positions` 以及自定义 Pulsar 参数 `property.pulsar_default_initial_position` 和 `property.auth.token`。参数 `pulsar_service_url`、`pulsar_topic` 和 `pulsar_subscription` 不能被修改。
- 如果需要修改要消耗的分区和匹配的初始位置，则需要确保在创建 Routine Load 作业时使用 `pulsar_partitions` 来指定分区，只能修改指定分区的初始位置 `pulsar_initial_positions`。
- 如果在创建 Routine Load 作业时仅指定了 Topic `pulsar_topic`，而没有指定分区 `pulsar_partitions`，则可以通过 `pulsar_default_initial_position` 修改主题下所有分区的起始位置。