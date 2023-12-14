---
displayed_sidebar: "中文"
---

# 从 Apache Kafka® 连续加载数据

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

本主题介绍如何创建例行加载作业以将 Kafka 消息（事件）流式传输到 StarRocks，并使您熟悉一些关于例行加载的基本概念。

要将流的消息连续加载到 StarRocks，您可以将消息流存储在 Kafka 主题中，并创建一个例行加载作业来消费这些消息。例行加载作业持久存在于 StarRocks 中，生成一系列加载任务以消费主题中的所有或部分分区中的消息，并将消息加载到 StarRocks 中。

例行加载作业支持精确一次交付语义，以确保加载到 StarRocks 中的数据既不丢失也不重复。

例行加载支持在数据加载时进行数据转换，并支持 UPSERT 和 DELETE 操作在数据加载期间对数据进行更改。有关更多信息，请参阅[加载过程中的数据转换](../loading/Etl_in_loading.md)和[加载到主键表中的数据更改](../loading/Load_to_Primary_Key_tables.md)。

<InsertPrivNote />

## 支持的数据格式

例行加载现在支持从 Kafka 集群中消费 CSV、JSON 和 Avro（自 v3.0.1 起支持）格式的数据。

> **注意**
>
> 对于 CSV 数据，请注意以下几点：
>
> - 您可以使用 UTF-8 字符串，如逗号（,）、制表符或者竖线（|），其长度不超过 50 个字节作为文本分隔符。
> - 空值使用 `\N` 表示。例如，数据文件包含三列，数据文件中的一条记录包含第一列和第三列的数据，但不包含第二列的数据。在这种情况下，您需要在第二列使用 `\N` 表示空值。这意味着记录必须编译为 `a,\N,b` 而不是 `a,,b`。`a,,b` 表示记录的第二列为空字符串。

## 基本概念

![routine load](../assets/4.5.2-1.png)

### 术语

- **加载作业**

   例行加载作业是一个长时间运行的作业。只要其状态为 RUNNING，加载作业就会连续生成一个或多个并发的加载任务，这些任务会消费 Kafka 集群的主题中的消息，并将数据加载到 StarRocks 中。

- **加载任务**

  加载作业根据特定规则分成多个加载任务。加载任务是数据加载的基本单元。作为一个独立的事件，加载任务实现基于[流加载](../loading/StreamLoad.md)的加载机制。多个加载任务同时从主题的不同分区中消费消息，并将数据加载到 StarRocks 中。

### 工作流程

1. **创建例行加载作业。**
   要从 Kafka 加载数据，您需要通过运行[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)语句来创建例行加载作业。前端解析该语句，并根据您指定的属性创建作业。

2. **前端（FE）将作业拆分为多个加载任务。**

    前端根据特定规则将作业拆分为多个加载任务。每个加载任务是一个独立的事务。
    拆分规则如下：
    - 前端根据期望的并发数`desired_concurrent_number`、Kafka 主题中的分区数以及活动的 BE 节点数计算实际的并发加载任务数。
    - 前端根据计算出的实际并发数拆分作业为加载任务，并将任务安排到任务队列中。

    每个 Kafka 主题由多个分区组成。主题分区和加载任务之间的关系如下：
    - 一个分区唯一分配给一个加载任务，并且加载任务会消费该分区中的所有消息。
    - 一个加载任务可以从一个或多个分区中消费消息。
    - 所有分区在加载任务之间均匀分布。

