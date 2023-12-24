---
displayed_sidebar: English
---

# 从 Apache Kafka® 持续加载数据

从 '../assets/commonMarkdown/insertPrivNote.md' 导入 InsertPrivNote

本主题介绍如何创建 Routine Load 作业，将 Kafka 消息（事件）流式传输到 StarRocks，并让您熟悉 Routine Load 的一些基本概念。

要持续加载流中的消息到 StarRocks，您可以将消息流存储在 Kafka 主题中，并创建一个 Routine Load 作业来消费这些消息。Routine Load 作业持久存在于 StarRocks 中，生成一系列加载任务以消费主题中全部或部分分区的消息，并将这些消息加载到 StarRocks 中。

Routine Load 作业支持“恰好一次”交付语义，以确保加载到 StarRocks 中的数据既不会丢失也不会重复。

Routine Load 支持数据加载时的数据转换，并支持数据加载过程中 UPSERT 和 DELETE 操作所做的数据更改。有关详细信息，请参阅[加载时转换数据](../loading/Etl_in_loading.md)和[通过加载更改数据](../loading/Load_to_Primary_Key_tables.md)。

<InsertPrivNote />

## 支持的数据格式

Routine Load 现在支持从 Kafka 集群中使用 CSV、JSON 和 Avro（自 v3.0.1 起支持）格式的数据。

> **注意**
>
> 对于 CSV 数据，请注意以下几点：
>
> - 您可以使用长度不超过 50 个字节的 UTF-8 字符串（例如逗号（,）、制表符或竖线（|））作为文本分隔符。
> - 空值使用 `\N` 表示。例如，数据文件由三列组成，该数据文件中的记录在第一列和第三列保存数据，但在第二列中不保存任何数据。在这种情况下，您需要在第二列使用 `\N` 来表示空值。这意味着记录必须编译为 `a,\N,b` 而不是 `a,,b`。`a,,b` 表示记录的第二列包含一个空字符串。

## 基本概念

![routine load](../assets/4.5.2-1.png)

### 术语

- **加载作业**

   Routine Load 作业是一个长时间运行的作业。只要其状态为 RUNNING，加载作业就会持续生成一个或多个并发加载任务，这些任务会消耗 Kafka 集群中主题的消息，并将数据加载到 StarRocks 中。

- **加载任务**

  一个加载作业会根据特定规则拆分为多个加载任务。加载任务是数据加载的基本单元。作为单个事件，加载任务实现了基于[Stream Load](../loading/StreamLoad.md)的加载机制。多个加载任务同时消耗来自主题不同分区的消息，并将数据加载到 StarRocks 中。

### 工作流程

1. **创建 Routine Load 作业。**
   要从 Kafka 加载数据，您需要通过运行[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)语句来创建 Routine Load 作业。前端（FE）解析语句，并根据您指定的属性创建作业。

2. **前端（FE）将作业拆分为多个加载任务。**

    前端根据一定的规则将作业拆分为多个加载任务。每个加载任务都是一个单独的事务。
    拆分规则如下：
    - 前端根据所需的并发数 `desired_concurrent_number`、Kafka 主题中的分区数以及活跃的 BE 节点数来计算加载任务的实际并发数。
    - 根据计算出的实际并发数，前端将作业拆分为加载任务，并将这些任务安排在任务队列中。

    每个 Kafka 主题都由多个分区组成。主题分区和加载任务的关系如下：
    - 一个分区唯一分配给一个加载任务，并且该分区中的所有消息都由加载任务消耗。
    - 一个加载任务可以消耗来自一个或多个分区的消息。
    - 所有分区均匀分布在加载任务之间。

