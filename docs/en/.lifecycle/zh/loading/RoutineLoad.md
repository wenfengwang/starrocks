---
displayed_sidebar: English
---

# 从 Apache Kafka® 持续加载数据

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

本主题介绍如何创建 Routine Load 作业以将 Kafka 消息（事件）流式传输到 StarRocks 中，并让您熟悉有关 Routine Load 的一些基本概念。

要将流中的消息持续加载到 StarRocks 中，您可以将消息流存储在 Kafka 主题中，并创建 Routine Load 作业来消费消息。Routine Load 作业在 StarRocks 中持久化，生成一系列加载任务来消费主题中全部或部分分区中的消息，并将消息加载到 StarRocks 中。

Routine Load 作业支持精确一次交付语义，以保证加载到 StarRocks 中的数据既不会丢失也不会重复。

Routine Load 支持在数据加载时进行数据转换，并支持在数据加载过程中通过 UPSERT 和 DELETE 操作进行的数据变更。有关详细信息，请参阅[在加载时转换数据](../loading/Etl_in_loading.md)和[通过加载更改数据](../loading/Load_to_Primary_Key_tables.md)。

<InsertPrivNote />


## 支持的数据格式

Routine Load 现在支持从 Kafka 集群消费 CSV、JSON 和 Avro（自 v3.0.1 起支持）格式的数据。

> **注意**
> 对于 CSV 数据，请注意以下几点：
- 您可以使用长度不超过 50 个字节的 UTF-8 字符串，如逗号（,）、制表符或竖线（|），作为文本分隔符。
- 空值用 `\N` 表示。例如，一个数据文件由三列组成，而该数据文件中的一条记录在第一列和第三列中有数据，但第二列中没有数据。在这种情况下，您需要在第二列中使用 `\N` 来表示空值。这意味着记录必须编译为 `a,\N,b` 而不是 `a,,b`。`a,,b` 表示记录的第二列包含一个空字符串。

## 基本概念

![routine load](../assets/4.5.2-1.png)

### 术语

- **加载作业**

  Routine Load 作业是一个长期运行的作业。只要其状态为 RUNNING，加载作业就会持续生成一个或多个并发加载任务，这些任务消费 Kafka 集群中某个主题的消息并将数据加载到 StarRocks 中。

- **加载任务**

  加载作业根据特定规则分割为多个加载任务。加载任务是数据加载的基本单元。作为一个独立事件，加载任务基于 [Stream Load](../loading/StreamLoad.md) 实现加载机制。多个加载任务并发消费来自不同分区的主题消息，并将数据加载到 StarRocks 中。

### 工作流程

1. **创建 Routine Load 作业**。
要从 Kafka 加载数据，您需要通过运行 [CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) 语句创建 Routine Load 作业。FE 解析该语句，并根据您指定的属性创建作业。

2. **FE 将作业拆分为多个加载任务**。

   FE 根据特定规则将作业拆分为多个加载任务。每个加载任务都是一个独立的事务。
拆分规则如下：
   -  FE 根据期望的并发数 `desired_concurrent_number`、Kafka 主题中的分区数以及存活的 BE 节点数计算加载任务的实际并发数。
   -  FE 根据计算出的实际并发数将作业拆分为加载任务，并将任务排列到任务队列中。

   每个 Kafka 主题由多个分区组成。主题分区与加载任务的关系如下：
   -  一个分区被唯一分配给一个加载任务，来自该分区的所有消息都由该加载任务消费。
   -  一个加载任务可以消费来自一个或多个分区的消息。
   -  所有分区在加载任务之间均匀分布。

3. **多个加载任务并发运行，消费来自多个 Kafka 主题分区的消息，并将数据加载到 StarRocks 中**