3. **多个加载任务同时运行以消费多个 Kafka 主题分区中的消息，并将数据加载到 StarRocks 中**

   1. **前端调度并提交加载任务**：前端定期调度队列中的加载任务，并将其分配给选定的协调器 BE 节点。加载任务之间的间隔由配置项 `max_batch_interval` 定义。前端将加载任务均匀地分配给所有 BE 节点。有关 `max_batch_interval` 的详细信息，请参见[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md#example)。

   2. 协调器 BE 开始加载任务，消费分区中的消息，解析和过滤数据。加载任务持续至消费预定义数量的消息或达到预定义的时间限制。消息批处理大小和时间限制在前端配置项 `max_routine_load_batch_size`和`routine_load_task_consume_second`中定义。有关详细信息，请参见[配置](../administration/Configuration.md)。然后，协调器 BE 将消息分发给执行器 BE，并由执行器 BE将消息写入磁盘。

         > **注意**
         >
         > StarRocks 支持通过安全认证机制 SASL_SSL、SASL 或 SSL 连接到 Kafka，或者不使用认证。本主题以不使用认证连接到 Kafka 为例。如果您需要通过安全认证机制连接到 Kafka，请参见[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。

4. **前端生成新的加载任务以持续加载数据。**
   在执行器 BE将数据写入磁盘后，协调器 BE向前端报告加载任务的结果。根据结果，前端随后生成新的加载任务以持续加载数据。或者，前端重试失败的任务以确保加载到 StarRocks 中的数据既不丢失也不重复。

## 创建例行加载作业

以下三个示例描述如何通过创建例行加载作业消费 Kafka 中的 CSV 格式、JSON 格式和 Avro 格式的数据，并将数据加载到 StarRocks。有关详细的语法和参数描述，请参见[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。

### 加载 CSV 格式数据

本部分介绍如何创建一个例行加载作业来消费 Kafka 集群中的 CSV 格式数据，并将数据加载到 StarRocks。

#### 准备数据集

假设 Kafka 集群的主题`ordertest1`中包含一个 CSV 格式的数据集。数据集中的每条消息均包含六个字段：订单 ID、支付日期、客户姓名、国籍、性别和价格。

```Plain
2020050802,2020-05-08,Johann Georg Faust,Deutschland,male,895
2020050802,2020-05-08,Julien Sorel,France,male,893
2020050803,2020-05-08,Dorian Grey,UK,male,1262
2020050901,2020-05-09,Anna Karenina",Russia,female,175
2020051001,2020-05-10,Tess Durbeyfield,US,female,986
2020051101,2020-05-11,Edogawa Conan,japan,male,8924
```

#### 创建表

根据 CSV 格式的数据字段，创建数据库`example_db`中的表`example_tbl1`。以下示例创建一个具有 5 个字段的表，其中不包括 CSV 格式数据中的客户性别字段。

```SQL
CREATE TABLE example_db.example_tbl1 ( 
    `order_id` bigint NOT NULL COMMENT "订单 ID",
    `pay_dt` date NOT NULL COMMENT "支付日期", 
    `customer_name` varchar(26) NULL COMMENT "客户姓名", 
    `nationality` varchar(26) NULL COMMENT "国籍", 
    `price` double NULL COMMENT "价格"
) 
ENGINE=OLAP 
DUPLICATE KEY (order_id, pay_dt) 
DISTRIBUTED BY HASH(`order_id`); 
```

> **注意**
>
> 从 v2.5.7 开始，StarRocks 可自动设置表或分区的桶数（BUCKETS）。无需手动设置桶数。有关详细信息，请参见[确定桶数](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

#### 提交一个例行加载作业

执行以下语句来提交一个名为 `example_tbl1_ordertest1` 的例行加载作业，以消费主题`ordertest1`中的消息，并将数据加载到表`example_tbl1` 中。加载任务将从指定分区的主题中的初始偏移量消费消息。

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
```
```json
{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US","pay_time": 1589191487,"price": 875}
{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK","pay_time": 1589191487,"price": 895}
{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry","country": "France","pay_time": 1589191487,"price": 895}
```

```json
{"commodity_id": "1", "customer_name": "马克吐温", "country": "美国","pay_time": 1589191487,"price": 875}
{"commodity_id": "2", "customer_name": "奥斯卡·王尔德", "country": "英国","pay_time": 1589191487,"price": 895}
{"commodity_id": "3", "customer_name": "安托万·德·圣埃克苏佩里","country": "法国","pay_time": 1589191487,"price": 895}
```

```SQL
CREATE TABLE `example_tbl2` ( 
    `commodity_id` varchar(26) NULL COMMENT "商品编号", 
    `customer_name` varchar(26) NULL COMMENT "客户姓名", 
    `country` varchar(26) NULL COMMENT "国家", 
    `pay_time` bigint(20) NULL COMMENT "支付时间", 
    `pay_dt` date NULL COMMENT "支付日期", 
    `price`double SUM NULL COMMENT "价格"
) 
ENGINE=OLAP
AGGREGATE KEY(`commodity_id`,`customer_name`,`country`,`pay_time`,`pay_dt`) 
DISTRIBUTED BY HASH(`commodity_id`); 
```

```SQL
CREATE ROUTINE LOAD example_db.example_tbl2_ordertest2 ON example_tbl2
COLUMNS(commodity_id, customer_name, country, pay_time, price, pay_dt=from_unixtime(pay_time, '%Y%m%d'))
PROPERTIES
(
    "desired_concurrent_number" = "5",
    "format" = "json",
    "jsonpaths" = "[\"$.commodity_id\",\"$.customer_name\",\"$.country\",\"$.pay_time\",\"$.price\"]"
 )
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest2",
    "kafka_partitions" ="0,1,2,3,4",
    "property.kafka_default_offsets" = "OFFSET_BEGINNING"
);
```
**数据映射:**

  - StarRocks提取JSON格式数据的`name`和`code`键，并将其映射到`jsonpaths`属性中声明的键上。

  - StarRocks提取`jsonpaths`属性中声明的键，并按**顺序**将其映射到`COLUMNS`参数中声明的字段上。

  - StarRocks提取`COLUMNS`参数中声明的字段，并按**名称**将其映射到StarRocks表的列上。

  **数据转换:**

  - 因为示例需要将键`pay_time`转换为DATE数据类型，并将数据加载到StarRocks表中的`pay_dt`列中，所以需要在`COLUMNS`参数中使用from_unixtime函数。其他字段直接映射到表`example_tbl2`的字段上。

  - 由于示例中排除了JSON格式数据的顾客性别列，所以在`COLUMNS`参数中使用`temp_gender`作为此字段的占位符。其他字段直接映射到StarRocks表`example_tbl1`的列上。

    有关数据转换的更多信息，请参见[加载时进行数据转换](./Etl_in_loading.md)。

    > **注意**
    >
    > 如果JSON对象中键的名称和数量与StarRocks表中字段的完全匹配，则无需指定`COLUMNS`参数。

### 加载 Avro 格式数据

自 v3.0.1 起，StarRocks 支持使用例行加载加载 Avro 数据。

#### 准备数据集

##### Avro 模式

1. 创建以下 Avro 模式文件 `avro_schema.avsc`：

      ```JSON
      {
          "type": "record",
          "name": "sensor_log",
          "fields" : [
              {"name": "id", "type": "long"},
              {"name": "name", "type": "string"},
              {"name": "checked", "type" : "boolean"},
              {"name": "data", "type": "double"},
              {"name": "sensor_type", "type": {"type": "enum", "name": "sensor_type_enum", "symbols" : ["TEMPERATURE", "HUMIDITY", "AIR-PRESSURE"]}}  
          ]
      }
      ```

2. 在[模式注册表](https://docs.confluent.io/cloud/current/get-started/schema-registry.html#create-a-schema)中注册Avro模式。

##### Avro 数据

准备Avro数据并将其发送到Kafka主题`topic_0`。

#### 创建表

根据Avro数据的字段，在StarRocks集群中的目标数据库`example_db`中创建表`sensor_log`。表的列名必须与Avro数据中的字段名称匹配。有关表列与Avro数据字段之间的数据类型映射，请参见[数据类型映射](#数据类型映射)。

```SQL
CREATE TABLE example_db.sensor_log ( 
    `id` bigint NOT NULL COMMENT "sensor id",
    `name` varchar(26) NOT NULL COMMENT "sensor name", 
    `checked` boolean NOT NULL COMMENT "checked", 
    `data` double NULL COMMENT "sensor data", 
    `sensor_type` varchar(26) NOT NULL COMMENT "sensor type"
) 
ENGINE=OLAP 
DUPLICATE KEY (id) 
DISTRIBUTED BY HASH(`id`); 
```

> **注意**
>
> 自 v2.5.7 起，StarRocks在创建表或添加分区时可以自动设置桶的数量（BUCKETS）。无需手动设置桶的数量。有关详细信息，请参见[确定桶的数量](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

#### 提交例行加载作业

执行以下语句，将名为`sensor_log_load_job`的例行加载作业提交以消费Kafka主题`topic_0`中的Avro消息，并将数据加载到数据库`sensor`中的表`sensor_log`中。加载作业从主题的指定分区中的初始偏移量消费消息。

```SQL
CREATE ROUTINE LOAD example_db.sensor_log_load_job ON sensor_log  
PROPERTIES  
(  
    "format" = "avro"  
)  
FROM KAFKA  
(  
    "kafka_broker_list" = "<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>,...",
    "confluent.schema.registry.url" = "http://172.xx.xxx.xxx:8081",  
    "kafka_topic" = "topic_0",  
    "kafka_partitions" = "0,1,2,3,4,5",  
    "property.kafka_default_offsets" = "OFFSET_BEGINNING"  
);
```

- 数据格式

  需要在`PROPERTIES`子句中指定`"format = "avro"`，以定义数据格式为Avro。

- 模式注册表

  需要配置`confluent.schema.registry.url`，以指定注册了Avro模式的模式注册表的URL。StarRocks通过使用此URL检索Avro模式。格式如下：

  ```Plaintext
  confluent.schema.registry.url = http[s]://[<schema-registry-api-key>:<schema-registry-api-secret>@]<hostname|ip address>[:<port>]
  ```

- 数据映射和转换

  要指定Avro格式数据与StarRocks表之间的映射和转换关系，需要指定参数`COLUMNS`和属性`jsonpaths`。在`COLUMNS`参数中指定的字段顺序必须与`jsonpaths`属性中字段的顺序相匹配，并且字段的名称必须与StarRocks表的字段名称相匹配。属性`jsonpaths`用于从Avro数据中提取所需的字段，然后由属性`COLUMNS`命名。

  有关数据转换的更多信息，请参见[加载时进行数据转换](https://docs.starrocks.io/en-us/latest/loading/Etl_in_loading)。

  > 注意
  >
  > 如果Avro记录中字段的名称和数量与StarRocks表中列的完全匹配，则无需指定`COLUMNS`参数。

提交加载作业后，您可以执行[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)语句来检查加载作业的状态。

#### 数据类型映射

要加载的Avro数据字段与StarRocks表列之间的数据类型映射如下：

##### 基本类型

| Avro    | StarRocks |
| ------- | --------- |
| nul     | NULL      |
| boolean | BOOLEAN   |
| int     | INT       |
| long    | BIGINT    |
| float   | FLOAT     |
| double  | DOUBLE    |
| bytes   | STRING    |
| string  | STRING    |

##### 复杂类型

| Avro           | StarRocks                                                    |
| -------------- | ------------------------------------------------------------ |
| record         | 将整个RECORD或其子字段作为JSON加载到StarRocks中。               |
| enums          | 字符串                                                         |
| arrays         | ARRAY                                                         |
| maps           | JSON                                                          |
| union(T, null) | NULLABLE(T)                                                   |
| fixed          | 字符串                                                         |

#### 限制

- 当前，StarRocks不支持模式演变。
- 每条Kafka消息必须只包含单个Avro数据记录。

## 检查加载作业和任务

### 检查加载作业

执行[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)语句，以检查加载作业`example_tbl2_ordertest2`的状态。StarRocks返回执行状态`State`、统计信息（包括总消耗行数和总加载行数）`Statistics`，以及加载作业的进度`progress`。

如果加载作业的状态自动更改为**PAUSED**，可能是因为错误行数已超过阈值。有关设置此阈值的详细说明，请参见[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。您可以检查文件`ReasonOfStateChanged`和`ErrorLogUrls`来识别和排除问题。解决问题后，可以执行[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md)语句以恢复**PAUSED**加载作业。

如果加载作业的状态为**CANCELLED**，可能是因为加载作业遇到异常（例如表已被删除）。您可以检查文件`ReasonOfStateChanged`和`ErrorLogUrls`来识别和排除问题。但是，**CANCELLED**加载作业无法恢复。

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


> **注意**
>
> 无法检查已停止或尚未启动的加载作业。

### 检查加载任务

执行[SHOW ROUTINE LOAD TASK](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD_TASK.md)语句，以检查加载作业`example_tbl2_ordertest2`的加载任务，例如当前运行的任务数量、已消耗的Kafka主题分区和消耗进度`DataSourceProperties`，以及相应的协调器 BE 节点`BeId`。

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

您可以执行[PAUSE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/PAUSE_ROUTINE_LOAD.md)语句来暂停一个加载作业。加载作业的状态在执行语句后会变为**PAUSED**。然而，它并没有停止。您可以执行[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md)语句来恢复它。您也可以使用[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)语句来检查它的状态。

以下示例暂停加载作业`example_tbl2_ordertest2`：

```SQL
PAUSE ROUTINE LOAD FOR example_tbl2_ordertest2;
```

## 恢复加载作业

您可以执行[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md)语句来恢复暂停的加载作业。加载作业的状态将暂时变为**NEED_SCHEDULE**（因为加载作业正在重新调度），然后变为**RUNNING**。您可以使用[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)语句来检查它的状态。

以下示例恢复暂停的加载作业`example_tbl2_ordertest2`：

```SQL
RESUME ROUTINE LOAD FOR example_tbl2_ordertest2;
```

## 修改加载作业

在修改加载作业之前，您必须使用[PAUSE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/PAUSE_ROUTINE_LOAD.md)语句将其暂停。然后您可以执行[ALTER ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md)语句。修改后，您可以执行[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md)语句将其恢复，并使用[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)语句检查其状态。

假设活跃的 BE 节点数增加到`6`，要消耗的 Kafka 主题分区是`"0,1,2,3,4,5,6,7"`。如果要增加实际的加载任务并发性，您可以执行以下语句将期望的任务并发数`desired_concurrent_number`增加到`6`（大于或等于活跃的 BE 节点数），并指定 Kafka 主题分区和初始偏移量。

> **注意**
>
> 因为实际任务并发性由多个参数的最小值确定，您必须确保 FE 动态参数`max_routine_load_task_concurrent_num`的值大于或等于`6`。

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

您可以执行[STOP ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/STOP_ROUTINE_LOAD.md)语句来停止加载作业。加载作业的状态在执行语句后会变为**STOPPED**，并且您无法恢复已停止的加载作业。您无法使用[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)语句检查已停止加载作业的状态。

以下示例停止加载作业`example_tbl2_ordertest2`：

```SQL
STOP ROUTINE LOAD FOR example_tbl2_ordertest2;
```

## 常见问题解答

请参阅[例行加载常见问题解答](../faq/loading/Routine_load_faq.md)。