3. **多个加载任务并发运行，消耗多个 Kafka 主题分区的消息，并将数据加载到 StarRocks 中**

   1. **前端（FE）调度并提交加载任务：**前端及时调度队列中的加载任务，并将其分配给选定的协调器 BE 节点。加载任务之间的间隔由配置项 `max_batch_interval` 定义。前端将加载任务均匀分配给所有 BE 节点。有关更多信息，请参见[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md#example)中的 `max_batch_interval`。

   2. 协调器 BE 启动加载任务，消耗分区中的消息，解析和过滤数据。加载任务会持续到消耗了预定义数量的消息或达到了预定义的时间限制。消息批大小和时间限制在前端配置中定义，`max_routine_load_batch_size` 和 `routine_load_task_consume_second`。有关详细信息，请参阅[配置](../administration/BE_configuration.md)。然后，协调器 BE 将消息分发给执行器 BE。执行器 BE 将消息写入磁盘。

         > **注意**
         >
         > StarRocks 支持通过安全认证机制 SASL_SSL、SASL 或 SSL 访问 Kafka，也可以不进行认证。本主题以连接 Kafka 而不进行认证为例。如果需要通过安全认证机制连接到 Kafka，请参见[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。

4. **前端（FE）生成新的加载任务，持续加载数据。**
   执行器 BE 将数据写入磁盘后，协调器 BE 将加载任务的结果报告给前端（FE）。根据结果，前端（FE）会生成新的加载任务以持续加载数据。或者前端（FE）会重试失败的任务，以确保加载到 StarRocks 中的数据既不会丢失也不会重复。

## 创建 Routine Load 作业

以下三个示例描述了如何通过创建 Routine Load 作业来消费 Kafka 中的 CSV 格式、JSON 格式和 Avro 格式的数据，并将数据加载到 StarRocks 中。有关详细语法和参数说明，请参阅[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。

### 加载 CSV 格式的数据

本节描述了如何创建 Routine Load 作业，以消费 Kafka 集群中的 CSV 格式数据，并将数据加载到 StarRocks 中。

#### 准备数据集

假设 Kafka 集群的主题 `ordertest1` 中有一个 CSV 格式的数据集。数据集中的每条消息都包含六个字段：订单 ID、付款日期、客户姓名、国籍、性别和价格。

```Plain
2020050802,2020-05-08,Johann Georg Faust,Deutschland,male,895
2020050802,2020-05-08,Julien Sorel,France,male,893
2020050803,2020-05-08,Dorian Grey,UK,male,1262
2020050901,2020-05-09,Anna Karenina",Russia,female,175
2020051001,2020-05-10,Tess Durbeyfield,US,female,986
2020051101,2020-05-11,Edogawa Conan,japan,male,8924
```

#### 创建表

根据 CSV 格式数据的字段，在数据库 `example_db` 中创建表 `example_tbl1`。以下示例创建了一个表，其中包含 5 个字段，不包括 CSV 格式数据中的客户性别字段。

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
>
> 从 v2.5.7 开始，StarRocks 可以在您创建表或添加分区时自动设置 BUCKET 数量。您不再需要手动设置存储桶数量。有关详细信息，请参阅[确定存储桶数量](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

#### 提交 Routine Load 作业

执行以下语句，提交一个名为 `example_tbl1_ordertest1` 的 Routine Load 作业，以消费主题 `ordertest1` 中的消息，并将数据加载到表 `example_tbl1` 中。加载任务将使用主题指定分区中的初始偏移量的消息。

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

提交加载作业后，您可以执行[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)语句来查看加载作业的状态。

- **加载作业名称**

  一个表上可能存在多个加载作业。因此，我们建议您使用对应的 Kafka 主题和提交加载作业的时间来命名加载作业。这有助于您区分每个表上的加载作业。

- **列分隔符**

  属性 `COLUMN TERMINATED BY` 定义了 CSV 格式数据的列分隔符。默认值为 `\t`。

- **Kafka 主题分区和偏移量**

  您可以指定属性 `kafka_partitions` 和 `kafka_offsets` 来指定消费消息的分区和偏移量。例如，如果您希望加载作业使用主题 `ordertest1` 中的 Kafka 分区 `"0,1,2,3,4"` 的消息，并且所有消息都具有初始偏移量，则可以按如下方式指定属性：如果您希望加载作业使用来自 Kafka 分区 `"0,1,2,3,4"` 的消息，并且需要为每个分区指定单独的起始偏移量，则可以按如下方式进行配置：

    ```SQL
    "kafka_partitions" ="0,1,2,3,4",
    "kafka_offsets" = "OFFSET_BEGINNING, OFFSET_END, 1000, 2000, 3000"
    ```

  您还可以使用属性 `property.kafka_default_offsets` 来设置所有分区的默认偏移量。

    ```SQL
    "kafka_partitions" ="0,1,2,3,4",
    "property.kafka_default_offsets" = "OFFSET_BEGINNING"
    ```

  有关详细信息，请参阅[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。

- **数据映射和转换**


  要指定 CSV 格式数据与 StarRocks 表之间的映射和转换关系，您需要使用 `COLUMNS` 参数。

  **数据映射：**

  - StarRocks 会按顺序提取 CSV 格式数据中的列，并将它们映射到 `COLUMNS` 参数中声明的字段上。

  - StarRocks 会提取 `COLUMNS` 参数中声明的字段，并按名称将其映射到 StarRocks 表的列上。

  **数据转换：**

  由于示例中排除了 CSV 格式数据中的客户性别列，因此需要在 `COLUMNS` 参数中使用字段 `temp_gender` 作为此字段的占位符。其他字段直接映射到 StarRocks 表 `example_tbl1` 的列上。

  有关数据转换的更多信息，请参见[加载时转换数据](./Etl_in_loading.md)。

    > **注意**
    >
    > 如果 CSV 格式数据中的列名、数量和顺序与 StarRocks 表完全对应，则无需指定 `COLUMNS` 参数。

- **任务并发**

  当 Kafka 主题分区较多且 BE 节点足够多时，可以通过增加任务并发来加快加载速度。

  要增加实际加载任务并发，可以在创建例行加载作业时增加所需的加载任务并发 `desired_concurrent_number`。还可以将 FE 的动态配置项 `max_routine_load_task_concurrent_num`（默认最大加载任务并发数）设置为较大的值。有关 `max_routine_load_task_concurrent_num` 的更多信息，请参见[FE 配置项](../administration/FE_configuration.md#fe-configuration-items)。

  实际任务并发取决于活动 BE 节点数、预先指定的 Kafka 主题分区数以及 `desired_concurrent_number` 和 `max_routine_load_task_concurrent_num` 的值中的最小值。

  在该示例中，活动 BE 节点数为 `5`，预先指定的 Kafka 主题分区数为 `5`，`max_routine_load_task_concurrent_num` 的值为 `5`。要增加实际加载任务并发，可以将 `desired_concurrent_number` 从默认值 `3` 增加到 `5`。

  有关属性的详细信息，请参见[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。有关加速加载的详细说明，请参见[例行加载常见问题解答](../faq/loading/Routine_load_faq.md)。

### 加载 JSON 格式数据

本节介绍如何创建例行加载作业以消费 Kafka 集群中的 JSON 格式数据，并将数据加载到 StarRocks 中。

#### 准备数据集

假设 Kafka 集群的主题 `ordertest2` 中有一个 JSON 格式的数据集。数据集包括商品 ID、客户名称、国籍、支付时间和价格等六个关键字段。此外，您还需要将支付时间列转换为 DATE 类型，并将其加载到 StarRocks 表的 `pay_dt` 列中。

```JSON
{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US","pay_time": 1589191487,"price": 875}
{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK","pay_time": 1589191487,"price": 895}
{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry","country": "France","pay_time": 1589191487,"price": 895}
```

> **注意**：每行中的 JSON 对象必须位于一条 Kafka 消息中，否则将返回 JSON 解析错误。

#### 创建表

根据 JSON 格式数据的键，在数据库 `example_db` 中创建名为 `example_tbl2` 的表。

```SQL
CREATE TABLE `example_tbl2` ( 
    `commodity_id` varchar(26) NULL COMMENT "Commodity ID", 
    `customer_name` varchar(26) NULL COMMENT "Customer name", 
    `country` varchar(26) NULL COMMENT "Country", 
    `pay_time` bigint(20) NULL COMMENT "Payment time", 
    `pay_dt` date NULL COMMENT "Payment date", 
    `price` double SUM NULL COMMENT "Price"
) 
ENGINE=OLAP
AGGREGATE KEY(`commodity_id`,`customer_name`,`country`,`pay_time`,`pay_dt`) 
DISTRIBUTED BY HASH(`commodity_id`); 
```

> **注意**：自 v2.5.7 起，StarRocks 可以在创建表或添加分区时自动设置存储桶（BUCKET）的数量。您无需再手动设置存储桶的数量。有关详细信息，请参见[确定存储桶数量](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

#### 提交例行加载作业

执行以下语句，提交一个名为 `example_tbl2_ordertest2` 的例行加载作业，以消费主题 `ordertest2` 中的消息，并将数据加载到表 `example_tbl2` 中。加载任务将从主题指定分区中的初始偏移量处消费消息。

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

  您需要在 `PROPERTIES` 子句中指定 `"format" = "json"`，以定义数据格式为 JSON。

- **数据映射和转换**

  要指定 JSON 格式数据与 StarRocks 表的映射和转换关系，您需要指定 `COLUMNS` 参数和 `jsonpaths` 属性。在 `COLUMNS` 参数中指定的字段顺序必须与 JSON 格式数据的顺序相匹配，字段名称必须与 StarRocks 表的名称相匹配。属性 `jsonpaths` 用于从 JSON 数据中提取所需字段，然后由属性 `COLUMNS` 命名。

  由于示例需要将支付时间字段转换为 DATE 数据类型，并将数据加载到 StarRocks 表的 `pay_dt` 列中，因此需要使用 from_unixtime 函数。其他字段直接映射到表 `example_tbl2` 的字段上。

  **数据映射：**

  - StarRocks 会提取 JSON 格式数据的 `name` 和 `code` 键，并将它们映射到 `jsonpaths` 属性中声明的键上。

  - StarRocks 会提取 `jsonpaths` 属性中声明的键，并**依次**映射到 `COLUMNS` 参数中声明的字段上。

  - StarRocks 会提取 `COLUMNS` 参数中声明的字段，并按名称将它们映射到 StarRocks 表的列上。

  **数据转换**：

  - 由于示例需要将键 `pay_time` 转换为 DATE 数据类型，并将数据加载到 StarRocks 表的 `pay_dt` 列中，因此需要在 `COLUMNS` 参数中使用 from_unixtime 函数。其他字段直接映射到表 `example_tbl2` 的字段上。

  - 由于示例从 JSON 格式数据中排除了客户性别列，因此需要在 `COLUMNS` 参数中使用字段 `temp_gender` 作为此字段的占位符。其他字段直接映射到表 `example_tbl1` 的列上。

    有关数据转换的更多信息，请参见[加载时转换数据](./Etl_in_loading.md)。

    > **注意**
    >
    > 如果 JSON 对象中的键名和键号与 StarRocks 表中的字段完全匹配，则无需指定参数。

### 加载 Avro 格式的数据

自 v3.0.1 起，StarRocks 支持使用例行加载加载 Avro 数据。

#### 准备数据集

##### Avro 架构

1. 创建以下 Avro 架构文件 `avro_schema.avsc`：

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

2. 在[架构注册表](https://docs.confluent.io/cloud/current/get-started/schema-registry.html#create-a-schema)中注册 Avro 架构。

##### Avro 数据

准备 Avro 数据并将其发送到 Kafka 主题 `topic_0`。

#### 创建表

根据 Avro 数据的字段，在 StarRocks 集群的目标数据库 `example_db` 中创建名为 `sensor_log` 的表。表的列名必须与 Avro 数据中的字段名匹配。有关表列和 Avro 数据字段之间的数据类型映射，请参见[数据类型映射](#Data types mapping)。

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
> 自 v2.5.7 起，StarRocks 可以在创建表或添加分区时自动设置存储桶（BUCKET）的数量。您无需再手动设置存储桶的数量。有关详细信息，请参见[确定存储桶数量](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

#### 提交例行加载作业

执行以下语句，提交一个名为 `sensor_log_load_job` 的例行加载作业，以消费 Kafka 主题 `topic_0` 中的 Avro 消息，并将数据加载到数据库 `sensor` 的表 `sensor_log` 中。加载作业将从主题指定分区中的初始偏移量处消费消息。

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

  您需要在 `PROPERTIES` 子句中指定 `"format = "avro"` 以定义数据格式为 Avro。

- 架构注册表

  您需要配置 `confluent.schema.registry.url` 来指定注册 Avro 架构的架构注册表的 URL。StarRocks 通过该 URL 获取 Avro Schema。格式如下：

  ```Plaintext
  confluent.schema.registry.url = http[s]://[<schema-registry-api-key>:<schema-registry-api-secret>@]<hostname|ip address>[:<port>]
  ```

- 数据映射和转换

  若要指定 Avro 格式数据与 StarRocks 表之间的映射和转换关系，您需要指定参数 `COLUMNS` 和属性 `jsonpaths`。在 `COLUMNS` 参数中指定的字段顺序必须与 `jsonpaths` 属性中的字段顺序相匹配，并且字段的名称必须与 StarRocks 表中的字段名称相匹配。`jsonpaths` 属性用于从 Avro 数据中提取所需字段，然后这些字段由 `COLUMNS` 属性命名。

  有关数据转换的更多信息，请参阅[加载时转换数据](https://docs.starrocks.io/en-us/latest/loading/Etl_in_loading)。

  > 注意
  >
  > 如果 Avro 记录中的字段名称和数量与 StarRocks 表中的列完全匹配，则无需指定 `COLUMNS` 参数。

提交加载作业后，您可以执行 [SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) 语句来检查加载作业的状态。

#### 数据类型映射

要加载的 Avro 数据字段与 StarRocks 表列之间的数据类型映射如下：

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
| record         | 将整个 RECORD 或其子字段以 JSON 格式加载到 StarRocks 中。 |
| enums          | 字符串                                                       |
| arrays         | ARRAY                                                        |
| maps           | JSON                                                         |
| union(T, null) | NULLABLE(T)                                                  |
| fixed          | STRING                                                       |

#### 限制

- 目前，StarRocks 不支持模式演变。
- 每条 Kafka 消息只能包含一条 Avro 数据记录。

## 检查加载作业和任务

### 检查加载作业

执行 [SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) 语句来检查加载作业 `example_tbl2_ordertest2` 的状态。StarRocks 返回执行状态 `State`、统计信息（包括消耗的总行数和加载的总行数） `Statistics` 以及加载作业的进度 `progress`。

如果加载作业的状态自动更改为 **PAUSED**，可能是因为错误行数已超过阈值。有关设置此阈值的详细说明，请参阅 [CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。您可以检查文件 `ReasonOfStateChanged` 和 `ErrorLogUrls` 以识别和解决问题。解决问题后，您可以执行 [RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) 语句来恢复 **PAUSED** 加载作业。

如果加载作业的状态为 **CANCELLED**，可能是因为加载作业遇到异常（例如表已被删除）。您可以检查文件 `ReasonOfStateChanged` 和 `ErrorLogUrls` 以识别和解决问题。但是，您无法恢复 **CANCELLED** 加载作业。

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
>
> 您无法检查已停止或尚未启动的加载作业。

### 检查加载任务

执行 [SHOW ROUTINE LOAD TASK](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD_TASK.md) 语句来检查加载作业 `example_tbl2_ordertest2` 的加载任务，例如当前运行的任务数、消耗的 Kafka 主题分区和消耗进度 `DataSourceProperties`，以及相应的 Coordinator BE 节点 `BeId`。

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

您可以执行 [PAUSE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/PAUSE_ROUTINE_LOAD.md) 语句来暂停加载作业。执行该语句后，加载作业的状态将变为 **PAUSED**。但是，它并没有停止。您可以执行 [RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) 语句来恢复它。您还可以使用 [SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) 语句来检查其状态。

以下示例暂停加载作业 `example_tbl2_ordertest2`：

```SQL
PAUSE ROUTINE LOAD FOR example_tbl2_ordertest2;
```

## 恢复加载作业

您可以执行 [RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) 语句来恢复暂停的加载作业。加载作业的状态将暂时变为 **NEED_SCHEDULE**（因为加载作业正在重新计划），然后变为 **RUNNING**。您可以使用 [SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) 语句来检查其状态。

以下示例恢复暂停的加载作业 `example_tbl2_ordertest2`：

```SQL
RESUME ROUTINE LOAD FOR example_tbl2_ordertest2;
```

## 更改加载作业

在更改加载作业之前，您必须使用 [PAUSE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/PAUSE_ROUTINE_LOAD.md) 语句暂停它。然后，您可以执行 [ALTER ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md)。更改后，可以执行 [RESUME](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) ROUTINE LOAD 语句来恢复它，并使用 [SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) 语句检查其状态。

假设活动的 BE 节点数增加到 `6`，并且要消费的 Kafka 主题分区为 `"0,1,2,3,4,5,6,7"`。如果要增加实际的加载任务并发数，可以执行以下语句，将期望的任务并发数 `desired_concurrent_number` 增加到 `6`（大于或等于活动的 BE 节点数），并指定 Kafka 主题分区和初始偏移量。

> **注意**
>
> 由于实际任务并发数由多个参数的最小值决定，因此必须确保 FE 动态参数 `max_routine_load_task_concurrent_num` 的值大于或等于 `6`。

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


您可以执行 [STOP ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/STOP_ROUTINE_LOAD.md) 语句来停止一个加载作业。执行该语句后，加载作业的状态将变为 **STOPPED**，并且无法恢复已停止的加载作业。您不能使用 [SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) 语句来检查已停止的加载作业的状态。

以下示例停止了名为 `example_tbl2_ordertest2` 的加载作业：

```SQL
STOP ROUTINE LOAD FOR example_tbl2_ordertest2;
```

## 常见问题

请参阅 [例程加载常见问题解答](../faq/loading/Routine_load_faq.md)。