1.    **FE 调度和提交加载任务**：FE 定时调度队列中的加载任务，并将它们分配给选定的 Coordinator BE 节点。加载任务之间的间隔由配置项 `max_batch_interval` 定义。FE 将加载任务均匀分配给所有 BE 节点。有关 `max_batch_interval` 的更多信息，请参阅 [CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md#example)。

2.    Coordinator BE 启动加载任务，消费分区中的消息，解析和过滤数据。加载任务持续到消费完预定的消息量或达到预定的时间限制。消息批量大小和时间限制在 FE 配置 `max_routine_load_batch_size` 和 `routine_load_task_consume_second` 中定义。有关详细信息，请参阅 [配置](../administration/BE_configuration.md)。然后，Coordinator BE 将消息分发给 Executor BE。Executor BE 将消息写入磁盘。

            > **注意**
            > StarRocks 支持通过安全认证机制 SASL_SSL、SASL 或 SSL，或者无认证访问 Kafka。本节以无认证连接 Kafka 为例。如果您需要通过安全认证机制连接到 Kafka，请参阅 [CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。

4. **FE 生成新的加载任务以持续加载数据**。
在 Executor BE 将数据写入磁盘后，Coordinator BE 向 FE 报告加载任务的结果。根据结果，FE 生成新的加载任务以持续加载数据。或者，FE 重试失败的任务以确保加载到 StarRocks 中的数据既不会丢失也不会重复。

## 创建 Routine Load 作业

以下三个示例描述了如何在 Kafka 中消费 CSV 格式、JSON 格式和 Avro 格式的数据，并通过创建 Routine Load 作业将数据加载到 StarRocks 中。有关详细的语法和参数说明，请参阅 [CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。

### 加载 CSV 格式数据

本节介绍如何创建 Routine Load 作业来消费 Kafka 集群中的 CSV 格式数据，并将数据加载到 StarRocks 中。

#### 准备数据集

假设 Kafka 集群中的主题 `ordertest1` 中有一个 CSV 格式的数据集。数据集中的每条消息都包含六个字段：订单 ID、付款日期、客户姓名、国籍、性别和价格。

```Plain
2020050802,2020-05-08,Johann Georg Faust,Deutschland,male,895
2020050802,2020-05-08,Julien Sorel,France,male,893
2020050803,2020-05-08,Dorian Grey,UK,male,1262
2020050901,2020-05-09,Anna Karenina",Russia,female,175
2020051001,2020-05-10,Tess Durbeyfield,US,female,986
2020051101,2020-05-11,Edogawa Conan,japan,male,8924
```

#### 创建表

根据 CSV 格式数据的字段，在数据库 `example_db` 中创建表 `example_tbl1`。以下示例创建一个包含 5 个字段的表，不包括 CSV 格式数据中的客户性别字段。

```SQL
CREATE TABLE example_db.example_tbl1 ( 
    `order_id` bigint NOT NULL COMMENT "Order ID",
    `pay_dt` date NOT NULL COMMENT "Payment date", 
    `customer_name` varchar(26) NULL COMMENT "Customer name", 
    `nationality` varchar(26) NULL COMMENT "Nationality", 
    `price` double NULL COMMENT "Price"
) 
ENGINE=OLAP 
DUPLICATE KEY (`order_id`, `pay_dt`) 
DISTRIBUTED BY HASH(`order_id`); 
```

> **注意**
> 从 v2.5.7 版本开始，StarRocks 可以在创建表或添加分区时自动设置桶（BUCKETS）的数量。您不再需要手动设置桶的数量。有关详细信息，请参阅[确定桶的数量](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

#### 提交 Routine Load 作业

执行以下语句提交名为 `example_tbl1_ordertest1` 的 Routine Load 作业，消费主题 `ordertest1` 中的消息，并将数据加载到表 `example_tbl1` 中。加载任务从主题指定分区的初始偏移量消费消息。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, temp_gender, price)
PROPERTIES
(
    "desired_concurrent_number" = "5"
)
FROM KAFKA
(
    "kafka_broker_list" = "<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1",
    "kafka_partitions" = "0,1,2,3,4",
    "property.kafka_default_offsets" = "OFFSET_BEGINNING"
);
```

提交加载作业后，您可以执行 [SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) 语句来检查加载作业的状态。

- **加载作业名称**

  一个表上可能有多个加载作业。因此，我们建议您以对应的 Kafka 主题和提交加载作业的时间来命名加载作业。这有助于您区分每个表上的加载作业。

- **列分隔符**

  属性 `COLUMN TERMINATED BY` 定义 CSV 格式数据的列分隔符。默认值是 `\t`。

- **Kafka 主题分区和偏移量**

  您可以指定属性 `kafka_partitions` 和 `kafka_offsets` 来指定消费消息的分区和偏移量。例如，如果您希望加载作业从主题 `ordertest1` 的 Kafka 分区 `"0,1,2,3,4"` 消费消息，并且所有分区都使用初始偏移量，您可以如下指定属性：如果您希望加载作业从 Kafka 分区 `"0,1,2,3,4"` 消费消息，并且需要为每个分区指定单独的起始偏移量，您可以如下配置：

  ```SQL
  "kafka_partitions" = "0,1,2,3,4",
  "kafka_offsets" = "OFFSET_BEGINNING, OFFSET_END, 1000, 2000, 3000"
  ```

  您还可以使用属性 `property.kafka_default_offsets` 设置所有分区的默认偏移量。

  ```SQL
  "kafka_partitions" = "0,1,2,3,4",
  "property.kafka_default_offsets" = "OFFSET_BEGINNING"
  ```
```
"confluent.schema.registry.url" = "http://172.xx.xxx.xxx:8081",  
    "kafka_topic" = "topic_0",  
    "kafka_partitions" = "0,1,2,3,4,5",  
    "property.kafka_default_offsets" = "OFFSET_BEGINNING"  
);
```

- 数据格式

  您需要在 PROPERTIES 子句中指定 `"format" = "avro"`，以定义数据格式为 Avro。

- 架构注册表

  您需要配置 `confluent.schema.registry.url`，以指定注册 Avro 架构的架构注册表的 URL。StarRocks 通过使用此 URL 检索 Avro 架构。格式如下：

  ```Plaintext
  confluent.schema.registry.url = http[s]://[<schema-registry-api-key>:<schema-registry-api-secret>@]<hostname|ip address>[:<port>]
  ```

- 数据映射和转换

  要指定 Avro 格式数据与 StarRocks 表之间的映射和转换关系，需要指定参数 `COLUMNS` 和属性 `jsonpaths`。`COLUMNS` 参数中指定的字段顺序必须与 `jsonpaths` 属性中的字段顺序匹配，并且字段名称必须与 StarRocks 表的名称匹配。属性 `jsonpaths` 用于从 Avro 数据中提取所需的字段。然后，这些字段由属性 `COLUMNS` 命名。

  有关数据转换的更多信息，请参阅[加载时转换数据](https://docs.starrocks.io/en-us/latest/loading/Etl_in_loading)。

    > **注意**
    > 如果要加载的 Avro 记录的字段名称和数量与 StarRocks 表的列完全匹配，则无需指定 `COLUMNS` 参数。

提交加载作业后，您可以执行[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)语句来检查加载作业的状态。

#### 数据类型映射

要加载的 Avro 数据字段与 StarRocks 表列之间的数据类型映射如下：

##### 基本类型

|Avro|StarRocks|
|---|---|
|nul|NULL|
|boolean|BOOLEAN|
|int|INT|
|long|BIGINT|
|float|FLOAT|
|double|DOUBLE|
|bytes|STRING|
|string|STRING|

##### 复杂类型

|Avro|StarRocks|
|---|---|
|record|将整个 RECORD 或其子字段作为 JSON 加载到 StarRocks 中。|
|enums|STRING|
|arrays|ARRAY|
|maps|JSON|
|union(T, null)|NULLABLE(T)|
|fixed|STRING|

#### 限制

- 目前，StarRocks 不支持模式演化。
- 每个 Kafka 消息只能包含一个 Avro 数据记录。

## 检查加载作业和任务

### 检查加载作业

执行[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)语句来检查加载作业`example_tbl2_ordertest2`的状态。StarRocks 返回执行状态`State`、统计信息（包括消费的总行数和加载的总行数）`Statistics`和加载作业的进度`progress`。

如果加载作业的状态自动更改为**PAUSED**，可能是因为错误行数超过了阈值。有关设置此阈值的详细说明，请参见[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。您可以检查文件`ReasonOfStateChanged`和`ErrorLogUrls`以识别和排除问题。修复问题后，可以执行[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md)语句以恢复**PAUSED**加载作业。

如果加载作业的状态为**CANCELLED**，可能是因为加载作业遇到异常（例如表已被删除）。您可以检查文件`ReasonOfStateChanged`和`ErrorLogUrls`以识别和排除问题。但是，您无法恢复已取消的加载作业。

```SQL
MySQL [example_db]> SHOW ROUTINE LOAD FOR example_tbl2_ordertest2 \G
*************************** 1. row ***************************
                  Id: 63013
                Name: example_tbl2_ordertest2
          CreateTime: 2022-08-10 17:09:00
           PauseTime: NULL
             EndTime: NULL
              DbName: default_cluster:example_db
           TableName: example_tbl2
               State: RUNNING
      DataSourceType: KAFKA
      CurrentTaskNum: 3
       JobProperties: {"partitions":"*","partial_update":"false","columnToColumnExpr":"commodity_id,customer_name,country,pay_time,pay_dt=from_unixtime(`pay_time`, '%Y%m%d'),price","maxBatchIntervalS":"20","whereExpr":"*","dataFormat":"json","timezone":"Asia/Shanghai","format":"json","json_root":"","strict_mode":"false","jsonpaths":"[\"$.commodity_id\",\"$.customer_name\",\"$.country\",\"$.pay_time\",\"$.price\"]","desireTaskConcurrentNum":"3","maxErrorNum":"0","strip_outer_array":"false","currentTaskConcurrentNum":"3","maxBatchRows":"200000"}
DataSourceProperties: {"topic":"ordertest2","currentKafkaPartitions":"0,1,2,3,4","brokerList":"<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>"}
    CustomProperties: {"kafka_default_offsets":"OFFSET_BEGINNING"}
           Statistic: {"receivedBytes":230,"errorRows":0,"committedTaskNum":1,"loadedRows":2,"loadRowsRate":0,"abortedTaskNum":0,"totalRows":2,"unselectedRows":0,"receivedBytesRate":0,"taskExecuteTimeMs":522}
            Progress: {"0":"1","1":"OFFSET_ZERO","2":"OFFSET_ZERO","3":"OFFSET_ZERO","4":"OFFSET_ZERO"}
ReasonOfStateChanged: 
        ErrorLogUrls: 
            OtherMsg: 
```

> **注意**
> 无法检查已停止或尚未启动的加载作业的状态。

### 检查加载任务

执行[SHOW ROUTINE LOAD TASK](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD_TASK.md)语句来检查加载作业`example_tbl2_ordertest2`的加载任务，例如当前运行的任务数、消费的 Kafka 主题分区和消费进度`DataSourceProperties`，以及相应的协调器 BE 节点`BeId`。

```SQL
MySQL [example_db]> SHOW ROUTINE LOAD TASK WHERE JobName = "example_tbl2_ordertest2" \G
*************************** 1. row ***************************
              TaskId: 18c3a823-d73e-4a64-b9cb-b9eced026753
               TxnId: -1
           TxnStatus: UNKNOWN
               JobId: 63013
          CreateTime: 2022-08-10 17:09:05
   LastScheduledTime: 2022-08-10 17:47:27
    ExecuteStartTime: NULL
             Timeout: 60
                BeId: -1
DataSourceProperties: {"1":0,"4":0}
             Message: there is no new data in kafka, wait for 20 seconds to schedule again
*************************** 2. row ***************************
              TaskId: f76c97ac-26aa-4b41-8194-a8ba2063eb00
               TxnId: -1
           TxnStatus: UNKNOWN
               JobId: 63013
          CreateTime: 2022-08-10 17:09:05
   LastScheduledTime: 2022-08-10 17:47:26
    ExecuteStartTime: NULL
             Timeout: 60
                BeId: -1
DataSourceProperties: {"2":0}
             Message: there is no new data in kafka, wait for 20 seconds to schedule again
*************************** 3. row ***************************
              TaskId: 1a327a34-99f4-4f8d-8014-3cd38db99ec6
               TxnId: -1
           TxnStatus: UNKNOWN
               JobId: 63013
          CreateTime: 2022-08-10 17:09:26
   LastScheduledTime: 2022-08-10 17:47:27
    ExecuteStartTime: NULL
             Timeout: 60
                BeId: -1
DataSourceProperties: {"0":2,"3":0}
             Message: there is no new data in kafka, wait for 20 seconds to schedule again
```

## 暂停加载作业

您可以执行[PAUSE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/PAUSE_ROUTINE_LOAD.md)语句来暂停加载作业。执行该语句后，加载作业的状态将变为**PAUSED**，但它并未停止。您可以执行[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md)语句来恢复加载作业。您还可以使用[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)语句来检查其状态。

以下示例暂停加载作业`example_tbl2_ordertest2`：

```SQL
PAUSE ROUTINE LOAD FOR example_tbl2_ordertest2;
```

## 恢复加载作业

您可以执行[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md)语句来恢复暂停的加载作业。执行该语句后，加载作业的状态将暂时变为**NEED_SCHEDULE**（因为加载作业正在重新调度），然后变为**RUNNING**。您可以使用[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)语句来检查其状态。

以下示例恢复暂停的加载作业`example_tbl2_ordertest2`：

```SQL
RESUME ROUTINE LOAD FOR example_tbl2_ordertest2;
```

## 修改加载作业

在修改加载作业之前，您必须使用[PAUSE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/PAUSE_ROUTINE_LOAD.md)语句暂停加载作业。然后，您可以执行[ALTER ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md)语句。修改后，您可以执行[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md)语句来恢复加载作业，并使用[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)语句来检查其状态。

假设存活的 BE 节点数量增加到 `6`，要消费的 Kafka 主题分区为 `"0,1,2,3,4,5,6,7"`。如果要增加实际加载任务并发度，可以执行以下语句将所需任务并发度 `desired_concurrent_number` 增加到 `6`（大于或等于存活的 BE 节点数量），并指定要消费的 Kafka 主题分区和初始偏移量。

> **注意**
> 由于实际任务并发度由多个参数的最小值确定，您必须确保 FE 的动态参数 `max_routine_load_task_concurrent_num` 的值大于或等于 `6`。

```SQL
ALTER ROUTINE LOAD FOR example_tbl2_ordertest2
PROPERTIES
(
    "desired_concurrent_number" = "6"
)
FROM kafka
(
    "kafka_partitions" = "0,1,2,3,4,5,6,7",
    "kafka_offsets" = "OFFSET_BEGINNING,OFFSET_BEGINNING,OFFSET_BEGINNING,OFFSET_BEGINNING,OFFSET_END,OFFSET_END,OFFSET_END,OFFSET_END"
);
```

## 停止加载作业

您可以执行[STOP ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/STOP_ROUTINE_LOAD.md)语句来停止加载作业。执行该语句后，加载作业的状态将变为**STOPPED**，您无法恢复已停止的加载作业。您无法使用[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)语句检查已停止的加载作业的状态。

以下示例停止加载作业`example_tbl2_ordertest2`：

```SQL
STOP ROUTINE LOAD FOR example_tbl2_ordertest2;
```

## 常见问题

请参见[Routine Load FAQ](../faq/loading/Routine_load_faq.md)。
"kafka_offsets" = "OFFSET_BEGINNING,OFFSET_BEGINNING,OFFSET_BEGINNING,OFFSET_BEGINNING,OFFSET_END,OFFSET_END,OFFSET_END,OFFSET_END"
);
```

## 停止加载作业

您可以执行 [STOP ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/STOP_ROUTINE_LOAD.md) 语句来停止加载作业。该语句执行后，加载作业的状态将变为 **STOPPED**，您无法恢复已停止的加载作业。您无法使用 [SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) 语句检查已停止的加载作业的状态。

以下示例停止加载作业 example_tbl2_ordertest2：

```SQL
STOP ROUTINE LOAD FOR example_tbl2_ordertest2;
```

## 常见问题解答

请参阅 [加载常见问题解答](../faq/loading/Routine_load_faq.md)。