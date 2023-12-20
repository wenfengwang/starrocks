---
displayed_sidebar: English
---

# 从 Apache Kafka® 持续加载数据

从 '../assets/commonMarkdown/insertPrivNote.md' 导入 InsertPrivNote

本主题介绍如何创建 Routine Load 任务以实现 Kafka 消息（事件）流式传输至 StarRocks，并使您熟悉一些关于 Routine Load 的基础概念。

为了将流消息持续加载到 StarRocks，您可以将消息流存储在 Kafka 主题中，并创建一个 Routine Load 任务来消费这些消息。Routine Load 任务在 StarRocks 中持久化，生成一系列加载任务来消费主题中全部或部分分区的消息，并将这些消息加载到 StarRocks。

Routine Load 任务支持精确一次交付语义，以确保加载到 StarRocks 的数据既不丢失也不重复。

Routine Load 支持在数据加载时进行数据转换，并且支持在数据加载过程中进行 **UPSERT** 和 **DELETE** 操作所产生的数据变更。更多信息，请参见[Transform data at loading](../loading/Etl_in_loading.md)和[Change data through loading](../loading/Load_to_Primary_Key_tables.md)。

<InsertPrivNote />


## 支持的数据格式

Routine Load 目前支持消费 Kafka 集群中的 CSV、JSON 和 Avro（自 v3.0.1 版本起支持）格式数据。

> **注意**
> 对于 CSV 数据，请注意以下几点：
- 您可以使用长度不超过 50 字节的 UTF-8 字符串作为文本分隔符，例如逗号（,）、制表符或竖线（|）。
- 空值使用 \N 来表示。例如，一个数据文件由三列组成，该数据文件中的一条记录在第一列和第三列中有数据，但第二列中没有数据。在这种情况下，您需要在第二列中使用 \N 来表示空值。这意味着记录应编译为 a,\N,b 而不是 a,,b。a,,b 表示记录的第二列包含一个空字符串。

## 基本概念

![routine load](../assets/4.5.2-1.png)

### 术语

- **加载作业**

  Routine Load 作业是一个长期运行的作业。只要其状态为 RUNNING，加载作业就会持续生成一个或多个并发加载任务，这些任务消费 Kafka 集群中某个主题的消息并将数据加载到 StarRocks。

- **加载任务**

  一个加载作业根据特定规则被分割成多个加载任务。加载任务是数据加载的基本单位。作为一个独立事件，加载任务实现加载机制基于[Stream Load](../loading/StreamLoad.md)。多个加载任务并发消费主题不同分区的消息，并将数据加载到StarRocks。

### 工作流程

1. **创建 Routine Load 作业**。
要从 Kafka 加载数据，您需要通过执行 [CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) 语句来创建 Routine Load 作业。FE 解析该语句，并根据您指定的属性创建作业。

2. **FE**将作业分割为多个加载任务。

   FE 根据特定规则将作业分割为多个加载任务。每个加载任务是一个独立的事务。分割规则如下：
   -  FE 根据期望的并发数 desired_concurrent_number、Kafka 主题中的分区数以及存活的 BE 节点数计算加载任务的实际并发数。
   -  FE 根据计算出的实际并发数将作业分割为加载任务，并将这些任务排入任务队列。

   每个 Kafka 主题包含多个分区。主题分区与加载任务之间的关系如下：
   -  一个分区唯一地分配给一个加载任务，且该分区的所有消息都由该加载任务消费。
   -  一个加载任务可以消费来自一个或多个分区的消息。
   -  所有分区在加载任务之间平均分配。

3. **多个加载任务并发运行，消费多个 Kafka 主题分区的消息，并将数据加载到 StarRocks**

