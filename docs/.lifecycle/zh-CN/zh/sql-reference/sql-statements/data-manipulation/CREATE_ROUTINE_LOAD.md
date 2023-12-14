---
displayed_sidebar: "Chinese"
---

# 创建例行加载

## 功能

例行加载支持持续消费 Apache Kafka® 的消息并导入至 StarRocks 中。例行加载支持 Kafka 中消息的格式为 CSV、JSON、Avro (自 v3.0.1)，并且访问 Kafka 时，支持多种安全协议，包括 `plaintext`、`ssl`、`sasl_plaintext` 和 `sasl_ssl`。

本文介绍创建例行加载的语法、参数说明和示例。

> **说明**
>
> - 例行加载的应用场景、基本原理和基本操作，请参见 [从 Apache Kafka® 持续导入](../../../loading/RoutineLoad.md)。
> - 例行加载操作需要目标表的 INSERT 权限。如果您的用户账号没有 INSERT 权限，请参考 [GRANT](../account-management/GRANT.md) 给用户赋权。

## 语法

```SQL
CREATE ROUTINE LOAD <database_name>.<job_name> ON <table_name>
[load_properties]
[job_properties]
FROM data_source
[data_source_properties]
```

## 参数说明

### `database_name`、`job_name`、`table_name`

`database_name`

选填，目标数据库的名称。

`job_name`

必填，导入作业的名称。一张表可能有多个导入作业，建议您利用具有辨识度的信息（例如 Kafka Topic 名称、创建导入作业的大致时间等）来设置具有意义的导入作业名称，用于区分多个导入作业。同一数据库内，导入作业的名称必须唯一。

`table_name`

必填，目标表的名称。

### `load_properties`

选填。源数据的属性。语法：

```SQL
[COLUMNS TERMINATED BY '<column_separator>'],
[ROWS TERMINATED BY '<row_separator>'],
[COLUMNS (<column1_name>[,<column2_name>,<column_assignment>,... ])],
[WHERE <expr>],
[PARTITION (<partition1_name>[,<partition2_name>,...])]
[TEMPORARY PARTITION (<temporary_partition1_name>[,<temporary_partition2_name>,...])]
```

如果导入 CSV 格式的数据，则可以指定列分隔符，默认为`\t`，即 Tab。例如可以输入 `COLUMNS TERMINATED BY ","`。指定列分隔符为逗号(,)。

> **说明**
>
> - 必须确保这里指定的列分隔符与源数据中的列分隔符一致。
> - StarRocks 支持设置长度最大不超过 50 个字节的 UTF-8 编码字符串作为列分隔符，包括常见的逗号 (,)、Tab 和 Pipe (|)。
> - 空值 (null) 用 `\N` 表示。比如，源数据一共有三列，其中某行数据的第一列、第三列数据分别为 `a` 和 `b`，第二列没有数据，则第二列需要用 `\N` 来表示空值，写作 `a,\N,b`，而不是 `a,,b`。`a,,b` 表示第二列是一个空字符串。

`ROWS TERMINATED BY`

用于指定源数据中的行分隔符。如果不指定该参数，则默认为 `\n`。

`COLUMNS`

