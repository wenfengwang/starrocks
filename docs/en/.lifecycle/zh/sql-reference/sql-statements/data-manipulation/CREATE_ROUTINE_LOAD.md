---
displayed_sidebar: English
---

# 创建例程加载

## 描述

例程加载可以持续消费来自 Apache Kafka® 的消息，并将数据加载到 StarRocks 中。例程加载可以使用来自 Kafka 集群的 CSV、JSON 和 Avro（从 v3.0.1 开始支持）数据，并通过多种安全协议访问 Kafka，包括 `plaintext` 、 `ssl`、 `sasl_plaintext` 和 `sasl_ssl`。

本主题描述了 CREATE ROUTINE LOAD 语句的语法、参数和示例。

> **注意**
>
> - 有关例程加载的应用场景、原理和基本操作，请参见 [从 Apache Kafka® 持续加载数据](../../../loading/RoutineLoad.md)。
> - 您只能以对 StarRocks 表具有 INSERT 权限的用户身份将数据加载到 StarRocks 表中。如果您没有 INSERT 权限，请按照 [GRANT](../account-management/GRANT.md) 中的说明，将 INSERT 权限授予您用于连接到 StarRocks 集群的用户。

## 语法

```SQL
CREATE ROUTINE LOAD <database_name>.<job_name> ON <table_name>
[load_properties]
[job_properties]
FROM data_source
[data_source_properties]
```

## 参数

### `database_name`、 `job_name` `table_name`

`database_name`

可选。StarRocks 数据库的名称。

`job_name`

必填。例程加载作业的名称。一个表可以从多个例程加载作业接收数据。建议您使用可识别的信息（例如 Kafka 主题名称和大致作业创建时间）设置有意义的例程加载作业名称，以区分多个例程加载作业。例程加载作业的名称在同一数据库中必须是唯一的。

`table_name`

必填。加载数据的 StarRocks 表的名称。

### `load_properties`

可选。数据的属性。语法：

```SQL
[COLUMNS TERMINATED BY '<column_separator>'],
[ROWS TERMINATED BY '<row_separator>'],
[COLUMNS (<column1_name>[, <column2_name>, <column_assignment>, ... ])],
[WHERE <expr>],
[PARTITION (<partition1_name>[, <partition2_name>, ...])]
[TEMPORARY PARTITION (<temporary_partition1_name>[, <temporary_partition2_name>, ...])]
```

`COLUMNS TERMINATED BY`

CSV 格式数据的列分隔符。默认列分隔符为 `\t` （Tab）。例如，您可以使用 `COLUMNS TERMINATED BY ","` 来指定逗号作为列分隔符。

> **注意**
>
> - 确保此处指定的列分隔符与要导入的数据中的列分隔符相同。
> - 您可以使用长度不超过 50 个字节的 UTF-8 字符串，例如逗号（,）、制表符或竖线（|）作为文本分隔符。
> - 空值用 `\N` 表示。例如，数据记录由三列组成，数据记录在第一列和第三列中保存数据，但在第二列中不保存数据。在这种情况下，您需要在第二列使用 `\N` 表示空值。这意味着必须将记录编译为 `a,\N,b` 而不是 `a,,b`。`a,,b` 表示记录的第二列包含一个空字符串。

`ROWS TERMINATED BY`

CSV 格式数据的行分隔符。默认行分隔符为 `\n`。

`COLUMNS`