1.    **FE调度和提交加载任务**：FE定时调度队列中的加载任务，并将它们分配给选定的Coordinator BE节点。加载任务之间的间隔由配置项 `max_batch_interval` 定义。FE将加载任务均匀分配到所有BE节点。有关`max_batch_interval`的更多信息，请参见[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md#example)。

2.    Coordinator BE 启动加载任务，消费分区中的消息，解析和过滤数据。加载任务持续到消费完预定义的消息数量或达到预定义的时间限制。消息批次大小和时间限制在 FE 配置 `max_routine_load_batch_size` 和 `routine_load_task_consume_second` 中定义。详细信息，请参见[配置](../administration/BE_configuration.md)。然后，Coordinator BE 将消息分发给 Executor BEs。Executor BEs 将消息写入磁盘。

            > **注意**
            > StarRocks 支持通过安全认证机制 **SASL_SSL**、**SASL** 或 **SSL** 或无认证访问 Kafka。本节以无认证连接 Kafka 为例。如果您需要通过安全认证机制连接 Kafka，请参见 [CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。

4. **FE 生成新的加载任务以持续加载数据。**
Executor BEs 将数据写入磁盘后，Coordinator BE 将加载任务的结果报告给 FE。根据结果，FE 生成新的加载任务以持续加载数据。或者，FE 重试失败的任务，以确保加载到 StarRocks 的数据既不会丢失也不会重复。

## 创建 Routine Load 作业

以下三个示例说明如何消费 CSV 格式、JSON 格式和 Avro 格式数据在 Kafka 中，并通过创建一个 Routine Load 作业将数据加载到 StarRocks。对于详细的语法和参数描述，请参见[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。

### 加载 CSV 格式数据

本节介绍如何创建 Routine Load 作业来消费 Kafka 集群中的 CSV 格式数据，并将数据加载到 StarRocks。

#### 准备数据集

假设 Kafka 集群中的主题 ordertest1 中有一个 CSV 格式的数据集。每条消息包括六个字段：订单 ID、付款日期、客户名称、国籍、性别和价格。

```Plain
2020050802,2020-05-08,Johann Georg Faust,Deutschland,male,895
2020050802,2020-05-08,Julien Sorel,France,male,893
2020050803,2020-05-08,Dorian Grey,UK,male,1262
2020050901,2020-05-09,Anna Karenina",Russia,female,175
2020051001,2020-05-10,Tess Durbeyfield,US,female,986
2020051101,2020-05-11,Edogawa Conan,japan,male,8924
```

#### 创建表

根据 CSV 格式数据的字段，在数据库 example_db 中创建表 example_tbl1。以下示例创建了一个包含 5 个字段的表，不包括 CSV 格式数据中的客户性别字段。

```SQL
CREATE TABLE example_db.example_tbl1 ( 
    `order_id` bigint NOT NULL COMMENT "Order ID",
    `pay_dt` date NOT NULL COMMENT "Payment date", 
    `customer_name` varchar(26) NULL COMMENT "Customer name", 
    `nationality` varchar(26) NULL COMMENT "Nationality", 
    `price`double NULL COMMENT "Price"
) 
ENGINE=OLAP 
DUPLICATE KEY (order_id,pay_dt) 
DISTRIBUTED BY HASH(`order_id`); 
```

> **注意**
> 从 v2.5.7 版本起，StarRocks 在创建表或添加分区时可以自动设置桶（BUCKETS）的数量，无需手动设置。详细信息，请参见[determine the number of buckets](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

#### 提交 Routine Load 作业

执行以下语句来提交一个名为 example_tbl1_ordertest1 的 Routine Load 作业，以消费主题 ordertest1 中的消息，并将数据加载到 example_tbl1 表。加载任务从主题指定分区的初始偏移量开始消费消息。

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

提交加载作业后，您可以执行[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)语句来检查加载作业的状态。

- **加载作业名称**

  一个表可能有多个加载作业。因此，我们建议您根据对应的 Kafka 主题和提交加载作业的时间来命名加载作业，以便区分每个表上的加载作业。

- **列分隔符**

  属性 COLUMN TERMINATED BY 定义了 CSV 格式数据的列分隔符，默认为 \t。

- **Kafka 主题分区和偏移**

  您可以指定 kafka_partitions 和 kafka_offsets 属性来指定消费消息的分区和偏移量。例如，如果您希望加载作业从主题 ordertest1 的 Kafka 分区“0,1,2,3,4”开始消费消息，并且所有分区都使用初始偏移量，您可以如下设置属性：如果您需要从 Kafka 分区“0,1,2,3,4”消费消息，并为每个分区指定不同的起始偏移量，您可以这样配置：

  ```SQL
  "kafka_partitions" ="0,1,2,3,4",
  "kafka_offsets" = "OFFSET_BEGINNING, OFFSET_END, 1000, 2000, 3000"
  ```

  您也可以使用 property.kafka_default_offsets 属性来设置所有分区的默认偏移量。

  ```SQL
  "kafka_partitions" ="0,1,2,3,4",
  "property.kafka_default_offsets" = "OFFSET_BEGINNING"
  ```

  详细信息，请参见 [CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。

- **数据映射和转换**

  要指定 CSV 格式数据与 StarRocks 表之间的映射和转换关系，需要使用 COLUMNS 参数。

  **数据映射**：

-   StarRocks提取CSV格式数据中的列，并**按顺序**映射到`COLUMNS`参数中声明的字段。

-   StarRocks 提取 `COLUMNS` 参数中声明的字段，并按名称**映射**到 StarRocks 表的列。

  **数据转换**：

  由于示例中排除了 CSV 格式数据的客户性别列，因此在 COLUMNS 参数中使用字段 temp_gender 作为该列的占位符。其他字段直接映射到 StarRocks 表 example_tbl1 的列。

  更多关于数据转换的信息，请参见[在加载过程中转换数据](./Etl_in_loading.md)。

    > **注意**
    > 如果 CSV 格式数据中的列名、数量和顺序与 StarRocks 表完全一致，则无需指定 `COLUMNS` 参数。

- **任务并发性**

  当 Kafka 主题分区众多且 BE 节点充足时，可以通过增加任务并发性来加快加载速度。

  要提高实际加载任务并发性，您可以增加所需的任务并发数`desired_concurrent_number`，当您创建常规加载作业时。您也可以将 FE 的动态配置项`max_routine_load_task_concurrent_num`（默认最大加载任务并发数）设置为更大的值。有关`max_routine_load_task_concurrent_num`的更多信息，请参见[FE配置项](../administration/FE_configuration.md#fe-configuration-items)。

  实际任务并发性由存活的 BE 节点数、预先指定的 Kafka 主题分区数以及 desired_concurrent_number 和 max_routine_load_task_concurrent_num 的值中的最小值定义。

  在示例中，存活的 BE 节点数为 5，预先指定的 Kafka 主题分区数为 5，max_routine_load_task_concurrent_num 的值为 5。要提高实际加载任务并发数，您可以将 desired_concurrent_number 从默认值 3 增加到 5。

  更多关于属性的信息，请参见 [CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。有关加快加载速度的详细指南，请参见 [Routine Load FAQ](../faq/loading/Routine_load_faq.md)。

### 加载 JSON 格式数据

本节介绍如何创建一个 Routine Load 作业来从 Kafka 集群中消费 JSON 格式数据，并将数据导入 StarRocks。

#### 准备数据集

假设在 Kafka 集群的主题 ordertest2 中有一个 JSON 格式的数据集。该数据集包含六个键：商品 ID、客户名称、国籍、支付时间和价格。此外，您希望将支付时间列转换为 DATE 类型，并将其导入 StarRocks 表的 pay_dt 列中。

```JSON
{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US","pay_time": 1589191487,"price": 875}
{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK","pay_time": 1589191487,"price": 895}
{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry","country": "France","pay_time": 1589191487,"price": 895}
```

> **注意**：每个JSON对象在一行中必须位于一条Kafka消息中，否则会返回JSON解析错误。

#### 创建表格

根据 JSON 格式数据的键，在数据库 example_db 中创建表 example_tbl2。

```SQL
CREATE TABLE `example_tbl2` ( 
    `commodity_id` varchar(26) NULL COMMENT "Commodity ID", 
    `customer_name` varchar(26) NULL COMMENT "Customer name", 
    `country` varchar(26) NULL COMMENT "Country", 
    `pay_time` bigint(20) NULL COMMENT "Payment time", 
    `pay_dt` date NULL COMMENT "Payment date", 
    `price`double SUM NULL COMMENT "Price"
) 
ENGINE=OLAP
AGGREGATE KEY(`commodity_id`,`customer_name`,`country`,`pay_time`,`pay_dt`) 
DISTRIBUTED BY HASH(`commodity_id`); 
```

> **注意**
> 自 v2.5.7 版本起，StarRocks可以在创建表或添加分区时自动设置桶（BUCKETS）的数量，您无需再手动设置桶数量。更多详细信息，请参考[确定桶的数量](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

#### 提交 Routine Load 作业

执行以下语句，提交名为 example_tbl2_ordertest2 的 Routine Load 作业，以消费主题 ordertest2 中的消息，并将数据加载到表 example_tbl2 中。加载任务将从主题指定分区的初始偏移量开始消费消息。

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

提交加载作业后，您可以执行[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)语句来检查加载作业的状态。

- **数据格式**

  您需要在 PROPERTIES 子句中指定 "format" = "json"，以定义数据格式为 JSON。

- **数据映射和转换**

  为了指定 JSON 格式数据与 StarRocks 表之间的映射和转换关系，您需要指定参数 COLUMNS 和属性 jsonpaths。COLUMNS 参数中指定的字段顺序必须与 JSON 格式数据的顺序相匹配，字段名称必须与 StarRocks 表的列名相匹配。属性 jsonpaths 用于从 JSON 数据中提取所需的字段，然后由属性 COLUMNS 命名。

  因为示例中需要将支付时间字段转换为 DATE 数据类型，并将其加载到 StarRocks 表的 pay_dt 列中，所以需要使用 from_unixtime 函数。其他字段则直接映射到表 example_tbl2 的相应字段中。

  **数据映射**：

-   StarRocks 从 JSON 格式数据中提取名称和代码键，并将它们映射到 jsonpaths 属性中声明的键上。

-   StarRocks提取在`jsonpaths`属性中声明的键，并将它们**按顺序**映射到`COLUMNS`参数中声明的字段上。

-   StarRocks将`COLUMNS`参数中声明的字段按名称映射到StarRocks表的列上。**按名称**

  **数据转换**：

-   由于示例中需要将 pay_time 键转换为 DATE 数据类型，并将其加载到 StarRocks 表的 pay_dt 列中，因此需要在 COLUMNS 参数中使用 from_unixtime 函数。其他字段则直接映射到表 example_tbl2 的相应字段中。

-   由于示例中从 JSON 格式数据中排除了客户性别列，COLUMNS 参数中的 temp_gender 字段被用作该字段的占位符。其他字段则直接映射到 StarRocks 表 example_tbl1 的相应列上。

    更多关于数据转换的信息，请参考[加载时的数据转换](./Etl_in_loading.md)。

        > **注意**
        > 如果 JSON 对象中的键的名称和数量与 StarRocks 表中的字段完全匹配，则无需指定 `COLUMNS` 参数。

### 加载 Avro 格式数据

自 v3.0.1 版本起，StarRocks 支持使用 Routine Load 功能加载 Avro 数据。

#### 准备数据集

##### Avro 架构

1. 创建以下 Avro 架构文件 avro_schema.avsc，并在 Schema Registry 中注册 Avro 架构。

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

2. 注册Avro模式到[Schema Registry](https://docs.confluent.io/cloud/current/get-started/schema-registry.html#create-a-schema)。

##### 准备 Avro 数据并将其发送至 Kafka 主题 topic_0。

创建表格

#### 根据 Avro 数据的字段，在 StarRocks 集群的目标数据库 example_db 中创建表 sensor_log。表的列名必须与 Avro 数据中的字段名相匹配。关于表列和 Avro 数据字段之间的数据类型映射，请参考[数据类型映射](#数据类型映射)。

注意：

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
> 自v2.5.7起，StarRocks在创建表或添加分区时可以自动设置存储桶（BUCKETS）的数量。您不再需要手动设置存储桶的数量。有关详细信息，请参阅[确定存储桶的数量](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

#### 执行以下语句，提交名为 sensor_log_load_job 的 Routine Load 作业，以消费 Kafka 主题 topic_0 中的 Avro 消息，并将数据加载到数据库 sensor 中的表 sensor_log。加载作业将从主题指定分区的初始偏移量开始消费消息。

数据格式

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

- 您需要在 PROPERTIES 子句中指定 "format" = "avro"，以定义数据格式为 Avro。

  Schema Registry

- 您需要配置 confluent.schema.registry.url 来指定注册 Avro 架构的 Schema Registry 的 URL。StarRocks 将使用此 URL 来检索 Avro 架构。格式如下：

  数据映射和转换

  ```Plaintext
  confluent.schema.registry.url = http[s]://[<schema-registry-api-key>:<schema-registry-api-secret>@]<hostname|ip address>[:<port>]
  ```

- 为了指定 Avro 格式数据与 StarRocks 表之间的映射和转换关系，您需要指定参数 COLUMNS 和属性 jsonpaths。COLUMNS 参数中指定的字段顺序必须与属性 jsonpaths 中的字段顺序相匹配，字段名称必须与 StarRocks 表的列名相匹配。属性 jsonpaths 用于从 Avro 数据中提取所需的字段，然后由属性 COLUMNS 命名。

  更多关于数据转换的信息，请参考加载时的数据转换。

  注意：有关数据转换的更多信息，请参阅[加载时数据转换](https://docs.starrocks.io/en-us/latest/loading/Etl_in_loading)。

    > 如果 Avro 记录中的字段名称和数量与 StarRocks 表中的列完全匹配，则无需指定 COLUMNS 参数。
    > 提交加载作业后，您可以执行 SHOW ROUTINE LOAD 语句来检查加载作业的状态。

在提交加载作业后，您可以执行[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)语句来检查加载作业的状态。

#### 您要加载的 Avro 数据字段与 StarRocks 表列之间的数据类型映射如下所示：

基本类型

##### | Avro | StarRocks |

|Avro|StarRocks|
|---|---|
|nul|NULL|
|布尔值|布尔值|
|int|INT|
|长|BIGINT|
|浮动|浮动|
|双|双|
|字节|STRING|
|字符串|字符串|

##### | Avro | StarRocks |

|Avro|StarRocks|
|---|---|
|record|将整个 RECORD 或其子字段作为 JSON 加载到 StarRocks 中。|
|枚举|STRING|
|数组|数组|
|地图|JSON|
|联合（T，空）|NULLABLE（T）|
|固定|STRING|

#### 目前，StarRocks 不支持模式演进。

- 每条 Kafka 消息只能包含一条 Avro 数据记录。
- 检查加载作业和任务

## 检查加载作业

### 执行 SHOW ROUTINE LOAD 语句来检查加载作业 example_tbl2_ordertest2 的状态。StarRocks 会返回执行状态 State、统计信息（包括消耗的总行数和加载的总行数）Statistics 以及加载作业的进度 progress。

如果加载作业的状态自动变为 PAUSED，可能是因为错误行数超过了阈值。有关设置此阈值的详细说明，请参考 [CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)。您可以检查 `ReasonOfStateChanged` 和 `ErrorLogUrls` 文件来识别并解决问题。解决问题后，您可以执行 `RESUME ROUTINE LOAD` 语句来恢复 PAUSED 状态的加载作业。

如果加载作业的状态为 **CANCELLED**，则可能是因为错误行数超过阈值。有关设置此阈值的详细说明，请参阅[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。您可以检查文件`ReasonOfStateChanged`和`ErrorLogUrls`以识别和解决问题。解决问题后，您可以执行[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md)语句来恢复**CANCELLED**加载作业。

如果加载作业的状态为**CANCELLED**，可能是因为加载作业遇到异常（例如表已被删除）。您可以检查文件`ReasonOfStateChanged`和`ErrorLogUrls`以识别和解决问题。但是，您无法恢复**CANCELLED**的加载作业。

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

> **您无法检查已停止或尚未开始的加载作业**。
> 你不能检查一个已经停止或尚未启动的加载任务。

### 执行 SHOW ROUTINE LOAD TASK 语句来检查加载作业 example_tbl2_ordertest2 的加载任务，例如当前正在运行的任务数量、已消费的 Kafka 主题分区及其消费进度 DataSourceProperties，以及对应的 Coordinator BE 节点 BeId。

执行[SHOW ROUTINE LOAD TASK](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD_TASK.md)语句，以检查加载作业`example_tbl2_ordertest2`的加载任务，例如当前运行的任务数量，已消耗的Kafka主题分区和消费进度`DataSourceProperties`，以及相应的协调器BE节点`BeId`。

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

## 您可以执行 PAUSE ROUTINE LOAD 语句来暂停加载作业。执行该语句后，加载作业的状态将变为 PAUSED，但作业并未停止。您可以执行 RESUME ROUTINE LOAD 语句来恢复作业。您还可以使用 SHOW ROUTINE LOAD 语句来检查其状态。

You can execute the [PAUSE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/PAUSE_ROUTINE_LOAD.md) statement to pause a load job. The state of the load job will be **PAUSED** after the statement is executed. However, it has not stopped. You can execute the [RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) statement to resume it. You can also check its status with the [SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) statement.

恢复加载作业

```SQL
PAUSE ROUTINE LOAD FOR example_tbl2_ordertest2;
```

## 您可以执行 RESUME ROUTINE LOAD 语句来恢复暂停的加载作业。加载作业的状态将暂时变为 NEED_SCHEDULE（因为作业正在重新调度），然后变为 RUNNING。您可以使用 SHOW ROUTINE LOAD 语句来检查其状态。

You can execute the [RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) statement to resume a paused load job. The state of the load job will be **NEED_SCHEDULE** temporarily (because the load job is being re-scheduled), and then become **RUNNING**. You can check its status with the [SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) statement.

更改加载作业

```SQL
RESUME ROUTINE LOAD FOR example_tbl2_ordertest2;
```

## 在更改加载作业之前，您必须先使用 PAUSE ROUTINE LOAD 语句来暂停它。然后，您可以执行 ALTER ROUTINE LOAD。更改后，您可以执行 RESUME ROUTINE LOAD 语句来恢复作业，并使用 SHOW ROUTINE LOAD 语句来检查其状态。

Before altering a load job, you must pause it with the [PAUSE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/PAUSE_ROUTINE_LOAD.md) statement. Then you can execute the [ALTER ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md). After altering it, you can execute the [RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) statement to resume it, and check its status with the [SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) statement.

注意：

> **由于实际任务并发数是由多个参数中的最小值决定的，因此您必须确保 FE 动态参数 max_routine_load_task_concurrent_num 的值大于或等于 6。**
> 因为实际任务并发取决于多个参数的最小值，您必须确保FE动态参数`max_routine_load_task_concurrent_num`的值大于或等于`6`。

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

## 您可以执行 STOP ROUTINE LOAD 语句来停止加载作业。执行该语句后，加载作业的状态将变为 STOPPED，并且您无法恢复已停止的加载作业。您无法使用 SHOW ROUTINE LOAD 语句来检查已停止加载作业的状态。

你可以执行[STOP ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/STOP_ROUTINE_LOAD.md)语句来停止一个加载作业。在语句执行后，加载作业的状态将变为**STOPPED**，并且你无法恢复已停止的加载作业。你无法使用[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)语句来检查已停止的加载作业的状态。

常见问题解答

```SQL
STOP ROUTINE LOAD FOR example_tbl2_ordertest2;
```

## 请参考 Routine Load 常见问题解答。

请参阅[Routine Load FAQ](../faq/loading/Routine_load_faq.md)。