源数据和目标表之间的列映射和转换关系。详细说明，请参见[列映射和转换关系](#列映射和转换关系)。

- `column_name`：映射列，源数据中这类列的值可以直接落入目标表的列中，不需要进行计算。
- `column_assignment`：衍生列，格式为 `column_name = expr`，源数据中这类列的值需要基于表达式 `expr` 进行计算后，才能落入目标表的列中。 建议将衍生列排在映射列之后，因为 StarRocks 先解析映射列，再解析衍生列。

> **说明**
>
> 以下情况不需要设置 `COLUMNS` 参数：
>
> - 待导入 CSV 数据中的列与目标表中列的数量和顺序一致。
> - 待导入 JSON 数据中的 Key 名与目标表中的列名一致。

`WHERE`

设置过滤条件，只有满足过滤条件的数据才会导入到 StarRocks 中。例如只希望导入 `col1` 大于 `100` 并且 `col2` 等于 `1000` 的数据行，则可以输入 `WHERE col1 > 100 and col2 = 1000`。

> **说明**
>
> 过滤条件中指定的列可以是源数据中本来就存在的列，也可以是基于源数据的列生成的衍生列。

`PARTITION`

将数据导入至目标表的指定分区中。如果不指定分区，则会将数据自动导入至其对应的分区中。 示例：

```SQL
PARTITION(p1, p2, p3)
```

### `job_properties`

必填。导入作业的属性。语法：

```SQL
PROPERTIES ("<key1>" = "<value1>"[, "<key2>" = "<value2>" ...])
```

参数说明如下：

| **参数**                  | **是否必选** | **说明**                                                     |
| ------------------------- | ------------ | ------------------------------------------------------------ |
| desired_concurrent_number | 否           | 单个例行加载导入作业的**期望**任务并发度，表示期望一个导入作业最多被分成多少个任务并行执行。默认值为 `3`。 但是**实际**任务并行度由如下多个参数组成的公式决定，并且实际任务并行度的上限为 BE 节点的数量或者消费分区的数量。`min(alive_be_number, partition_number, desired_concurrent_number, max_routine_load_task_concurrent_num)`。<ul> <li>`alive_be_number`：存活的 BE 节点数量。</li><li>`partition_number`：消费分区数量。</li><li>`desired_concurrent_number`：单个例行加载导入作业的期望任务并发度。默认值为 `3`。</li><li>`max_routine_load_task_concurrent_num`：例行加载导入作业的默认最大任务并行度，默认值为 `5`。该参数为 [FE 动态参数](../../../administration/Configuration.md)。</li></ul> |
| max_batch_interval        | 否           | 任务的调度间隔，即任务多久执行一次。单位：秒。取值范围：`5`～`60`。默认值：`10`。建议取值为导入间隔 10s 以上，否则会因为导入频率过高可能会报错版本数过多。 |
| max_batch_rows            | 否           | 该参数只用于定义错误检测窗口范围，错误检测窗口范围为单个例行加载导入任务所消费的 `10 * max-batch-rows` 行数据，默认为 `10 * 200000 = 2000000`。例行加载任务时会检测窗口中数据是否存在错误。错误数据是指 StarRocks 无法解析的数据，比如非法的 JSON。 |
| max_error_number          | 否           | 错误检测窗口范围内允许的错误数据行数的上限。当错误数据行数超过该值时，导入作业会暂停，此时您需要执行 [SHOW ROUTINE LOAD](../data-manipulation/SHOW_ROUTINE_LOAD.md)，根据 `ErrorLogUrls`，检查 Kafka 中的消息并且更正错误。默认为 `0`，表示不允许有错误行。错误行不包括通过 WHERE 子句过滤掉的数据。 |
| strict_mode               | 否           | Whether to enable strict mode. The value range is: `TRUE` or `FALSE`. Default value: `FALSE`. When enabled, if the value of a column in the source data is `NULL`, but the target table does not allow `NULL` for that column, the row data will be filtered out.<br />For an introduction to this mode, see [Strict Mode](../../../loading/load_concept/strict_mode.md). |
| log_rejected_record_num | 否 | Specifies the maximum number of data rows that are allowed to be filtered out due to poor data quality. This parameter is supported since version 3.1. The value range is: `0`, `-1`, a positive integer greater than 0. Default value: `0`. <ul><li>A value of `0` indicates that filtered out data rows are not recorded.</li><li>A value of `-1` indicates that all filtered out data rows are recorded.</li><li>A positive integer greater than 0 (such as `n`) indicates that up to `n` filtered out data rows can be recorded on each BE node.</li></ul> |
| timezone                  | 否           | The value of this parameter will affect the results returned by all functions related to time zone settings involved in imports. The functions affected by the time zone include strftime, alignment_timestamp, and from_unixtime, for more information, please see [Set Timezone](../../../administration/timezone.md). The time zone settings set by the import parameter `timezone` correspond to the session-level time zone described in [Set Timezone](../../../administration/timezone.md). |
| merge_condition           | 否           | Used to specify the column name that acts as the update effective condition. In this way, the update will only take effect when the value of this column in the imported data is greater than or equal to the current value. See [Implementing Data Changes through Import](../../../loading/Load_to_Primary_Key_tables.md) for details. The specified column must be a non-primary key column, and conditional updates are only supported for primary key model tables. |
| format                    | 否           | The format of the source data, with a value range of: `CSV`, `JSON`, or `Avro` (since v3.0.1). Default value: `CSV`.|
| trim_space                | 否           | Specifies whether to remove leading and trailing spaces before the column delimiter in the CSV file. Value type: BOOLEAN. Default value: `false`.<br />Some databases add some spaces before and after the column delimiter when exporting data to a CSV file. These spaces, depending on their position, can be called "leading spaces" or "trailing spaces". By setting this parameter, StarRocks can remove these unnecessary spaces when importing data.<br />It should be noted that StarRocks will not remove spaces within fields enclosed by the specified `enclose` character (including leading and trailing spaces in the field). For example, if the column delimiter is a vertical line (<code class="language-text">&#124;</code>) and the `enclose` character is a double quotation mark (`"`): <code class="language-text">&#124; "Love StarRocks" &#124;</code>. If `trim_space` is set to true, the result data processed by StarRocks is <code class="language-text">&#124;"Love StarRocks"&#124;</code>.|
| enclose                   | 否           | According to [RFC4180](https://www.rfc-editor.org/rfc/rfc4180), it is used to specify the character enclosing the fields in the CSV file. Value type: single-byte character. Default value: `NONE`. The most commonly used `enclose` characters are single quotation mark (`'`) or double quotation mark (`"`).<br />All special characters (including line delimiters, column delimiters, etc.) within the field enclosed by the `enclose` specified character are treated as normal symbols, which go a step further than the RFC4180 standard. The `enclose` property provided by StarRocks supports setting any single-byte character.<br />If a field contains the specified `enclose` character, the same character can be used to escape the `enclose` character. For example, when `enclose` is set to double quotation mark (`"`), the field value `a "quoted" c` should be written as `"a ""quoted"" c"` in the CSV file. |
| escape                    | 否           | Specifies the escape character. It is used to escape various special characters, such as line delimiters, column delimiters, escape characters, `enclose` specified characters, etc., so that StarRocks parses these special characters as part of the field value. Value type: single-byte character. Default value: `NONE`. The most commonly used `escape` character is the slash (`\`), which should be written as double slashes (`\\`) in SQL statements.<br />The `escape` specified character works both inside and outside the `enclose` specified character.<br />Here are two examples:<ul><li>When `enclose` is set to double quotation mark (`"`) and `escape` is set to slash (`\`), StarRocks will parse `"say \"Hello world\""` as a field value of `say "Hello world"`.</li><li>Assuming the column delimiter is a comma (`,`), when `escape` is set to slash (`\`), StarRocks will parse `a, b\, c` as two field values of `a` and `b, c`.</li></ul> |
| strip_outer_array         | 否           | Whether to trim the outermost array structure of JSON data. Value range: `TRUE` or `FALSE`. Default value: `FALSE`. In real business scenarios, the JSON data to be imported may have a pair of square brackets `[]` representing the array structure on the outermost level. In this case, it is generally recommended to specify the parameter value as `true`, so that StarRocks will trim the outer square brackets `[]`, and import each inner array as a separate data row. If you specify the parameter value as `false`, StarRocks will parse the entire JSON data as an array and import it as a single data row. For example, if the JSON data to be imported is `[{"category" : 1, "author" : 2}, {"category" : 3, "author" : 4}]`, if you specify the parameter as `true`, StarRocks will parse `{"category" : 1, "author" : 2}` and `{"category" : 3, "author" : 4}` as two data rows and import them into the corresponding data rows of the target table. |
| jsonpaths                 | 否           | Used to specify the names of the fields to be imported. This parameter is only required when importing JSON data using a matching pattern. The parameter value is in JSON format. See [The target table has derived columns, and the column values are calculated by expressions](#The target table has derived columns, and the column values are calculated by expressions) for details.|
| json_root                 | 否           | If there is no need to import the entire JSON data, specify the actual root node of the JSON data to be imported. The parameter value is a valid JsonPath. The default value is empty, indicating that the entire JSON data will be imported. For specific examples, please refer to the examples provided in this document [Specify the actual root node of the JSON data to be imported](#Specify the actual root node of the JSON data to be imported). |
| task_consume_second                 | 是           | 单个 Routine Load 导入作业中每个 Routine Load 导入任务消费数据的最大时长，单位为秒。相较于[FE 动态参数](../../../administration/Configuration.md) `routine_load_task_consume_second`（作用于集群内部所有 Routine Load 导入作业），该参数仅针对单个 Routine Load 导入作业，更加灵活。该参数自 v3.1.0 起新增。 <ul><li>当未配置 `task_consume_second` 和 `task_timeout_second` 时，StarRocks 则使用 FE 动态参数 `routine_load_task_consume_second` 和`routine_load_task_timeout_second` 来控制导入行为。</li><li>当只配置 `task_consume_second` 时，默认 `task_timeout_second` = `task_consume_second` * 4。</li><li>当只配置 `task_timeout_second` 时，默认 `task_consume_second` = `task_timeout_second` /4。</li></ul>|
| task_timeout_second                 | 是           | Routine Load 导入作业中每个 Routine Load 导入任务超时时间，单位为秒。相较于[FE 动态参数](../../../administration/Configuration.md) `routine_load_task_timeout_second`（作用于集群内部所有 Routine Load 导入作业），该参数仅针对单个 Routine Load 导入作业，更加灵活。该参数自 v3.1.0 起新增。<ul><li>当未配置 `task_consume_second` 和 `task_timeout_second` 时，StarRocks 则使用 FE 动态参数 `routine_load_task_consume_second` 和 `routine_load_task_timeout_second` 来控制导入行为。</li><li>当只配置 `task_timeout_second` 时，默认 `task_consume_second` = `task_timeout_second` /4。</li><li>当只配置 `task_consume_second` 时，默认 `task_timeout_second` = `task_consume_second` * 4。</li></ul>|

### `data_source`、`data_source_properties`

数据源和数据源属性。语法:

```Bash
FROM <data_source>
 ("<key1>" = "<value1>"[, "<key2>" = "<value2>" ...])
```

`data_source`

必填。指定数据源，目前仅支持取值为 `KAFKA`。

`data_source_properties`

必填。数据源属性，参数以及说明如下：

| **参数**          | **说明**                                                     |
| ----------------- | ------------------------------------------------------------ |
| kafka_broker_list | Kafka 的 Broker 连接信息。格式为 `<kafka_broker_ip>:<kafka port>`，多个 Broker 之间以英文逗号 (,) 分隔。 Kafka Broker 默认端口号为 `9092`。示例：`"kafka_broker_list" = "xxx.xx.xxx.xx:9092,xxx.xx.xxx.xx:9092"` |
| kafka_topic       | Kafka Topic 名称。一个导入作业仅支持消费一个 Topic 的消息。  |
| kafka_partitions  | 待消费的分区。示例：`"kafka_partitions" = "0, 1, 2, 3"`。如果不配置该参数，则默认消费所有分区。 |
| kafka_offsets     | 待消费分区的起始消费位点，必须一一对应 `kafka_partitions` 中指定的每个分区。如果不配置该参数，则默认为从分区的末尾开始消费。支持取值为<ul><li> 具体消费位点：从分区中该消费位点的数据开始消费。</li><li>`OFFSET_BEGINNING`：从分区中有数据的位置开始消费。</li><li>`OFFSET_END`：从分区的末尾开始消费。</li></ul>多个起始消费位点之间用英文逗号（, ）分隔。<br />示例： `"kafka_offsets" = "1000, OFFSET_BEGINNING, OFFSET_END, 2000"`。|
| property.kafka_default_offsets | 所有待消费分区的默认起始消费位点。支持的取值与 `kafka_offsets` 一致。 |
| confluent.schema.registry.url| 注册该 Avro schema 的 Schema Registry 的 URL，StarRocks 会从该 URL 获取 Avro schema。格式为 `confluent.schema.registry.url = http[s]://[<schema-registry-api-key>:<schema-registry-api-secret>@]<hostname or ip address>[:<port>]`。|

**更多数据源相关参数**

支持设置更多数据源 Kafka 相关参数，功能等同于 Kafka 命令行 `--property`， 支持参数，请参见 [librdkafka 配置项文档](https://github.com/edenhill/librdkafka/blob/master/CONFIGURATION.md)中适用于客户端的配置项。

> **说明**
>
> 当参数的取值是文件时，则值前加上关键词 `FILE:`。关于如何创建文件，请参见 [CREATE FILE](../Administration/CREATE_FILE.md) 命令文档。

**指定所有待消费分区的默认起始消费位点。**

```Plaintext
"property.kafka_default_offsets" = "OFFSET_BEGINNING"
```

`property.kafka_default_offsets` 的取值为具体的消费位点，或者：

- `OFFSET_BEGINNING`：从分区中有数据的位置开始消费。
- `OFFSET_END`：从分区的末尾开始消费。

**指定导入任务消费 Kafka 时所基于 Consumer Group 的 `group.id`**

```Plaintext
"property.group.id" = "group_id_0"
```

如果没有指定 `group.id`，StarRocks 会根据 Routine Load 的导入作业名称生成一个随机值，具体格式为`{job_name}_{random uuid}`，如 `simple_job_0a64fe25-3983-44b2-a4d8-f52d3af4c3e8`。

**指定 BE 访问 Kafka 时的安全协议并配置相关参数**

支持安全协议为 `plaintext`（默认）、`ssl`、`sasl_plaintext` 和 `sasl_ssl`，并且需要根据安全协议配置相关参数。

当安全协议为 `sasl_plaintext` 或 `sasl_ssl` 时，支持如下 SASL 认证机制：

- PLAIN
- SCRAM-SHA-256 和 SCRAM-SHA-512
- OAUTHBEARER

- **访问 Kafka 时，使用安全协议 SSL**

```sql
"property.security.protocol" = "ssl", -- 指定安全协议为 SSL
"property.ssl.ca.location" = "FILE:ca-cert", -- CA 证书的位置
--如果 Kafka server 端开启了 client 认证，则还需设置如下三个参数：
"property.ssl.certificate.location" = "FILE:client.pem", -- Client 的 public key 的位置
"property.ssl.key.location" = "FILE:client.key", -- Client 的 private key 的位置
"property.ssl.key.password" = "abcdefg" -- Client 的 private key 的密码
```

- **访问 Kafka 时，使用 SASL_PLAINTEXT 安全协议和 SASL/PLAIN 认证机制

```sql
"property.security.protocol"="SASL_PLAINTEXT", -- 指定安全协议为 SASL_PLAINTEXT
"property.sasl.mechanism"="PLAIN", -- 指定 SASL 认证机制为 PLAIN
"property.sasl.username"="admin", -- SASL 的用户名
"property.sasl.password"="admin" -- SASL 的密码
```

### FE 和 BE 配置项

Routine Load 相关配置项，请参见[配置参数](../../../administration/Configuration.md)。

## 列映射和转换关系

### 导入 CSV 数据

**如果 CSV 格式的数据中的列与目标表中的列的数量或顺序不一致**，则需要通过 `COLUMNS` 参数来指定源数据和目标表之间的列映射和转换关系。一般包括如下两种场景：

- **The number of columns in the source data is the same as that in the target table, but the order is different**. And the data does not need to be calculated through functions, and can be directly imported into the corresponding columns in the target table.
  You need to configure the column mapping and transformation relationship in the `COLUMNS` parameter according to the order of the columns in the source data and using the corresponding column names in the target table.

  For example, there are three columns in the target table, named `col1`, `col2`, and `col3` in order; in the source data, there are also three columns, corresponding to the `col3`, `col2`, and `col1` in the target table. In this case, you need to specify `COLUMNS(col3, col2, col1)`.
- **The number of columns in the source data is different from that in the target table, and even some of the data in the columns needs to be transformed (calculated by functions) to be imported into the corresponding columns in the target table**.
  You not only need to configure the column mapping relationship in the `COLUMNS` parameter according to the order of the columns in the source data and the corresponding column names in the target table, but also need to specify the functions involved in data calculation. Here are two examples:

  - **More columns in the source data than in the target table**.
    For example, there are three columns in the target table, named `col1`, `col2`, and `col3` in order; in the source data, there are four columns, the first three correspond to the `col1`, `col2`, and `col3` in the target table, and the fourth column does not have a corresponding column in the target table. In this case, you need to specify `COLUMNS(col1, col2, col3, temp)`, where the last column can be arbitrarily named (such as `temp`) for placeholder.
  - **Derived columns generated by calculating based on the columns in the source data for the target table**.
    For example, there is only one column containing time data in the source data, formatted as `yyyy-mm-dd hh:mm:ss`. There are three columns in the target table, named `year`, `month`, and `day`, all of which are derived columns generated by calculating based on the column containing time data in the source data. In this case, you can specify `COLUMNS(col, year = year(col), month=month(col), day=day(col))`. Where `col` is the temporary name of the column contained in the source data, and `year = year(col)`, `month=month(col)`, and `day=day(col)` are used to specify to extract the corresponding data from the `col` column in the source data and import it into the corresponding derived columns in the target table, such as `year = year(col)` indicating to extract the `yyyy` part of the data in the `col` column in the source data using the `year` function and import it into the `year` column in the target table.

  For operation examples, please refer to [Setting column mapping and transformation relationship](#setting-column-mapping-and-transformation-relationship).

### Import JSON or Avro Data

> **Note**
>
> Since version 3.0.1, StarRocks supports importing Avro data using Routine Load. When importing JSON or Avro data, the configuration of the column mapping and transformation relationship is the same. Therefore, this section uses importing Avro data as an example to explain.

**If the key name in the JSON formatted data does not match the column names in the target table**, you need to use matching mode to import JSON data, that is, use the `jsonpaths` and `COLUMNS` parameters to specify the column mapping and transformation relationship between the source data and the target table:

- The `jsonpaths` parameter specifies the key of the JSON data to be imported, and sorts it (just like generating CSV data).
- The `COLUMNS` parameter specifies the mapping relationship and data transformation relationship between the keys of the JSON data to be imported and the columns of the target table.
  - Corresponds one-to-one in order with the keys specified in `jsonpaths`.
  - Corresponds one-to-one in name with the columns in the target table.

For detailed examples, please refer to [The target table contains derived columns, and their column values are generated by expression calculation](#The-target-table-contains-derived-columns,-and-their-column-values-are-generated-by-expression-calculation).

> **Note**
>
> If the key names in the JSON data to be imported can match the column names in the target table (the order and quantity of the keys do not need to correspond), you can use the simple mode to import JSON data without configuring `jsonpaths` and `COLUMNS`.

## Example

### Import CSV Data

This section takes CSV formatted data as an example, focusing on how to use various parameter configurations when creating import jobs to meet various import requirements in different business scenarios.

**Data set**

Suppose the Topic `ordertest1` in the Kafka cluster contains CSV formatted data as follows, where the columns in the CSV data mean the order number, payment date, customer name, nationality, gender, and payment amount respectively.

```Plaintext
2020050802,2020-05-08,Johann Georg Faust,Deutschland,male,895
2020050802,2020-05-08,Julien Sorel,France,male,893
2020050803,2020-05-08,Dorian Grey,UK,male,1262
2020050901,2020-05-09,Anna Karenina,Russia,female,175
2020051001,2020-05-10,Tess Durbeyfield,US,female,986
2020051101,2020-05-11,Edogawa Conan,japan,male,8924
```

**Target database and table**

Based on the columns that need to be imported from the CSV data, create a table `example_tbl1` in the target database `example_db` in the StarRocks cluster. The table creation statement is as follows:

```SQL
CREATE TABLE example_db.example_tbl1 ( 
    `order_id` bigint NOT NULL COMMENT "Order number",
    `pay_dt` date NOT NULL COMMENT "Payment date", 
    `customer_name` varchar(26) NULL COMMENT "Customer name", 
    `nationality` varchar(26) NULL COMMENT "Nationality", 
    `gender` varchar(26) NULL COMMENT "Gender", 
    `price` double NULL COMMENT "Payment amount") 
DUPLICATE KEY (order_id,pay_dt) 
DISTRIBUTED BY HASH(`order_id`);
```

#### Consume from the specified partitions and starting positions in the Topic

If it is necessary to specify the partitions and the starting positions for each partition, you need to configure the `kafka_partitions` and `kafka_offsets` parameters.

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1",
    "kafka_partitions" ="0,1,2,3", -- Specify partitions
    "kafka_offsets" = "1000, OFFSET_BEGINNING, OFFSET_END, 2000" -- Specify starting positions
);
```

#### Adjusting import performance

If it is necessary to improve the import performance and avoid consumption backlogs, you can set the `desired_concurrent_number` of the single Routine Load import job to increase the actual task parallelism, splitting a single import job into as many parallel import tasks as possible.

> For more ways to improve import performance, see [Routine Load FAQ](../../../faq/loading/Routine_load_faq.md).

Please note that the actual task parallelism is determined by the formula composed of multiple parameters and has a maximum limit of the number of BE nodes or the number of consuming partitions.

```SQL
min(alive_be_number, partition_number, desired_concurrent_number, max_routine_load_task_concurrent_num)
```

因此当消费分区和 BE 节点数量较多，并且大于其余两个参数时，如果您需要增加实际任务并行度，则可以提高`max_routine_load_task_concurrent_num`、 `desired_concurrent_number` 的值。

假设消费分区数量为 `7`，存活 BE 数量为 `5`，`max_routine_load_task_concurrent_num` 为默认值 `5`。此时如果需要增加实际任务并发度至上限，则需要将 `desired_concurrent_number` 设置为 `5`（默认值为 `3`），则计算实际任务并行度 `min(5,7,5,5)` 为 `5`。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
"desired_concurrent_number" = "5" -- 设置单个 Routine Load 导入作业的期望任务并发度
)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### 设置列的映射和转换关系

如果 CSV 格式的数据中的列与目标表中的列的数量或顺序不一致，假设无需导入 CSV 数据的第五列至目标表，则需要通过 `COLUMNS` 参数来指定源数据和目标表之间的列映射和转换关系。

**目标数据库和表**

根据 CSV 数据中需要导入的几列（例如除第五列性别外的其余五列需要导入至 StarRocks）， 在 StarRocks 集群的目标数据库 `example_db` 中创建表 `example_tbl2`。

```SQL
CREATE TABLE example_db.example_tbl2 ( 
    `order_id` bigint NOT NULL COMMENT "订单编号",
    `pay_dt` date NOT NULL COMMENT "支付日期", 
    `customer_name` varchar(26) NULL COMMENT "顾客姓名", 
    `nationality` varchar(26) NULL COMMENT "国籍", 
    `price` double NULL COMMENT "支付金额"
) 
DUPLICATE KEY (order_id,pay_dt) 
DISTRIBUTED BY HASH(`order_id`);
```

**导入作业**

本示例中，由于无需导入 CSV 数据的第五列至目标表，因此`COLUMNS`中把第五列临时命名为 `temp_gender` 用于占位，其他列都直接映射至表 `example_tbl2` 中。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl2_ordertest1 ON example_tbl2
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, temp_gender, price)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### 设置过滤条件 筛选待导入的数据

如果仅导入满足条件的数据，则可以在 WHERE 子句中设置过滤条件，例如`price > 100`。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price),
WHERE price > 100
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### 设置导入任务为严格模式

在`PROPERTIES`中设置`"strict_mode" = "true"`，表示导入作业为严格模式。如果源数据某列的值为 NULL，但是目标表中该列不允许为 NULL，则该行数据会被过滤掉。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
"strict_mode" = "true" -- 设置导入作业为严格模式
)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### 设置导入任务的容错率

如果业务场景对数据质量的有要求，则需要设置参数`max_batch_rows`和`max_error_number`设置错误检测窗口的范围和允许的错误数据行数的上限，当错误数据行数超过该值时，导入作业会暂停。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
"max_batch_rows" = "100000", -- 错误检测窗口范围为单个 Routine Load 导入任务所消费的 10 * max-batch-rows 行数。
"max_error_number" = "100" -- 错误检测窗口范围内允许的错误数据行数的上限
)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### 指定安全协议为 SSL 并配置相关参数

如果需要指定 BE 访问 Kafka 时使用的安全协议为 SSL，则需要配置 `"property.security.protocol" = "ssl"` 等参数。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
"desired_concurrent_number" = "5"
)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1",
    "property.security.protocol" = "ssl", -- 使用 SSL 加密
    "property.ssl.ca.location" = "FILE:ca-cert", -- CA 证书的位置
    -- 如果 Kafka Server 端开启了 Client 身份认证，则还需设置如下三个参数：
    "property.ssl.certificate.location" = "FILE:client.pem", -- Client 的 Public Key 的位置
    "property.ssl.key.location" = "FILE:client.key", -- Client 的 Private Key 的位置
    "property.ssl.key.password" = "abcdefg" -- Client 的 Private Key 的密码
);
```

#### 设置 `trim_space`、`enclose` 和 `escape`

假设 Kafka 集群的 Topic `test_csv` 存在如下 CSV 格式的数据，其中 CSV 数据中列的含义依次是订单编号、支付日期、顾客姓名、国籍、性别、支付金额。

```Plain
 "2020050802" , "2020-05-08" , "Johann Georg Faust" , "Deutschland" , "male" , "895"
 "2020050802" , "2020-05-08" , "Julien Sorel" , "France" , "male" , "893"
 "2020050803" , "2020-05-08" , "Dorian Grey\,Lord Henry" , "UK" , "male" , "1262"
 "2020050901" , "2020-05-09" , "Anna Karenina" , "Russia" , "female" , "175"
 "2020051001" , "2020-05-10" , "Tess Durbeyfield" , "US" , "female" , "986"
```
如果要把 Topic `test_csv` 中所有的数据都导入到 `example_tbl1` 中，并且希望去除被 `enclose` 指定字符括起来的字段前后的空格、 指定 `enclose` 字符为双引号 (")、并且指定 `escape` 字符为斜杠 (`\`)，可以执行如下语句：

```SQL

CREATE ROUTINE LOAD example_db.example_tbl1_test_csv ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
    "trim_space"="true",
    "enclose"="\"",
    "escape"="\\",
)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic"="test_csv",
    "property.kafka_default_offsets"="OFFSET_BEGINNING"
);
```


### 导入 JSON 格式数据

#### 目标表的列名与 JSON 数据的 Key 一致

可以使用简单模式导入数据，即创建导入作业时无需使用 `jsonpaths` 和 `COLUMNS` 参数。StarRocks 会按照目标表的列名去对应 JSON 数据的 Key。

**数据集**

假设 Kafka 集群的 Topic `ordertest2` 中存在如下 JSON 数据。

```JSON
{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US","pay_time": 1589191487,"price": 875}
{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK","pay_time": 1589191487,"price": 895}
{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry","country": "France","pay_time": 1589191487,"price": 895}
```

> 注意
>
> 这里每行一个 JSON 对象必须在一个 Kafka 消息中，否则会出现“JSON 解析错误”的问题。

**目标数据库和表**

在 StarRocks 集群的目标数据库 `example_db` 中创建表 `example_tbl3` ，并且列名与 JSON 数据中需要导入的 Key 一致。

```SQL
CREATE TABLE example_db.example_tbl3 ( 
    `commodity_id` varchar(26) NULL COMMENT "品类ID", 
    `customer_name` varchar(26) NULL COMMENT "顾客姓名", 
    `country` varchar(26) NULL COMMENT "顾客国籍", 
    `pay_time` bigint(20) NULL COMMENT "支付时间", 
    `price` double SUM NULL COMMENT "支付金额") 
AGGREGATE KEY(`commodity_id`,`customer_name`,`country`,`pay_time`) 
DISTRIBUTED BY HASH(`commodity_id`);
```

**导入作业**

提交导入作业时使用简单模式，即无需使用`jsonpaths` 和 `COLUMNS` 参数，就可以将 Kafka 集群的 Topic `ordertest2` 中的 JSON 数据导入至目标表 `example_tbl3` 中。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl3_ordertest2 ON example_tbl3
PROPERTIES
(
    "format" = "json"
 )
FROM KAFKA
(
    "kafka_broker_list" = "<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest2"
);
```

> **说明**
>
> - 如果 JSON 数据最外层是数组结构，则需要在`PROPERTIES`设置`"strip_outer_array"="true"`，表示裁剪最外层的数组结构。并且需要注意在设置 `jsonpaths` 时，整个 JSON 数据的根节点是裁剪最外层的数组结构后**展平的 JSON 对象**。
> - 如果不需要导入整个 JSON 数据，则需要使用 `json_root` 指定实际所需导入的 JSON 数据根节点。

#### 目标表存在衍生列，其列值通过表达式计算生成

需要使用匹配模式导入数据，即需要使用 `jsonpaths` 和 `COLUMNS` 参数，`jsonpaths`指定待导入 JSON 数据的 Key，`COLUMNS` 参数指定待导入 JSON 数据的 Key 与目标表的列的映射关系和数据转换关系。

**数据集**

假设 Kafka 集群的 Topic `ordertest2` 中存在如下 JSON 格式的数据。

```JSON
{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US","pay_time": 1589191487,"price": 875}
{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK","pay_time": 1589191487,"price": 895}
{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry","country": "France","pay_time": 1589191487,"price": 895}
```

**目标数据库和表**

假设在 StarRocks 集群的目标数据库 `example_db` 中存在目标表 `example_tbl4` 其中有一列衍生列 `pay_dt`，是基于 JSON 数据的Key `pay_time` 进行计算后的数据。其建表语句如下：

```SQL
CREATE TABLE example_db.example_tbl4 ( 
    `commodity_id` varchar(26) NULL COMMENT "品类ID", 
    `customer_name` varchar(26) NULL COMMENT "顾客姓名", 
    `country` varchar(26) NULL COMMENT "顾客国籍",
    `pay_time` bigint(20) NULL COMMENT "支付时间",  
    `pay_dt` date NULL COMMENT "支付日期", 
    `price`double SUM NULL COMMENT "支付金额") 
AGGREGATE KEY(`commodity_id`,`customer_name`,`country`,`pay_time`,`pay_dt`) 
DISTRIBUTED BY HASH(`commodity_id`);
```

**导入作业**

提交导入作业时使用匹配模式。使用 `jsonpaths` 指定待导入 JSON 数据的 Key。并且由于 JSON 数据中 key `pay_time` 需要转换为 DATE 类型，才能导入到目标表的列 `pay_dt`，因此 `COLUMNS` 中需要使用函数`from_unixtime`进行转换。JSON 数据的其他 key 都能直接映射至表 `example_tbl4` 中。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl4_ordertest2 ON example_tbl4
COLUMNS(commodity_id, customer_name, country, pay_time, pay_dt=from_unixtime(pay_time, '%Y%m%d'), price)
PROPERTIES
(
    "format" = "json",
    "jsonpaths" = "[\"$.commodity_id\",\"$.customer_name\",\"$.country\",\"$.pay_time\",\"$.price\"]"
 )
FROM KAFKA
(
    "kafka_broker_list" = "<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest2"
);
```

> **说明**
>
> - 如果 JSON 数据最外层是数组结构，则需要在`PROPERTIES`设置`"strip_outer_array"="true"`，表示裁剪最外层的数组结构。并且需要注意在设置 `jsonpaths` 时，整个 JSON 数据的根节点是裁剪最外层的数组结构后**展平的 JSON 对象**。
> - 如果不需要导入整个 JSON 数据，则需要使用 `json_root` 指定实际所需导入的 JSON 数据根节点。

#### 目标表存在衍生列，其列值通过 CASE 表达式计算生成

**数据集**

假设 Kafka 集群的 Topic `topic-expr-test` 中存在如下 JSON 格式的数据。

```JSON
{"key1":1, "key2": 21}
{"key1":12, "key2": 22}
{"key1":13, "key2": 23}
{"key1":14, "key2": 24}
```

**目标数据库和表**

假设在 StarRocks 集群的目标数据库 `example_db` 中存在目标表 `tbl_expr_test` 包含两列，其中列 `col2` 的值基于 JSON 数据进行 CASE 表达式计算得出。其建表语句如下：

```SQL
CREATE TABLE tbl_expr_test (
    col1 string, col2 string)
DISTRIBUTED BY HASH (col1);
```

**导入作业**

目标表中列 `col2` 的值需要基于 JSON 数据进行 CASE 表达式计算后得出，因此您需要在导入作业中的 `COLUMNS` 参数配置对应的 CASE 表达式。

```SQL
CREATE ROUTINE LOAD rl_expr_test ON tbl_expr_test
COLUMNS (
      key1,
      key2,
      col1 = key1,
      col2 = CASE WHEN key1 = "1" THEN "key1=1" 
                  WHEN key1 = "12" THEN "key1=12"
                  ELSE "nothing" END) 
PROPERTIES ("format" = "json")
FROM KAFKA
(
    "kafka_broker_list" = "<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "topic-expr-test"
);
```

**查询数据**

查询目标表中的数据，返回结果显示列 `col2` 的值是使用 CASE 表达式计算后输出的值。

```SQL
MySQL [example_db]> SELECT * FROM tbl_expr_test;
+------+---------+
| col1 | col2    |
+------+---------+
| 1    | key1=1  |
| 12   | key1=12 |
| 13   | nothing |
| 14   | nothing |
+------+---------+
4 rows in set (0.015 sec)
```

#### 指定实际待导入 JSON 数据的根节点

如果不需要导入整个 JSON 数据，则需要使用 `json_root` 指定实际上所需导入的 JSON 数据的根对象，参数取值为合法的 JsonPath。

**数据集**

假设 Kafka 集群的 Topic `ordertest3` 中存在如下 JSON 格式的数据，实际导入时仅需要导入 key `RECORDS`的值。

```JSON
{"RECORDS":[{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US","pay_time": 1589191487,"price": 875},{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK","pay_time": 1589191487,"price": 895},{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry","country": "France","pay_time": 1589191487,"price": 895}]}
```

**目标数据库和表**

假设在 StarRocks 集群的目标数据库 `example_db` 中存在目标表 `example_tbl3` ，其建表语句如下：

```SQL
CREATE TABLE example_db.example_tbl3 ( 
    `commodity_id` varchar(26) NULL COMMENT "品类ID", 
    `customer_name` varchar(26) NULL COMMENT "顾客姓名", 
    `country` varchar(26) NULL COMMENT "顾客国籍", 
    `pay_time` bigint(20) NULL COMMENT "支付时间", 
    `price`double SUM NULL COMMENT "支付金额") 
AGGREGATE KEY(`commodity_id`,`customer_name`,`country`,`pay_time`) 
DISTRIBUTED BY HASH(`commodity_id`);
```

**导入作业**

提交导入作业，设置`"json_root" = "$.RECORDS"`指定实际待导入的 JSON 数据的根节点。并且由于实际待导入的 JSON 数据是数组结构，因此还需要设置`"strip_outer_array" = "true"`，裁剪外层的数组结构。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl3_ordertest3 ON example_tbl3
PROPERTIES
(
    "format" = "json",
    "strip_outer_array" = "true",
    "json_root" = "$.RECORDS"
 )
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest2"
);
```

### 导入 Avro 数据

自 3.0.1 版本开始，StarRocks 支持使用 Routine Load 导入 Avro 数据。

#### Avro schema 只包含简单数据类型

假设 Avro schema 只包含简单数据类型，并且您需要导入 Avro 数据中的所有字段。

**数据集**

**Avro schema**

1. 创建如下 Avro schema 文件 `avro_schema1.avsc`：

      ```json
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

2. 注册该 Avro schema 至 [Schema Registry](https://docs.confluent.io/platform/current/schema-registry/index.html)。

**Avro 数据**

构建 Avro 数据并且发送至 Kafka 集群的 topic `topic_1`。

**目标数据库和表**

根据 Avro 数据中需要导入的字段，在 StarRocks 集群的目标数据库 `sensor` 中创建表 `sensor_log1`。表的列名与 Avro 数据的字段名保持一致。两者的数据类型映射关系，请参见xxx。

```SQL
CREATE TABLE sensor.sensor_log1 ( 
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

**导入作业**

提交导入作业时使用简单模式，即无需使用 `jsonpaths` 参数，就可以将 Kafka 集群的 Topic `topic_1` 中的 Avro 数据导入至数据库 `sensor` 中的表 `sensor_log1`。

```sql
CREATE ROUTINE LOAD sensor.sensor_log_load_job1 ON sensor_log1  
PROPERTIES  
(  
  "format" = "avro"  
)  
FROM KAFKA  
(  
  "kafka_broker_list" = "<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>,...",
  "confluent.schema.registry.url" = "http://172.xx.xxx.xxx:8081",  
  "kafka_topic" = "topic_1",  
  "kafka_partitions" = "0,1,2,3,4,5",  
  "property.kafka_default_offsets" = "OFFSET_BEGINNING"  
);
```

#### Avro schema 嵌套 Record 类型的字段

假设 Avro schema嵌套Record类型的字段，并且您需要导入嵌套 Record 字段中的子字段。

**数据集**

**Avro schema**

1. 创建如下 Avro schema文件`avro_schema2.avsc`。其中最外层 Record的字段依次是`id`、`name`、`checked`、`sensor_type`和`data`。并且字段`data`包含嵌套Record`data_record`。

   ```JSON
   {
       "type": "record",
       "name": "sensor_log",
       "fields" : [
           {"name": "id", "type": "long"},
           {"name": "name", "type": "string"},
           {"name": "checked", "type" : "boolean"},
           {"name": "sensor_type", "type": {"type": "enum", "name": "sensor_type_enum", "symbols" : ["TEMPERATURE", "HUMIDITY", "AIR-PRESSURE"]}},
           {"name": "data", "type": 
               {
                   "type": "record",
```JSON
{
    "type": "record",
    "name": "sensor_log3",
    "fields" : [
        {"name": "id", "type": "long"},
        {"name": "name", "type": "string"},
        {"name": "checked", "type" : "boolean"},
        {"name": "sensor_type", "type": {"type": "enum", "name": "sensor_type_enum", "symbols" : ["TEMPERATURE", "HUMIDITY", "AIR-PRESSURE"]}},
        {"name": "data", "type": [null,
              {
                  "type": "record",
                  "name": "data_record",
                  "fields" : [
                      {"name": "data_x", "type" : "boolean"},
                      {"name": "data_y", "type": "long"}
                  ]
              }
            ]
        }
    ]
}
```

#### 注册该 Avro schema 至 [Schema Registry](https://docs.confluent.io/platform/current/schema-registry/index.html)。

**Avro 数据**

构建 Avro 数据并且发送至 Kafka 集群的 topic `topic_3`。

**目标数据库和表**

根据 Avro 数据中需要导入的字段，在 StarRocks 集群的目标数据库 `sensor` 中创建表 `sensor_log3` 。

假设您除了需要导入最外层 Record 的 `id`、`name`、`checked`、`sensor_type` 字段之外，还需要导入 Union 类型的字段 `data` 中元素 `data_record` 包含的字段 `data_y`。

```sql
CREATE TABLE sensor.sensor_log3 ( 
    `id` bigint NOT NULL COMMENT "sensor id",
    `name` varchar(26) NOT NULL COMMENT "sensor name", 
    `checked` boolean NOT NULL COMMENT "checked", 
    `sensor_type` varchar(26) NOT NULL COMMENT "sensor type",
    `data_y` long NULL COMMENT "sensor data" 
) 
ENGINE=OLAP 
DUPLICATE KEY (id) 
DISTRIBUTED BY HASH(`id`);
```

**导入作业**

提交导入作业，使用 `jsonpaths` 指定实际待导入的 Avro 数据的字段。其中您需要指定字段 `data_y`的`jsonpaths` 为 `"$.data.data_y"`。

```sql
CREATE ROUTINE LOAD sensor.sensor_log_load_job3 ON sensor_log3  
PROPERTIES  
(  
  "format" = "avro",
  "jsonpaths" = "[\"$.id\",\"$.name\",\"$.checked\",\"$.sensor_type\",\"$.data.data_y\"]"
)  
FROM KAFKA  
(  
  "kafka_broker_list" = "<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>,...",
  "confluent.schema.registry.url" = "http://172.xx.xxx.xxx:8081",  
  "kafka_topic" = "topic_1",  
  "kafka_partitions" = "0,1,2,3,4,5",  
  "property.kafka_default_offsets" = "OFFSET_BEGINNING"  
);
```