源数据中的列与 StarRocks 表中的列之间的映射关系。有关详细信息，请参阅本主题中的[列映射](#column-mapping)。

- `column_name`：如果源数据的某列无需任何计算即可映射到 StarRocks 表的某列，则只需指定列名即可。这些列可以称为映射列。
- `column_assignment`：如果源数据的某列无法直接映射到 StarRocks 表的某列，且在加载数据之前必须使用函数计算该列的值，则必须在中指定计算函数 `expr`。这些列可以称为派生列。建议将派生列放在映射列之后，因为 StarRocks 会先解析映射列。

`WHERE`

筛选条件。只有满足筛选条件的数据才能加载到 StarRocks 中。例如，如果只想引入值 `col1` 大于 `100` 且 `col2` 值等于 `1000` 的行，您可以使用 `WHERE col1 > 100 and col2 = 1000`。

> **注意**
>
> 筛选条件中指定的列可以是源列，也可以是派生列。

`PARTITION`

如果 StarRocks 表分布在分区 p0、p1、p2 和 p3 上，并且您只想将数据加载到 StarRocks 的 p1、p2、p3 中，并过滤掉将存储在 p0 中的数据，则可以指定 `PARTITION(p1, p2, p3)` 为过滤条件。默认情况下，如果不指定此参数，则数据将加载到所有分区中。例如：

```SQL
PARTITION (p1, p2, p3)
```

`TEMPORARY PARTITION`

要将数据加载到其中的[临时分区](../../../table_design/Temporary_partition.md)的名称。您可以指定多个临时分区，这些分区必须用逗号（,）分隔。

### `job_properties`

必填。加载作业的属性。语法：

```SQL
PROPERTIES ("<key1>" = "<value1>"[, "<key2>" = "<value2>" ...])
```

| **属性**              | **必填** | **描述**                                              |
| ------------------------- | ------------ | ------------------------------------------------------------ |
| desired_concurrent_number | 否           | 单个例程加载作业的预期任务并行度。默认值： `3`。实际任务并行度由多个参数的最小值确定： `min(alive_be_number, partition_number, desired_concurrent_number, max_routine_load_task_concurrent_num)`。 <ul><li>`alive_be_number`：活动 BE 节点的数量。</li><li>`partition_number`：要消耗的分区数。</li><li>`desired_concurrent_number`：单个例程加载作业的预期任务并行度。默认值： `3`。</li><li>`max_routine_load_task_concurrent_num`：例程加载作业的默认最大任务并行度，即 `5`。参见 [FE 动态参数](../../../administration/FE_configuration.md#configure-fe-dynamic-parameters)。</li></ul>最大实际任务并行度由活动 BE 节点数或要使用的分区数决定。|
| max_batch_interval        | 否           | 任务的调度间隔，即任务的执行频率。单位：秒。取值范围： `5` ~ `60`。默认值： `10`。建议设置一个大于 `10` 的值。如果调度时间短于 10 秒，则由于加载频率过高，会生成过多的平板电脑版本。 |
| max_batch_rows            | 否           | 此属性仅用于定义错误检测窗口。窗口是单个例程加载任务消耗的数据行数。值为 `10 * max_batch_rows`。默认值为 `10 * 200000 = 200000`。例程加载任务在错误检测窗口中检测错误数据。错误数据是指 StarRocks 无法解析的数据，例如无效的 JSON 格式数据。 |
| max_error_number          | 否           | 错误检测时段内允许的最大错误数据行数。如果错误数据行数超过此值，则加载作业将暂停。您可以执行 [SHOW ROUTINE LOAD](./SHOW_ROUTINE_LOAD.md) 并使用 `ErrorLogUrls` 查看错误日志。之后，您可以根据错误日志更正 Kafka 中的错误。默认值为 `0`，表示不允许出现错误行。<br />**注意**<br />：错误数据行不包括由 WHERE 子句过滤掉的数据行。 |
| strict_mode               | 否           | 指定是否启用[严格模式](../../../loading/load_concept/strict_mode.md)。有效值： `true` 和 `false`。默认值： `false`。启用严格模式后，如果加载的数据中某列的值为 `NULL`，但目标表不允许该列的值为 `NULL`，则该数据行将被过滤掉。 |
| log_rejected_record_num | 否 | 指定可以记录的最大非限定数据行数。从 v3.1 开始支持此参数。有效值： `0`、 `-1` 和任何非零正整数。默认值： `0`。<ul><li>该值 `0` 指定不会记录筛选出的数据行。</li><li>该值 `-1` 指定将记录筛选出的所有数据行。</li><li>非零正整数（例如） `n` 指定每个 BE 上最多可以记录的筛选出的数据行数。</li></ul> |
| 时区                  | 否           | 加载作业使用的时区。默认值： `Asia/Shanghai`。此参数的值会影响 strftime()、alignment_timestamp() 和 from_unixtime() 等函数返回的结果。此参数指定的时区是会话级别的时区。有关详细信息，请参阅 [配置时区](../../../administration/timezone.md)。 |
| merge_condition           | 否           | 指定要用作确定是否更新数据的条件的列的名称。只有当要加载到此列的数据值大于或等于此列的当前值时，才会更新数据。有关详细信息，请参阅 [通过加载更改数据](../../../loading/Load_to_Primary_Key_tables.md)。<br />**注意**<br />：只有主键表支持条件更新。指定的列不能是主键列。 |
| 格式                    | 否           | 要加载的数据的格式。取值范围： `CSV`、 `JSON` 和 `Avro`（从 v3.0.1 开始支持）。默认值： `CSV`。 |
| trim_space                | 否           | 指定当数据文件为 CSV 格式时，是否从数据文件中删除列分隔符前后的空格。类型：BOOLEAN。默认值： `false`。<br />对于某些数据库，当您将数据导出为 CSV 格式的数据文件时，会将空格添加到列分隔符中。此类空间称为前导空格或尾随空格，具体取决于它们的位置。通过设置该 `trim_space` 参数，您可以让 StarRocks 在数据加载过程中去除这些不必要的空格。<br />需要注意的是，StarRocks 不会移除用 -指定字符包裹的字段中的空格（包括前导空格和尾随空格`enclose`）。例如，以下字段值使用管道 （<code class="language-text">&#124;</code>） 作为列分隔符，双引号 （） 作为`"` `enclose`-指定字符：<code class="language-text">&#124;《爱上星石》&#124;</code>。如果设置为 `trim_space` `true`，StarRocks 会将上述字段值处理为 <code class="language-text">&#124;”爱上星石“&#124;</code>。 |

| 包围符号                   | 否           |指定数据文件中字段值的包围符号，遵循[RFC4180](https://www.rfc-editor.org/rfc/rfc4180)。类型：单字节字符。默认值：`NONE`。最常用的字符是单引号（`'`）和双引号（`"`）。<br />使用`enclose`指定的字符包围的所有特殊字符（包括行分隔符和列分隔符）都被视为普通符号。StarRocks不仅支持RFC4180，还允许您指定任何单字节字符作为`enclose`指定的字符。<br />如果字段值包含`enclose`指定的字符，您可以使用相同的字符对该`enclose`指定的字符进行转义。例如，将`enclose`设置为`"`，字段值为`a "quoted" c`。在这种情况下，您可以在数据文件中输入字段值`"a ""quoted"" c"`。 |
| 转义符                    | 否           |指定用于转义各种特殊字符的字符，例如行分隔符、列分隔符、转义字符和`enclose`指定的字符，StarRocks将其视为普通字符，并解析为它们所在的字段值的一部分。类型：单字节字符。默认值：`NONE`。最常用的字符是斜杠（\），在SQL语句中必须写成双斜杠（\\）。<br />**注意**<br />：`escape`指定的字符应用于每对`enclose`指定的字符的内部和外部。以下是两个例子：<ul><li>当您将`enclose`设置为`"`，`escape`设置为`\`时，StarRocks将`"say \"Hello world\""`解析为`say "Hello world"`。</li><li>假设列分隔符为逗号（,）。当您将`escape`设置为`\`时，StarRocks将`a, b\, c`解析为两个单独的字段值：`a`和`b, c`。</li></ul> |
| strip_outer_array         | 否           | 指定是否去除JSON格式数据的最外层数组结构。有效值：`true`和`false`。默认值：`false`。在实际业务场景中，JSON格式的数据可能具有最外层的数组结构，如一对方括号所示`[]`。在这种情况下，建议您将该参数设置为`true`，这样StarRocks就会去掉最外层的方括号`[]`，并将每个内部数组作为单独的数据记录加载。如果将该参数设置为`false`，StarRocks将整个JSON格式的数据解析为一个数组，并将该数组作为单个数据记录加载。以JSON格式的数据`[{"category" : 1, "author" : 2}, {"category" : 3, "author" : 4} ]`为例。如果将该参数设置为`true`，`{"category" : 1, "author" : 2}`和`{"category" : 3, "author" : 4}`将被解析为两条独立的数据记录，并加载到两个StarRocks数据行中。 |
| jsonpaths（JSON路径）                 | 否           | 要从JSON格式的数据加载的字段的名称。此参数的值是有效的JsonPath表达式。更多信息，请参阅本文的[StarRocks表包含使用表达式生成值的派生列](#starrocks-table-contains-derived-columns-whose-values-are-generated-by-using-expressions)。|
| json_root                 | 否           | 要加载的JSON格式数据的根元素。StarRocks通过`json_root`提取根节点的元素进行解析。默认情况下，该参数的值为空，表示将加载所有JSON格式的数据。有关详细信息，请参阅[在本主题中](#specify-the-root-element-of-the-json-formatted-data-to-be-loaded)指定要加载的JSON格式数据的根元素。|
| task_consume_second | 否 |指定例程加载作业中每个例程加载任务消耗数据的最长时间。单位：秒。与[FE动态参数（适用于集群中的所有例程加载作业）不同，此参数](../../../administration/FE_configuration.md)`routine_load_task_consume_second`特定于单个例程加载作业，后者更灵活。从v3.1.0开始支持该参数。<ul> <li>当`task_consume_second`和`task_timeout_second`未配置时，StarRocks使用FE动态参数`routine_load_task_consume_second`和`routine_load_task_timeout_second`控制负载行为。</li> <li>当只配置`task_consume_second`时，默认值为`task_timeout_second`计算为`task_consume_second` * 4。</li> <li>当只配置`task_timeout_second`时，默认值为`task_consume_second`计算为`task_timeout_second`/4。</li> </ul> |
|task_timeout_second|否|指定例程加载作业中每个例程加载任务的超时持续时间。单位：秒。与[FE动态参数（适用于集群内的所有例程加载作业）不同，此参数](../../../administration/FE_configuration.md)`routine_load_task_timeout_second`特定于单个例程加载作业，后者更灵活。从v3.1.0开始支持该参数。<ul> <li>当`task_consume_second`和`task_timeout_second`未配置时，StarRocks使用FE动态参数`routine_load_task_consume_second`和`routine_load_task_timeout_second`控制负载行为。</li> <li>当只配置`task_timeout_second`时，默认值为`task_consume_second`计算为`task_timeout_second`/4。</li> <li>当只配置`task_consume_second`时，默认值为`task_timeout_second`计算为`task_consume_second` * 4。</li> </ul>|

### `data_source`, `data_source_properties`

必填。数据源和相关属性。

```sql
FROM <data_source>
 ("<key1>" = "<value1>"[, "<key2>" = "<value2>" ...])
```

`data_source`

必填。要加载的数据源。有效值：`KAFKA`。

`data_source_properties`

数据源的属性。

| 属性          | 必填 | 描述                                                  |
| ----------------- | -------- | ------------------------------------------------------------ |
| kafka_broker_list | 是      | Kafka的代理连接信息。格式为`<kafka_broker_ip>:<broker_ port>`。多个代理之间用逗号（,）分隔。Kafka代理使用的默认端口是`9092`。示例：`"kafka_broker_list" = ""xxx.xx.xxx.xx:9092,xxx.xx.xxx.xx:9092"`. |
| kafka_topic       | 是      | 要消费的Kafka主题。例程加载作业只能从一个主题消费消息。 |
| kafka_partitions  | 否       | 要消费的Kafka分区，例如`"kafka_partitions" = "0, 1, 2, 3"`。如果未指定此属性，则默认情况下消费所有分区。 |
| kafka_offsets     | 否       |要从Kafka分区中消费数据的起始偏移量，如在`kafka_partitions`中指定的那样。如果未指定此属性，则例程加载作业将从`kafka_partitions`中的最新偏移量开始消费数据。有效值：<ul><li>特定偏移量：从特定偏移量开始消费数据。</li><li>`OFFSET_BEGINNING`：从尽可能早的偏移量开始消费数据。</li><li>`OFFSET_END`：从最新偏移量开始消费数据。</li></ul>多个起始偏移量用逗号（,）分隔，例如`"kafka_offsets" = "1000, OFFSET_BEGINNING, OFFSET_END, 2000"`。|
| property.kafka_default_offsets| 否|所有消费者分区的默认起始偏移量。此属性支持的值与`kafka_offsets`属性相同。|
| confluent.schema.registry.url|否 |Avro模式注册表的URL。StarRocks通过此URL获取Avro模式。格式如下：<br />`confluent.schema.registry.url = http[s]://[<schema-registry-api-key>:<schema-registry-api-secret>@]<hostname or ip address>[:<port>]`|

#### 更多与数据源相关的属性

您可以指定其他数据源（Kafka）相关属性，这些属性等同于使用Kafka命令行`--property`。有关更多受支持的属性，请参阅[librdkafka配置属性中的Kafka使用者客户端的属性](https://github.com/confluentinc/librdkafka/blob/master/CONFIGURATION.md)。

> **注意**
>
> 如果属性的值是文件名，请在文件名前添加关键字`FILE:`。有关如何创建文件的信息，请参阅[CREATE FILE](../Administration/CREATE_FILE.md)。

- **指定要使用的所有分区的默认初始偏移量**

```SQL
"property.kafka_default_offsets" = "OFFSET_BEGINNING"
```

- **指定例程加载作业使用的消费组的ID**

```SQL
"property.group.id" = "group_id_0"
```

如果未指定`property.group.id`，StarRocks将根据例程加载作业的名称生成一个基于随机UUID的随机值，格式为`{job_name}_{random uuid}`，例如`simple_job_0a64fe25-3983-44b2-a4d8-f52d3af4c3e8`。

- **指定BE用于访问Kafka的安全协议和相关参数**

  安全协议可以指定为`plaintext`（默认）、`ssl`、`sasl_plaintext`或`sasl_ssl`。您需要根据指定的安全协议配置相关参数。

  当安全协议设置为`sasl_plaintext`或`sasl_ssl`时，支持以下SASL身份验证机制：

  - PLAIN
  - SCRAM-SHA-256和SCRAM-SHA-512
  - OAUTHBEARER

  - 使用SSL安全协议访问Kafka：

    ```SQL
    -- 指定安全协议为SSL。
    "property.security.protocol" = "ssl"
    -- 用于验证kafka代理密钥的CA证书（文件或目录路径）。
    -- 如果Kafka服务器启用客户端身份验证，则还需要以下三个参数：
    -- 用于认证的客户端公钥的路径。
    "property.ssl.certificate.location" = "FILE:client.pem"
    -- 用于认证的客户端私钥的路径。
    "property.ssl.key.location" = "FILE:client.key"
    -- 客户端私钥的密码。
    "property.ssl.key.password" = "xxxxxx"
    ```

  - 使用SASL_PLAINTEXT安全协议和SASL/PLAIN身份验证机制访问Kafka：

    ```SQL
    -- 指定安全协议为SASL_PLAINTEXT
    "property.security.protocol" = "SASL_PLAINTEXT"
    -- 指定SASL机制为PLAIN，这是一种简单的用户名/密码身份验证机制
    "property.sasl.mechanism" = "PLAIN" 
    -- SASL用户名
    "property.sasl.username" = "admin"
    -- SASL密码
    "property.sasl.password" = "xxxxxx"
    ```

### FE和BE配置项

有关与例程加载相关的FE和BE配置项，请参见[配置项](../../../administration/FE_configuration.md)。

## 列映射

### 配置加载CSV格式数据的列映射

如果CSV格式数据的列可以依次映射到StarRocks表的列中，则无需配置数据与StarRocks表的列映射关系。


如果 CSV 格式数据的列无法依次映射到 StarRocks 表的列中，则需要使用 `columns` 参数配置数据文件与 StarRocks 表的列映射关系。这包括以下两个用例：

- **列数相同，但列顺序不同。此外，数据文件中的数据在加载到匹配的 StarRocks 表列中之前，不需要通过函数进行计算。**

  - 在 `columns` 参数中，需要按照数据文件列排列顺序相同的顺序指定 StarRocks 表列的名称。

  - 例如，StarRocks 表由三列组成，分别是 `col1`、`col2` 和 `col3`，数据文件也由三列组成，可以映射到 StarRocks 表列 `col3`、`col2` 和 `col1` 顺序。在这种情况下，您需要指定 `"columns: col3, col2, col1"`。

- **不同的列数和不同的列顺序。此外，数据文件中的数据需要经过函数计算，然后才能加载到匹配的 StarRocks 表列中。**

  在 `columns` 参数中，您需要按照数据文件列排列相同的顺序指定 StarRocks 表列的名称，并指定要用于计算数据的函数。下面两个示例：

  - StarRocks 表由三列组成，分别是 `col1`、`col2` 和 `col3`。数据文件由四列组成，其中前三列可以依次映射到 StarRocks 表列 `col1`、`col2`、`col3`，第四列不能映射到 StarRocks 表的任何列。这种情况下，您需要为数据文件的第四列临时指定一个名称，并且该临时名称必须与任何 StarRocks 表列名称不同。例如，可以指定 `"columns: col1, col2, col3, temp"`，其中数据文件的第四列临时命名为 `temp`。
  - StarRocks 表由三列组成，分别是 `year`、`month` 和 `day`。数据文件仅包含一列，该列以格式容纳日期和时间值 `yyyy-mm-dd hh:mm:ss`。在这种情况下，您可以指定 `"columns: col, year = year(col), month=month(col), day=day(col)"`，其中数据文件列和函数的临时名称 `col`，用于 `year = year(col)` 从数据文件列中提取数据，`month=month(col)`，`day=day(col)`，`col` 并将数据加载到映射的 StarRocks 表列中。例如，`year = year(col)` 用于从 `yyyy` 数据文件列 `col` 中提取数据，并将数据加载到 StarRocks 表列 `year` 中。

有关更多示例，请参阅 [配置列映射](#configure-column-mapping)。

### 配置列映射以加载 JSON 格式或 Avro 格式的数据

> **注意**
>
> 从 v3.0.1 开始，StarRocks 支持使用 Routine Load 加载 Avro 数据。加载 JSON 或 Avro 数据时，列映射和转换的配置是相同的。因此，本节以 JSON 数据为例，介绍配置。

如果 JSON 格式数据的 key 与 StarRocks 表的列同名，您可以使用简单模式加载 JSON 格式的数据。在简单模式下，无需指定 `jsonpaths` 参数。此模式要求 JSON 格式的数据必须是用大括号表示的对象 `{}`，例如 `{"category": 1, "author": 2, "price": "3"}`。在此示例中，`category`、`author` 和 `price` 是键名，这些键可以按名称一一对应映射到 `category`、`author`、`price` StarRocks 表的、和列。有关示例，请参阅[简单模式]（目标表的 #Column 名称与 JSON 键一致）。

如果 JSON 格式数据的键名称与 StarRocks 表的列不同，您可以使用匹配模式加载 JSON 格式的数据。在匹配模式下，需要使用 `jsonpaths` and `COLUMNS` 参数指定 JSON 格式数据与 StarRocks 表的列对应关系：

- 在 `jsonpaths` 参数中，将序列中的 JSON 键指定为它们在 JSON 格式数据中的排列方式。
- 在参数中 `COLUMNS`，指定 JSON 键与 StarRocks 表列的对应关系：
  - 参数中指定的列名 `COLUMNS` 将依次映射到 JSON 格式的数据。
  - 参数中指定的列名 `COLUMNS` 将按名称一一对应到 StarRocks 表列中。

例如，[请参见 StarRocks 表包含使用表达式生成值的派生列](#starrocks-table-contains-derived-columns-whose-values-are-generated-by-using-expressions)。

## 例子

### 加载 CSV 格式的数据

本节以 CSV 格式的数据为例，介绍如何采用各种参数设置和组合来满足不同的加载要求。

**准备数据集**

假设您要从名为 `ordertest1` 的 Kafka 主题加载 CSV 格式的数据。数据集中的每条消息都包含六列：订单 ID、付款日期、客户姓名、国籍、性别和价格。

```plaintext
2020050802,2020-05-08,Johann Georg Faust,Deutschland,male,895
2020050802,2020-05-08,Julien Sorel,France,male,893
2020050803,2020-05-08,Dorian Grey,UK,male,1262
2020050901,2020-05-09,Anna Karenina,Russia,female,175
2020051001,2020-05-10,Tess Durbeyfield,US,female,986
2020051101,2020-05-11,Edogawa Conan,japan,male,8924
```

**创建表**

根据 CSV 格式数据的列，在数据库中创建一个名为 `example_tbl1` 的表 `example_db`。

```SQL
CREATE TABLE example_db.example_tbl1 ( 
    `order_id` bigint NOT NULL COMMENT "Order ID",
    `pay_dt` date NOT NULL COMMENT "Payment date", 
    `customer_name` varchar(26) NULL COMMENT "Customer name", 
    `nationality` varchar(26) NULL COMMENT "Nationality", 
    `gender` varchar(26) NULL COMMENT "Gender", 
    `price` double NULL COMMENT "Price") 
DUPLICATE KEY (order_id,pay_dt) 
DISTRIBUTED BY HASH(`order_id`); 
```

#### 使用从指定分区的指定偏移量开始的数据

如果例程加载作业需要使用从指定分区和偏移量开始的数据，则需要配置参数 `kafka_partitions` 和 `kafka_offsets`。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1",
    "kafka_partitions" ="0,1,2,3,4", -- partitions to be consumed
    "kafka_offsets" = "1000, OFFSET_BEGINNING, OFFSET_END, 2000" -- corresponding initial offsets
);
```

#### 通过提高任务并行度来提高加载性能

为了提高加载性能并避免累积消耗，您可以通过增加 `desired_concurrent_number` 的值来增加任务并行度。任务并行性允许将一个例程加载作业拆分为尽可能多的并行任务。

> **注意**
>
> 有关提高加载性能的更多方法，请参阅 [例程加载常见问题解答](../../../faq/loading/Routine_load_faq.md)。

请注意，实际任务并行度由以下多个参数中的最小值决定：

```SQL
min(alive_be_number, partition_number, desired_concurrent_number, max_routine_load_task_concurrent_num)
```

> **注意**
>
> 最大实际任务并行度是活动 BE 节点数或要消耗的分区数。

因此，当活动 BE 节点数和要消耗的分区数大于 `max_routine_load_task_concurrent_num` 和 `desired_concurrent_number` 的值时，可以增加其他两个参数的值，以提高实际任务并行度。

假设要消耗的分区数为 7，活动 BE 节点数为 5，`max_routine_load_task_concurrent_num` 为默认值 `5`。如果要增加实际任务并行度，可以将 `desired_concurrent_number` 设置为 `5`（默认值为 `3`）。在本例中，实际任务并行度 `min(5,7,5,5)` 配置为 `5`。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
"desired_concurrent_number" = "5" -- 将 desired_concurrent_number 的值设置为 5
)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### 配置列映射

如果 CSV 格式数据中的列序列与目标表中的列不一致，假设 CSV 格式数据中的第五列不需要导入到目标表中，则需要通过 `COLUMNS` 参数指定 CSV 格式数据与目标表的列映射关系。

**目标数据库和表**

根据 CSV 格式数据的列，在目标数据库中 `example_db` 创建一个名为 `example_tbl2` 的表。在此方案中，您需要创建与 CSV 格式数据中的五列相对应的五列，但存储性别的第五列除外。

```SQL
CREATE TABLE example_db.example_tbl2 ( 
    `order_id` bigint NOT NULL COMMENT "Order ID",
    `pay_dt` date NOT NULL COMMENT "Payment date", 
    `customer_name` varchar(26) NULL COMMENT "Customer name", 
    `nationality` varchar(26) NULL COMMENT "Nationality", 
    `price` double NULL COMMENT "Price"
) 
DUPLICATE KEY (order_id,pay_dt) 
DISTRIBUTED BY HASH(order_id); 
```

**例行加载作业**

在此示例中，由于 CSV 格式数据中的第五列不需要加载到目标表中，因此临时在 `COLUMNS` 中命名第五列为 `temp_gender`，其他列直接映射到表 `example_tbl2` 中。

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

#### 设置筛选条件

如果只想加载满足特定条件的数据，可以在 `WHERE` 子句中设置过滤条件，例如，`price > 100`。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl2_ordertest1 ON example_tbl2
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price),
WHERE price > 100 -- 设置筛选条件
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### 启用严格模式以过滤掉具有 NULL 值的行

在 `PROPERTIES` 中，您可以设置 `"strict_mode" = "true"`，这意味着例程加载作业处于严格模式。如果源列中有 `NULL` 值，但目标 StarRocks 表列不允许 NULL 值，则会过滤掉源列中保存 NULL 值的行。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
"strict_mode" = "true" -- 启用严格模式
)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### 设置错误容忍度

如果您的业务场景对不合格数据的容忍度较低，则需要通过配置参数 `max_batch_rows` 和 `max_error_number` 来设置错误检测窗口和错误数据的最大行数。当错误检测窗口中的错误数据行数超过 `max_error_number` 的值时，例程加载作业将暂停。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
"max_batch_rows" = "100000",-- max_batch_rows 的值乘以 10 等于错误检测窗口。
"max_error_number" = "100" -- 错误检测窗口内允许的最大错误数据行数。
)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### 指定安全协议为 SSL 并配置相关参数

如果需要将安全协议指定为 BE 用于访问 Kafka 的 SSL，则需要配置 `"property.security.protocol" = "ssl"` 和相关参数。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
-- 指定安全协议为 SSL。
"property.security.protocol" = "ssl",
-- CA 证书的位置。
"property.ssl.ca.location" = "FILE:ca-cert",
-- 如果为 Kafka 客户端启用了身份验证，则需要配置以下属性：
-- Kafka 客户端公钥的位置。
"property.ssl.certificate.location" = "FILE:client.pem",
-- Kafka 客户端私钥的位置。
"property.ssl.key.location" = "FILE:client.key",
-- Kafka 客户端私钥的密码。
"property.ssl.key.password" = "abcdefg"
)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### 设置 trim_space、enclose 和 escape

假设您要从名为 `test_csv` 的 Kafka 主题加载 CSV 格式的数据。数据集中的每条消息都包含六列：订单 ID、付款日期、客户姓名、国籍、性别和价格。

```Plaintext
 "2020050802" , "2020-05-08" , "Johann Georg Faust" , "Deutschland" , "male" , "895"
 "2020050802" , "2020-05-08" , "Julien Sorel" , "France" , "male" , "893"
 "2020050803" , "2020-05-08" , "Dorian Grey\,Lord Henry" , "UK" , "male" , "1262"
 "2020050901" , "2020-05-09" , "Anna Karenina" , "Russia" , "female" , "175"
 "2020051001" , "2020-05-10" , "Tess Durbeyfield" , "US" , "female" , "986"
 "2020051101" , "2020-05-11" , "Edogawa Conan" , "japan" , "male" , "8924"
```

如果要将 Kafka 主题中的所有数据加载到 `example_tbl1` 中，并删除列分隔符前后的空格，并将 `enclose` 设置为 `"`，`escape` 设置为 `\`，请运行以下命令：

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

### 加载 JSON 格式的数据

#### StarRocks 表列名与 JSON 键名一致

**准备数据集**

例如，Kafka 主题中存在以下 JSON 格式的数据 `ordertest2`。

```SQL
{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US","pay_time": 1589191487,"price": 875}
{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK","pay_time": 1589191487,"price": 895}
{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry","country": "France","pay_time": 1589191487,"price": 895}
```

> **注意** 每个 JSON 对象必须位于一条 Kafka 消息中。否则，将发生指示解析 JSON 格式数据失败的错误。

**目标数据库和表**

在 StarRocks 集群的目标数据库 `example_db` 中创建表 `example_tbl3`。表的列名与 JSON 格式数据中的键名一致。

```SQL
CREATE TABLE example_db.example_tbl3 ( 
    commodity_id varchar(26) NULL, 
    customer_name varchar(26) NULL, 
    country varchar(26) NULL, 
    pay_time bigint(20) NULL, 
    price double SUM NULL COMMENT "Price") 
AGGREGATE KEY(commodity_id,customer_name,country,pay_time)
DISTRIBUTED BY HASH(commodity_id); 
```

**例行加载作业**

您可以使用简单模式进行例程加载作业。也就是说，在创建例程加载作业时，您不需要指定 `jsonpaths` 和 `COLUMNS` 参数。StarRocks 会根据目标表 `example_tbl3` 的列名，提取 Kafka 集群中 `ordertest2` Topic 的 JSON 格式数据的键，并将 JSON 格式的数据加载到目标表中。

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

> **注意**
>
> - 如果 JSON 格式数据的最外层是数组结构，则需要设置 `"strip_outer_array"="true"` 在 `PROPERTIES` 中去除最外层的数组结构。此外，当需要指定 `jsonpaths` 时，整个 JSON 格式数据的根元素是扁平化的 JSON 对象，因为 JSON 格式数据的最外层数组结构被剥离。
> - 可以使用 `json_root` 指定 JSON 格式数据的根元素。

#### StarRocks 表包含使用表达式生成值的派生列

**准备数据集**

例如，Kafka 集群 `ordertest2` 的 Topic 中存在以下 JSON 格式的数据。

```SQL
{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US","pay_time": 1589191487,"price": 875}
{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK","pay_time": 1589191487,"price": 895}
{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry","country": "France","pay_time": 1589191487,"price": 895}
```

**目标数据库和表**

在 StarRocks 集群的目标数据库 `example_db` 中创建表 `example_tbl4`。表中的 `pay_dt` 列是一个派生列，其值是通过计算 JSON 格式数据中 `pay_time` 键的值生成的。

```SQL
CREATE TABLE example_db.example_tbl4 ( 
    `commodity_id` varchar(26) NULL, 
    `customer_name` varchar(26) NULL, 
    `country` varchar(26) NULL,
    `pay_time` bigint(20) NULL,  
    `pay_dt` date NULL, 
    `price` double SUM NULL) 
AGGREGATE KEY(`commodity_id`,`customer_name`,`country`,`pay_time`,`pay_dt`) 
DISTRIBUTED BY HASH(`commodity_id`); 
```

**例行加载作业**

您可以使用匹配模式进行例程加载作业。也就是说，在创建例程加载作业时，您需要指定 `jsonpaths` 和 `COLUMNS` 参数。

您需要指定 JSON 格式数据的键，并在参数中按顺序排列。并且由于 JSON 格式数据中 `pay_time` 键的值需要先转换为 DATE 类型，然后才能将值存储在表 `example_tbl4` 的 `pay_dt` 列中，因此您需要在 `COLUMNS` 中指定计算 `pay_dt=from_unixtime(pay_time,'%Y%m%d')`。JSON 格式数据中其他键的值可以直接映射到表 `example_tbl4` 中。

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

> **注意**
>
> - 如果 JSON 数据的最外层是数组结构，则需要在 `PROPERTIES` 中设置 `"strip_outer_array"="true"` 以去除最外层的数组结构。此外，当您需要指定 `jsonpaths` 时，整个 JSON 数据的根元素是扁平化的 JSON 对象，因为 JSON 数据的最外层数组结构被剥离。
> - 可以使用 `json_root` 指定 JSON 格式数据的根元素。

#### StarRocks 表包含使用 CASE 表达式生成的派生列

**准备数据集**

例如，Kafka 主题中存在以下 JSON 格式的数据 `topic-expr-test`。

```JSON
{"key1":1, "key2": 21}
{"key1":12, "key2": 22}
{"key1":13, "key2": 23}
{"key1":14, "key2": 24}
```

**目标数据库和表**

在 StarRocks 集群的目标数据库 `example_db` 中创建表 `tbl_expr_test`。目标表 `tbl_expr_test` 包含两列，其中 `col2` 列的值需要通过使用 CASE 表达式在 JSON 数据上进行计算。

```SQL
CREATE TABLE tbl_expr_test (
    col1 string, col2 string)
DISTRIBUTED BY HASH (col1);
```

**例行加载作业**
由于目标表中 `col2` 列的值是使用 CASE 表达式生成的，因此您需要在例程加载作业的 `COLUMNS` 参数中指定相应的表达式。

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

**查询 StarRocks 表**

查询 StarRocks 表。结果显示，`col2` 列中的值是 CASE 表达式的输出。

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

#### 指定要加载的 JSON 格式数据的根元素

您需要使用 `json_root` 来指定要加载的 JSON 格式数据的根元素，该值必须是有效的 JsonPath 表达式。

**准备数据集**

例如，Kafka 集群的 Topic 中存在以下 JSON 格式的数据 `ordertest3` 。要加载的 JSON 格式数据的根元素是 `$.RECORDS`。

```SQL
{"RECORDS":[{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US","pay_time": 1589191487,"price": 875},{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK","pay_time": 1589191487,"price": 895},{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry","country": "France","pay_time": 1589191487,"price": 895}]}
```

**目标数据库和表**

在 StarRocks 集群的 `example_db` 数据库中创建名为 `example_tbl3` 的表。

```SQL
CREATE TABLE example_db.example_tbl3 ( 
    commodity_id varchar(26) NULL, 
    customer_name varchar(26) NULL, 
    country varchar(26) NULL, 
    pay_time bigint(20) NULL, 
    price double SUM NULL) 
AGGREGATE KEY(commodity_id,customer_name,country,pay_time) 
ENGINE=OLAP
DISTRIBUTED BY HASH(commodity_id); 
```

**例程加载作业**

您可以在 `PROPERTIES` 中设置 `"json_root" = "$.RECORDS"` 来指定要加载的 JSON 格式数据的根元素。此外，由于要加载的 JSON 格式数据采用数组结构，因此还必须设置 `"strip_outer_array" = "true"` 以剥离最外层的数组结构。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl3_ordertest3 ON example_tbl3
PROPERTIES
(
    "format" = "json",
    "json_root" = "$.RECORDS",
    "strip_outer_array" = "true"
 )
FROM KAFKA
(
    "kafka_broker_list" = "<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest2"
);
```

### 加载 Avro 格式的数据

从 v3.0.1 开始，StarRocks 支持使用 Routine Load 加载 Avro 数据。

#### Avro 架构很简单

假设 Avro 架构相对简单，您需要加载 Avro 数据的所有字段。

**准备数据集**

- **Avro 架构**

    1. 创建以下 Avro 架构文件 `avro_schema1.avsc`：

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

    2. 在[架构注册表](https://docs.confluent.io/platform/current/schema-registry/index.html)中注册 Avro 架构。

- **Avro 数据**

准备 Avro 数据并将其发送到 Kafka 主题 `topic_1`。

**目标数据库和表**

根据 Avro 数据的字段，在 StarRocks 集群的 `sensor` 目标数据库中创建名为 `sensor_log1` 的表。表的列名必须与 Avro 数据中的字段名匹配。有关 Avro 数据加载到 StarRocks 时的数据类型映射，请参见 [数据类型映射](#Data types mapping)。

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

**例程加载作业**

您可以使用简单模式进行例程加载作业。也就是说，在创建例程加载作业时，无需指定 `jsonpaths` 参数。执行以下语句，提交一个名为 `sensor_log_load_job1` 的例程加载作业，以消费 Kafka 主题 `topic_1` 中的 Avro 消息，并将数据加载到数据库 `sensor` 中的表 `sensor_log1` 中。

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
  "kafka_topic"= "topic_1",  
  "kafka_partitions" = "0,1,2,3,4,5",  
  "property.kafka_default_offsets" = "OFFSET_BEGINNING"  
);
```

#### Avro 架构包含嵌套的记录类型字段

假设 Avro schema 中包含一个嵌套的记录类型字段，您需要将嵌套记录类型字段中的子字段加载到 StarRocks 中。

**准备数据集**

- **Avro 架构**

    1. 创建以下 Avro 架构文件 `avro_schema2.avsc`。外部 Avro 记录包括五个字段，分别是 `id`、 `name`、 `checked`、 `sensor_type` 和 `data`。字段 `data` 包含一个嵌套记录 `data_record`。

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
                        "name": "data_record",
                        "fields" : [
                            {"name": "data_x", "type" : "boolean"},
                            {"name": "data_y", "type": "long"}
                        ]
                    }
                }
            ]
        }
        ```

    2. 在[架构注册表](https://docs.confluent.io/platform/current/schema-registry/index.html)中注册 Avro 架构。

- **Avro 数据**

准备 Avro 数据并将其发送到 Kafka 主题 `topic_2`。

**目标数据库和表**

根据 Avro 数据的字段，在 StarRocks 集群的 `sensor` 目标数据库中创建名为 `sensor_log2` 的表。

假设除了加载外部 Record 的字段 `id`、 `name`、 `checked` 和 `sensor_type` 外，还需要加载嵌套 Record `data_record` 中的子字段 `data_y`。

```sql
CREATE TABLE sensor.sensor_log2 ( 
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

**例程加载作业**

提交加载作业，使用 `jsonpaths` 指定需要加载的 Avro 数据字段。请注意，对于嵌套 Record 中的子字段 `data_y`，您需要将其指定为 `"$.data.data_y"`。

```sql
CREATE ROUTINE LOAD sensor.sensor_log_load_job2 ON sensor_log2  
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

#### Avro 架构包含一个 Union 字段

**准备数据集**

假设 Avro schema 中包含一个 Union 字段，您需要将 Union 字段加载到 StarRocks 中。

- **Avro 架构**

    1. 创建以下 Avro 架构文件 `avro_schema3.avsc`。外部 Avro 记录包括五个字段，分别是 `id`、 `name`、 `checked`、 `sensor_type` 和 `data`。字段 `data` 是 Union 类型，包括两个元素和一个 `null` 嵌套记录 `data_record`。

        ```JSON
        {
            "type": "record",
            "name": "sensor_log",
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

    2. 在[架构注册表](https://docs.confluent.io/platform/current/schema-registry/index.html)中注册 Avro 架构。

- **Avro 数据**

准备 Avro 数据并将其发送到 Kafka 主题 `topic_3`。

**目标数据库和表**

根据 Avro 数据的字段，在 StarRocks 集群的 `sensor` 目标数据库中创建名为 `sensor_log3` 的表。

假设除了加载外部 Record 的字段 `id`、`name`、`checked` 和 `sensor_type`之外，您还需要加载 Union 类型字段 `data` 中的 `data_record` 元素的字段 `data_y`。

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

**常规加载作业**

提交加载作业，使用 `jsonpaths` 指定 Avro 数据中需要加载的字段。请注意，对于字段 `data_y`，您需要将其 `jsonpath` 指定为 `"$.data.data_y"`。

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

当 Union 类型字段 `data` 的值为 `null` 时，加载到 StarRocks 表中的 `data_y` 列的值为 `null`。当 Union 类型字段 `data` 的值为数据记录时，加载到 `data_y` 列中的值为 Long 类型。