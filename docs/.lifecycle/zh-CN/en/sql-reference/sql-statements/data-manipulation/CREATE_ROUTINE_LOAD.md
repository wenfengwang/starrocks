---
displayed_sidebar: "Chinese"
---

# 创建 ROUTINE LOAD

## 描述

Routine Load可以持续消费来自Apache Kafka®的消息，并将数据加载到StarRocks中。Routine Load可以从Kafka集群中消费CSV、JSON和Avro（自v3.0.1起支持）数据，并通过多种安全协议访问Kafka，包括 `plaintext`、`ssl`、`sasl_plaintext`和`sasl_ssl`。

本主题描述了CREATE ROUTINE LOAD语句的语法、参数和示例。

> **注意**
>
> - 有关Routine Load的应用场景、原理和基本操作的信息，请参见[从Apache Kafka®持续加载数据](../../../loading/RoutineLoad.md)。
> - 您只能以在目标StarRocks表具有INSERT权限的用户身份加载数据到StarRocks表中。如果没有INSERT权限，您可以按照提供的[GRANT](../account-management/GRANT.md)说明为您用于连接到StarRocks集群的用户授予INSERT权限。

## 语法

```SQL
CREATE ROUTINE LOAD <database_name>.<job_name> ON <table_name>
[load_properties]
[job_properties]
FROM data_source
[data_source_properties]
```

## 参数

### `database_name`、`job_name`、`table_name`

`database_name`

可选。StarRocks数据库的名称。

`job_name`

必需。Routine Load作业的名称。一个表可以从多个Routine Load作业接收数据。建议您使用可识别的信息设置有意义的Routine Load作业名称，例如Kafka主题名称和大致作业创建时间，以区分多个Routine Load作业。Routine Load作业的名称必须在同一数据库中是唯一的。

`表名`

必需。要加载数据的StarRocks表的名称。

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

CSV格式数据的列分隔符。默认列分隔符为 `\t`（制表符）。例如，您可以使用 `COLUMNS TERMINATED BY ","` 来指定逗号作为列分隔符。

> **注意**
>
> - 确保此处指定的列分隔符与要导入的数据中的列分隔符相同。
> - 您可以使用UTF-8字符串，如逗号(,)、制表符或管道(|)等长度不超过50字节的文本分隔符。
> - 空值以 `\N` 表示。例如，数据记录由三列组成，数据记录在第一列和第三列中保存数据，但第二列中不保存数据。在这种情况下，您需要在第二列使用 `\N` 来表示空值。这意味着记录必须编译为 `a,\N,b` 而不是 `a,,b`。`a,,b` 表示记录的第二列保存空字符串。

`ROWS TERMINATED BY`

CSV格式数据的行分隔符。默认行分隔符为 `\n`。

`COLUMNS`

源数据列与StarRocks表列之间的映射。有关更多信息，请参见本主题中的[列映射](#列映射)。

- `column_name`：如果源数据的某一列可以直接映射到StarRocks表的列而无需任何计算，则只需指定列名。这些列可以称为映射列。
- `column_assignment`：如果源数据的某一列无法直接映射到StarRocks表的列，并且在加载数据之前必须使用函数计算列的值，则必须在`expr`中指定计算函数。这些列可以称为派生列。
  建议在映射列之后放置派生列，因为StarRocks首先解析映射列。

`WHERE`

过滤条件。只有符合过滤条件的数据才能加载到StarRocks中。例如，如果只想要导入其 `col1` 值大于 `100` 且 `col2` 值等于 `1000` 的行，则可以使用`WHERE col1 > 100 and col2 = 1000`。

> **注意**
>
> 在过滤条件中指定的列可以是源列或派生列。

`PARTITION`

如果一个StarRocks表在分区 p0、p1、p2 和 p3 上分布，并且您只希望将数据加载到StarRocks中的 p1、p2 和 p3，并过滤掉将存储在 p0 中的数据，则可以指定 `PARTITION(p1, p2, p3)` 作为过滤条件。默认情况下，如果不指定此参数，数据将被加载到所有分区。示例：

```SQL
PARTITION (p1, p2, p3)
```

`TEMPORARY PARTITION`

要将数据加载到的[临时分区](../../../table_design/Temporary_partition.md)的名称。可以指定多个临时分区，必须用逗号(,)分隔。

### `job_properties`

必需。加载作业的属性。语法：

```SQL
PROPERTIES ("<key1>" = "<value1>"[, "<key2>" = "<value2>" ...])
```

| **属性**                   | **必需** | **描述**                                   |
| ------------------------- | -------- | ---------------------------------------- |
| desired_concurrent_number | 否       | 单个Routine Load作业的预期任务并行度。默认值：`3`。实际任务并行度由多个参数的最小值确定：`min(alive_be_number, partition_number, desired_concurrent_number, max_routine_load_task_concurrent_num)`。<ul><li>`alive_be_number`：活动BE节点的数量。</li><li>`partition_number`：需消费的分区数。</li><li>`desired_concurrent_number`：单个Routine Load 作业的预期任务并行度。默认值：`3`。</li><li>`max_routine_load_task_concurrent_num`：Routine Load作业的默认最大任务并行度，为`5`。请参见[FE 动态参数](../../../administration/Configuration.md#configure-fe-dynamic-parameters)。</li></ul>实际任务并行度的最大值由活动BE节点数或需消费的分区数决定。 |
| max_batch_interval        | 否       | 任务的调度间隔，即任务的执行频率。单位：秒。值范围：`5` ~ `60`。默认值：`10`。建议设置大于`10`的值。如果调度小于10秒，则由于加载频率过高会生成过多的tablet版本。 |
| max_batch_rows            | 否       | 该属性仅用于定义错误检测窗口。窗口是单个Routine Load任务消耗的数据行数。该值为`10 * max_batch_rows`。默认值为`10 * 200000 = 200000`。Routine Load任务在错误检测窗口中检测错误数据。错误数据指StarRocks无法解析的数据，如无效的JSON格式数据。 |
| max_error_number          | 否       | 允许在错误检测窗口内的最大错误数据行数。如果超过此值，加载作业将暂停。您可以执行[SHOW ROUTINE LOAD](./SHOW_ROUTINE_LOAD.md)并使用`ErrorLogUrls`查看错误日志。然后，您可以根据错误日志在Kafka中纠正错误。默认值为`0`，表示不允许错误行。<br />**注意** <br />错误数据行不包括由WHERE子句过滤掉的数据行。 |
| strict_mode               | 否       | 指定是否启用[严格模式](../../../loading/load_concept/strict_mode.md)。有效值：`true` 和 `false`。默认值：`false`。启用严格模式时，如果加载数据中的某一列值为`NULL`，但目标表不允许该列的`NULL`值，则数据行将被过滤掉。 |
| log_rejected_record_num | 否 | 指定将不合格数据行记录的最大数量。该参数从v3.1版本开始支持。有效值：`0`、`-1`和任何非零正整数。默认值：`0`。<ul><li>值`0`指定被过滤的数据行将不会被记录。</li><li>值`-1`指定所有被过滤的数据行都会被记录。</li><li>诸如`n`之类的非零正整数指定每个BE最多可记录`n`个被过滤的数据行。</li></ul> |

| 时区                      | 否           | 加载作业使用的时区。默认值：`Asia/Shanghai`。该参数的值会影响函数（如strftime()、alignment_timestamp() 和 from_unixtime()）返回的结果。由此参数指定的时区为会话级时区。有关更多信息，请参见[设置时区](../../../administration/timezone.md)。 |
| merge_condition           | 否           | 指定要用作条件以确定是否更新数据的列的名称。仅当要加载到此列的数据值大于或等于此列的当前值时，才会更新数据。有关更多信息，请参见[通过加载更改数据](../../../loading/Load_to_Primary_Key_tables.md)。<br />**注意**<br />仅支持主键表进行条件更新。您指定的列不能是主键列。 |
| 格式                    | 否           | 要加载的数据的格式。有效值：`CSV`、`JSON` 和 `Avro`（自 v3.0.1 起支持）。默认值：`CSV`。 |
| trim_space                | 否           | 指定当数据文件为 CSV 格式时，是否删除数据文件中列分隔符之前和之后的空格。类型：BOOLEAN。默认值：`false`。<br />对于一些数据库，在将数据导出为 CSV 格式的数据文件时，会添加空格到列分隔符。这些空格根据它们的位置被称为前导空格或尾随空格。通过设置 `trim_space` 参数，您可以启用 StarRocks 删除这些不必要的空格。请注意，StarRocks 不会删除使用 `enclose` 指定字符包裹的字段内的空格（包括前导空格和尾随空格）。例如，以下字段值使用管道（<code class="language-text">&#124;</code>）作为列分隔符，双引号 (`"`) 作为 `enclose` 指定字符：<code class="language-text">&#124; "热爱 StarRocks" &#124;</code>。如果将 `trim_space` 设置为 `true`，则 StarRocks 会处理前述字段值为 <code class="language-text">&#124;"热爱 StarRocks"&#124;</code>。 |
| enclose                   | 否           | 指定数据文件中字段值根据 [RFC4180](https://www.rfc-editor.org/rfc/rfc4180) 使用的字符进行包装。类型：单字节字符。默认值：`NONE`。最常见的字符为单引号（`'`）和双引号（`"`）。<br />使用 `enclose` 指定字符包裹的所有特殊字符（包括行分隔符和列分隔符）都被视为普通符号。与 RFC4180 不同的是，StarRocks 允许您指定任何单字节字符作为 `enclose` 指定字符。<br />如果字段值包含 `enclose` 指定字符，则可以使用同一字符对该 `enclose` 指定字符进行转义。例如，您将 `enclose` 设置为 `"`，字段值为 `a "quoted" c`。在这种情况下，您可以在数据文件中输入字段值 `"`a ""quoted"" c"`。 |
| 转义                    | 否           | 指定用于转义各种特殊字符的字符，例如行分隔符、列分隔符、转义字符和 `enclose` 指定字符。类型：单字节字符。默认值：`NONE`。最常见的字符为反斜杠（`\`），在 SQL 语句中必须写为双斜线（`\\`）。<br />**注意**<br />`escape` 指定的字符适用于每对 `enclose` 指定字符内外。<br />以下是两个示例：<br /><ul><li>当您将 `enclose` 设置为 `"`，`escape` 设置为 `\` 时，StarRocks 将 `"say \"Hello world\""` 解析为 `say "Hello world"`。</li><li>假设列分隔符为逗号（`,`）。当您将 `escape` 设置为 `\` 时，StarRocks 将 `a, b\, c` 解析为两个独立的字段值：`a` 和 `b, c`。</li></ul> |
| strip_outer_array         | 否           | 指定是否去掉 JSON 格式数据的最外层数组结构。有效值：`true` 和 `false`。默认值：`false`。在实际业务场景中，JSON 格式数据可能具有以一对方括号 `[]` 表示的最外层数组结构。在此情况下，建议您将此参数设置为 `true`，以便 StarRocks 去掉最外层方括号 `[]` 并将每个内部数组加载为单独的数据记录。如果将此参数设置为 `false`，StarRocks 将解析整个 JSON 格式数据为一个数组，并将该数组作为单个数据记录加载。以 JSON 格式数据 `[{"category" : 1, "author" : 2}, {"category" : 3, "author" : 4} ]` 为例。如果将此参数设置为 `true`，则 `{"category" : 1, "author" : 2}` 和 `{"category" : 3, "author" : 4}` 将被解析为两个单独的数据记录，并加载到两行 StarRocks 数据中。 |
| jsonpaths                 | 否           | 您要从 JSON 格式数据中加载的字段名称。此参数的值是有效的 JsonPath 表达式。有关更多信息，请参见本主题中的 [StarRocks 表包含通过使用表达式生成的派生列的值](#starrocks-table-contains-derived-columns-whose-values-are-generated-by-using-expressions)。 |
| json_root                 | 否           | 要加载的 JSON 格式数据的根元素。StarRocks 通过 `json_root` 提取根节点的元素进行解析。默认情况下，此参数的值为空，表示将加载所有的 JSON 格式数据。有关更多信息，请参见本主题中的 [指定要加载的 JSON 格式数据的根元素](#specify-the-root-element-of-the-json-formatted-data-to-be-loaded)。 |
| task_consume_second | 否 | 每个指定例行加载作业中的每个例行加载任务消耗数据的最大时间。单位：秒。不同于 [FE 动态参数](../../../administration/Configuration.md) `routine_load_task_consume_second`（适用于集群内的所有例行加载作业），此参数专门用于单个例行加载作业，更为灵活。此参数自 v3.1.0 起得到支持。<ul> <li>当未配置 `task_consume_second` 和 `task_timeout_second` 时，StarRocks 使用 FE 动态参数 `routine_load_task_consume_second` 和 `routine_load_task_timeout_second` 来控制加载行为。</li> <li>当仅配置 `task_consume_second` 时，`task_timeout_second` 的默认值计算为 `task_consume_second` * 4。</li> <li>当仅配置 `task_timeout_second` 时，`task_consume_second` 的默认值计算为 `task_timeout_second`/4。</li> </ul> |
|task_timeout_second|否|指定例行加载作业中的每个例行加载任务的超时持续时间。单位：秒。不同于 [FE 动态参数](../../../administration/Configuration.md) `routine_load_task_timeout_second`（适用于集群内的所有例行加载作业），此参数专门用于单个例行加载作业，更为灵活。此参数自 v3.1.0 起得到支持。<ul> <li>当未配置 `task_consume_second` 和 `task_timeout_second` 时，StarRocks 使用 FE 动态参数 `routine_load_task_consume_second` 和 `routine_load_task_timeout_second` 来控制加载行为。</li> <li>当仅配置 `task_timeout_second` 时，`task_consume_second` 的默认值计算为 `task_timeout_second`/4。</li> <li>当仅配置 `task_consume_second` 时，`task_timeout_second` 的默认值计算为 `task_consume_second` * 4。</li> </ul>|

### `data_source`, `data_source_properties`

必需项。数据源及其相关属性。

```sql
FROM <data_source>
 ("<key1>" = "<value1>"[, "<key2>" = "<value2>" ...])
```

`data_source`

必需项。要加载的数据源。有效值：`KAFKA`。

`data_source_properties`

数据源的属性。

| 属性          | 必需 | 描述                                                  |
| ----------------- | -------- | ------------------------------------------------------------ |
| kafka_broker_list | 是      | Kafka 的经纪连接信息。格式为 `<kafka_broker_ip>:<broker_ port>`。多个经纪以逗号（,）分隔。Kafka 经纪的默认端口为 `9092`。示例：`"kafka_broker_list" = ""xxx.xx.xxx.xx:9092,xxx.xx.xxx.xx:9092"`. |
| kafka_topic       | 是      | 要消费的 Kafka 主题。例行加载作业只能从一个主题中消费消息。 |
| kafka_partitions  | 否  | 要消费的Kafka分区，例如，`"kafka_partitions" = "0, 1, 2, 3"`。如果未指定此属性，默认情况下将消费所有分区。 |
| kafka_offsets     | 否  | 从Kafka分区中指定的起始偏移量开始消费数据，如`kafka_partitions`中所述。如果未指定此属性，Routine Load作业将从`kafka_partitions`中的最新偏移量开始消费数据。有效值：<ul><li>特定偏移量：从特定偏移量开始消费数据。</li><li>`OFFSET_BEGINNING`：从可能的最早偏移量开始消费数据。</li><li>`OFFSET_END`：从最新偏移量开始消费数据。</li></ul> 多个起始偏移量由逗号（，）分隔，例如，`"kafka_offsets" = "1000, OFFSET_BEGINNING, OFFSET_END, 2000"`。|
| property.kafka_default_offsets| 否| 所有消费者分区的默认起始偏移量。此属性的支持值与`kafka_offsets`属性相同。|
| confluent.schema.registry.url|否 |注册Avro模式的Schema Registry的URL。StarRocks使用此URL检索Avro模式。格式如下：<br />`confluent.schema.registry.url = http[s]://[<schema-registry-api-key>:<schema-registry-api-secret>@]<hostname or ip address>[:<port>]`|

#### 更多与数据源相关的属性

您可以指定其他数据源（Kafka）相关的属性，这相当于使用Kafka命令行`--property`。有关支持的属性，请参见[Kafka消费者客户端的属性](https://github.com/confluentinc/librdkafka/blob/master/CONFIGURATION.md)。

> **注意**
>
> 如果属性值是文件名，请在文件名之前添加关键字`FILE:`。有关如何创建文件的信息，请参见[CREATE FILE](../Administration/CREATE_FILE.md)。

- **指定要消费的所有分区的默认初始偏移量**

```SQL
"property.kafka_default_offsets" = "OFFSET_BEGINNING"
```

- **指定由Routine Load作业使用的消费者组的ID**

```SQL
"property.group.id" = "group_id_0"
```

如果未指定`property.group.id`，StarRocks会基于Routine Load作业的名称生成一个随机值，格式为`{job_name}_{random uuid}`，例如`simple_job_0a64fe25-3983-44b2-a4d8-f52d3af4c3e8`。

- **指定BE用于访问Kafka的安全协议和相关参数**

  安全协议可以指定为`plaintext`（默认值）、`ssl`、`sasl_plaintext`或`sasl_ssl`。您需要根据指定的安全协议配置相关参数。

  当安全协议设置为`sasl_plaintext`或`sasl_ssl`时，支持以下SASL认证机制：

  - PLAIN
  - SCRAM-SHA-256 和SCRAM-SHA-512
  - OAUTHBEARER

  - 使用SSL安全协议访问Kafka：

    ```SQL
    -- 指定安全协议为SSL。
    "property.security.protocol" = "ssl"
    -- CA证书文件的路径。
    -- 如果Kafka服务器启用客户端身份验证，则还需要以下三个参数：
    -- 用于认证的客户端公钥的路径。
    "property.ssl.certificate.location" = "FILE:client.pem"
    -- 用于认证的客户端私钥的路径。
    "property.ssl.key.location" = "FILE:client.key"
    -- 客户端私钥的密码。
    "property.ssl.key.password" = "xxxxxx"
    ```

  - 使用SASL_PLAINTEXT安全协议和SASL/PLAIN认证机制访问Kafka：

    ```SQL
    -- 指定安全协议为SASL_PLAINTEXT
    "property.security.protocol" = "SASL_PLAINTEXT"
    -- 指定SASL机制为PLAIN，这是一种简单的用户名/密码认证机制
    "property.sasl.mechanism" = "PLAIN" 
    -- SASL用户名
    "property.sasl.username" = "admin"
    -- SASL密码
    "property.sasl.password" = "xxxxxx"
    ```

### FE和BE配置项

有关Routine Load相关的FE和BE配置项，请参见[配置项](../../../administration/Configuration.md)。

## 列映射

### 配置用于加载CSV格式数据的列映射

- 如果CSV格式数据的列可以按顺序一对一映射到StarRocks表的列，则无需配置数据和StarRocks表之间的列映射。

- 如果CSV格式数据的列不能按顺序一对一映射到StarRocks表的列，则需要使用`columns`参数配置数据文件和StarRocks表之间的列映射。这包括以下两种用例：

  - **列数相同但列顺序不同。此外，数据文件的数据在加载到匹配的StarRocks表列之前无需通过函数计算。**

    - 在`columns`参数中，您需要按与数据文件列排列方式相同的顺序指定StarRocks表列的名称。

    - 例如，StarRocks表由三列组成，这三列依次是`col1`、`col2`和`col3`，数据文件也由三列组成，可以依次映射到StarRocks表列`col3`、`col2`和`col1`，这种情况下，您需要指定`"columns: col3, col2, col1"`。

  - **列数不同且列顺序不同。此外，数据文件的数据在加载到匹配的StarRocks表列之前需要通过函数计算。**

    在`columns`参数中，您需要按照数据文件列的排列方式指定StarRocks表列的名称，并指定您要用于计算数据的函数。以下是两个示例：

    - StarRocks表由三列组成，这三列依次是`col1`、`col2`和`col3`，数据文件由四列组成，其中前三列可以按顺序映射到StarRocks表列`col1`、`col2`和`col3`，第四列无法映射到任何StarRocks表列。在这种情况下，您需要临时为数据文件的第四列指定一个名称，并且临时名称必须与任何StarRocks表列名称不同。例如，您可以指定`"columns: col1, col2, col3, temp"`，在这种情况下，数据文件的第四列临时命名为`temp`。


    - StarRocks表由三列组成，这三列依次是`year`、`month`和`day`，数据文件只包含一列，其中包含`yyyy-mm-dd hh:mm:ss`格式的日期和时间值。在这种情况下，您可以指定`"columns: col, year = year(col), month=month(col), day=day(col)"`，其中`col`是数据文件列的临时名称，函数`year = year(col)`、`month=month(col)`和`day=day(col)`用于从数据文件列`col`中提取数据并将数据加载到映射的StarRocks表列。例如，`year = year(col)`用于从数据文件列`col`中提取`yyyy`数据并将数据加载到StarRocks表列`year`。

有关更多示例，请参见[配置列映射](#configure-column-mapping)。

### 配置用于加载JSON格式或Avro格式数据的列映射

> **注意**

>

如果 JSON 格式数据的键与 StarRocks 表的列具有不同的名称，可以使用匹配模式加载 JSON 格式数据。在匹配模式下，需要使用 `jsonpaths` 和 `COLUMNS` 参数来指定 JSON 格式数据与 StarRocks 表之间的列映射关系：

- 在 `jsonpaths` 参数中，按照 JSON 格式数据中键的顺序指定 JSON 键。
- 在 `COLUMNS` 参数中，指定 JSON 键与 StarRocks 表列之间的映射关系:
  - 在 `COLUMNS` 参数中指定的列名按顺序与 JSON 格式数据一一对应。
  - 在 `COLUMNS` 参数中指定的列名按名称与 StarRocks 表列一一对应。

有关示例，请参见 [StarRocks table contains derived columns whose values are generated by using expressions](#starrocks-table-contains-derived-columns-whose-values-are-generated-by-using-expressions)。

## 示例

### 加载 CSV 格式数据

本节以 CSV 格式数据为例，描述您如何使用各种参数设置和组合满足各种加载需求。

**准备数据集**

假设您想从名为 `ordertest1` 的 Kafka 主题加载 CSV 格式数据。数据集中的每条消息都包括六列：订单 ID、付款日期、客户姓名、国籍、性别和价格。

```plaintext
2020050802,2020-05-08,Johann Georg Faust,Deutschland,male,895
2020050802,2020-05-08,Julien Sorel,France,male,893
2020050803,2020-05-08,Dorian Grey,UK,male,1262
2020050901,2020-05-09,Anna Karenina,Russia,female,175
2020051001,2020-05-10,Tess Durbeyfield,US,female,986
2020051101,2020-05-11,Edogawa Conan,japan,male,8924
```

**创建表**

根据 CSV 格式数据的列，创建名为 `example_tbl1` 的表，在数据库 `example_db` 中。

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

#### 从指定偏移量开始消费指定分区的数据

如果 Routine Load 任务需要从指定分区和偏移量开始消费数据，则需要配置参数 `kafka_partitions` 和 `kafka_offsets`。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1",
    "kafka_partitions" ="0,1,2,3,4", -- 要消费的分区
    "kafka_offsets" = "1000, OFFSET_BEGINNING, OFFSET_END, 2000" -- 对应的初始偏移量
);
```

#### 通过增加任务并行性来提高加载性能

为了提高加载性能并避免累积消耗，您可以在创建 Routine Load 任务时增加 `desired_concurrent_number` 的值来增加任务并行性。任务并行性允许将一个 Routine Load 任务拆分为尽可能多的并行任务。 

> **注意**
>
> 要了解提高加载性能的更多方法，请参阅 [Routine Load FAQ](../../../faq/loading/Routine_load_faq.md)。

请注意，实际任务并行性取决于以下多个参数中的最小值：

```SQL
min(alive_be_number, partition_number, desired_concurrent_number, max_routine_load_task_concurrent_num)
```

> **注意**
>
> 实际任务并行性的最大值是存活的 BE 节点数或要消费的分区数中的较小值。

因此，当存活的 BE 节点数和要消费的分区数大于其他两个参数 `max_routine_load_task_concurrent_num` 和 `desired_concurrent_number` 的值时，可以增加其他两个参数的值来增加实际任务并行性。

假设要消费的分区数为7，存活的 BE 节点数为5，`max_routine_load_task_concurrent_num` 是默认值 `5`。如果要增加实际任务并行性，可以将 `desired_concurrent_number` 设置为 `5`（默认值是 `3`）。在此情况下，实际任务并行性 `min(5,7,5,5)` 被配置为 `5`。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
"desired_concurrent_number" = "5" -- 将 desired_concurrent_number 的值设为 5
)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### 配置列映射

如果 CSV 格式数据中的列顺序与目标表中的列不一致，假设 CSV 格式数据中的第五列不需要导入到目标表中，则需要通过 `COLUMNS` 参数指定 CSV 格式数据与目标表之间的列映射关系。

**目标数据库和表**

根据 CSV 格式数据的列，在目标数据库 `example_db` 中创建目标表 `example_tbl2`。在此示例中，需要创建与 CSV 格式数据中五列对应的五个列，除了存储性别的第五列。

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

**Routine Load 任务**

在此示例中，由于 CSV 格式数据中的第五列不需要加载到目标表中，第五列在 `COLUMNS` 中临时命名为 `temp_gender`，而其他列则直接映射到表 `example_tbl2`。

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

#### 设置过滤条件

如果只想加载符合特定条件的数据，可以在 `WHERE` 子句中设置过滤条件，例如 `price > 100.`

```SQL
CREATE ROUTINE LOAD example_db.example_tbl2_ordertest1 ON example_tbl2
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price),
WHERE price > 100 -- 设置过滤条件
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### 开启严格模式以过滤掉含有 NULL 值的行

在 `PROPERTIES` 中，您可以设置 `"strict_mode" = "true"`，表示 Routine Load 任务处于严格模式。如果源列中有 `NULL` 值，但目标 StarRocks 表列不允许 NULL 值，那么包含源列中的 NULL 值的行将被过滤掉。

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
```
    "kafka_broker_list" = "<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### 设置错误容限

如果您的业务场景对不合格数据的容忍度较低，您需要通过配置参数 `max_batch_rows` 和 `max_error_number` 来设置错误检测窗口和错误数据行的最大数量。当错误检测窗口内的错误数据行数量超过 `max_error_number` 的值时，常规加载作业会暂停。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
"max_batch_rows" = "100000",-- max_batch_rows 的值乘以 10 等于错误检测窗口的大小。
"max_error_number" = "100" -- 错误检测窗口内允许的错误数据行的最大数量。
)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### 指定安全协议为 SSL 并配置相关参数

如果您需要将 BE 访问 Kafka 的安全协议指定为 SSL，您需要配置 `"property.security.protocol" = "ssl"` 和相关参数。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
-- 将安全协议指定为 SSL。
"property.security.protocol" = "ssl",
-- CA 证书的位置。
"property.ssl.ca.location" = "FILE:ca-cert",
-- 如果 Kafka 客户端启用了身份验证，您需要配置以下属性：
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

假设您希望从名为 `test_csv` 的 Kafka 主题加载 CSV 格式的数据。数据集中的每个消息包括六列：订单 ID、支付日期、客户名称、国籍、性别和价格。

```Plaintext
 "2020050802" , "2020-05-08" , "Johann Georg Faust" , "Deutschland" , "male" , "895"
 "2020050802" , "2020-05-08" , "Julien Sorel" , "France" , "male" , "893"
 "2020050803" , "2020-05-08" , "Dorian Grey\,Lord Henry" , "UK" , "male" , "1262"
 "2020050901" , "2020-05-09" , "Anna Karenina" , "Russia" , "female" , "175"
 "2020051001" , "2020-05-10" , "Tess Durbeyfield" , "US" , "female" , "986"
 "2020051101" , "2020-05-11" , "Edogawa Conan" , "japan" , "male" , "8924"
```

如果您希望将来自 Kafka 主题 `test_csv` 的所有数据加载到 `example_tbl1`，并且希望删除列分隔符前后的空格以及将 `enclose` 设置为 `"`，`escape` 设置为 `\`，请运行以下命令：

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

### 加载 JSON 格式数据

#### StarRocks 表列名与 JSON 键名一致

**准备数据集**

例如，在名为 `ordertest2` 的 Kafka 主题中存在以下 JSON 格式数据。

```SQL
{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US","pay_time": 1589191487,"price": 875}
{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK","pay_time": 1589191487,"price": 895}
{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry","country": "France","pay_time": 1589191487,"price": 895}
```

> **注意** 每个 JSON 对象必须位于一个 Kafka 消息中。否则，会出现未能解析 JSON 格式数据的错误。

**目标数据库和表**

在 StarRocks 集群的目标数据库 `example_db` 中创建表 `example_tbl3`。列名与 JSON 格式数据中的键名一致。

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

**常规加载作业**

您可以在创建常规加载作业时使用简单模式。也就是说，在创建常规加载作业时无需指定 `jsonpaths` 和 `COLUMNS` 参数。StarRocks 根据目标表 `example_tbl3` 的列名提取 Kafka 集群的主题 `ordertest2` 中 JSON 格式数据的键，并将其加载到目标表中。

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
> - 如果 JSON 格式数据的最外层是数组结构，则您需要在 `PROPERTIES` 中设置 `"strip_outer_array"="true"` 以剥离最外层的数组结构。此外，当您需要指定 `jsonpaths` 时，整个 JSON 格式数据的根元素是扁平化的 JSON 对象，因为已经剥离了 JSON 格式数据的最外层数组结构。
> - 您可以使用 `json_root` 指定 JSON 格式数据的根元素。

#### StarRocks 表包含由表达式生成其值的派生列

**准备数据集**

例如，在 Kafka 集群的主题 `ordertest2` 中存在以下 JSON 格式数据。

```SQL
{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US","pay_time": 1589191487,"price": 875}
{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK","pay_time": 1589191487,"price": 895}
{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry","country": "France","pay_time": 1589191487,"price": 895}
```

**目标数据库和表**

在 StarRocks 集群的数据库 `example_db` 中创建名为 `example_tbl4` 的表。列 `pay_dt` 是一个派生列，其值由计算 JSON 格式数据中 `pay_time` 键的值生成。

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

**常规加载作业**

您可以在常规加载作业中使用匹配模式。也就是说，当创建常规加载作业时，您需要指定`jsonpaths` 和 `COLUMNS` 参数。

在`jsonpaths` 参数中，您需要指定 JSON 格式数据的键，并按顺序排列这些键。

由于 JSON 格式数据中键 `pay_time` 的值在存储到 `example_tbl4` 表的 `pay_dt` 列之前需要转换为 DATE 类型，因此您需要在 `COLUMNS` 中使用 `pay_dt=from_unixtime(pay_time,'%Y%m%d')` 来指定计算方式。JSON 格式数据中其他键的值可以直接映射到 `example_tbl4` 表。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl4_ordertest2 TO example_tbl4
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
> - 如果 JSON 数据的最外层是数组结构，则您需要在 `PROPERTIES` 中设置 `"strip_outer_array"="true"` 以去除最外层的数组结构。此外，当您需要指定 `jsonpaths` 时，由于已去除 JSON 数据的最外层数组结构，整个 JSON 数据的根元素将是扁平化的 JSON 对象。
> - 您可以使用 `json_root` 来指定 JSON 格式数据的根元素。

#### StarRocks 表包含使用 CASE 表达式生成的派生列的情况

**准备数据集**

例如，Kafka 主题 `topic-expr-test` 中存在以下 JSON 格式数据。

```JSON
{"key1":1, "key2": 21}
{"key1":12, "key2": 22}
{"key1":13, "key2": 23}
{"key1":14, "key2": 24}
```

**目标数据库和表**

在 StarRocks 集群中的数据库 `example_db` 中创建名为 `tbl_expr_test` 的表。目标表 `tbl_expr_test` 包含两列，其中 `col2` 列的值需要通过对 JSON 数据使用 CASE 表达式进行计算。

```SQL
CREATE TABLE tbl_expr_test (
    col1 string, col2 string)
DISTRIBUTED BY HASH (col1);
```

**常规加载作业**

由于目标表中 `col2` 列的值是使用 CASE 表达式生成的，因此您需要在常规加载作业的 `COLUMNS` 参数中指定相应的表达式。

```SQL
CREATE ROUTINE LOAD rl_expr_test TO tbl_expr_test
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

查询 StarRocks 表。结果显示 `col2` 列的值是 CASE 表达式的输出。

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

您需要使用 `json_root` 来指定要加载的 JSON 格式数据的根元素，其值必须是有效的 JsonPath 表达式。

**准备数据集**

例如，Kafka 集群的主题 `ordertest3` 中存在以下 JSON 格式数据。要加载的 JSON 格式数据的根元素为 `$.RECORDS`。

```SQL
{"RECORDS":[{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US","pay_time": 1589191487,"price": 875},{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK","pay_time": 1589191487,"price": 895},{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry","country": "France","pay_time": 1589191487,"price": 895}]}
```

**目标数据库和表**

在 StarRocks 集群的数据库 `example_db` 中创建名为 `example_tbl3` 的表。

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

**常规加载作业**

您可以在 `PROPERTIES` 中设置 `"json_root" = "$.RECORDS"` 以指定要加载的 JSON 格式数据的根元素。此外，由于要加载的 JSON 格式数据是数组结构，您还必须设置 `"strip_outer_array" = "true"` 以去除最外层的数组结构。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl3_ordertest3 TO example_tbl3
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

### 加载 Avro 格式数据

自 v3.0.1 起，StarRocks 支持使用常规加载来加载 Avro 数据。

#### Avro 模式简单

假设 Avro 模式相对简单，并且您需要加载 Avro 数据的所有字段。

**准备数据集**

- **Avro 模式**

    1. 创建以下 Avro 模式文件 `avro_schema1.avsc`：

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

    2. 在 [模式注册表](https://docs.confluent.io/platform/current/schema-registry/index.html) 中注册 Avro 模式。

- **Avro 数据**

准备 Avro 数据并将其发送到 Kafka 主题 `topic_1`。

**目标数据库和表**

根据 Avro 数据的字段，创建一个名为 `sensor_log1` 的表，该表位于 StarRocks 集群的目标数据库 `sensor` 中。表中的列名必须与 Avro 数据中的字段名匹配。在将 Avro 数据加载到 StarRocks 中时，有关数据类型的映射，请参阅 [数据类型映射](#Data types mapping)。

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

**常规加载作业**

可以在【常规加载】作业中使用简单模式。也就是说，在创建【常规加载】作业时，无需指定参数 `jsonpaths`。执行以下语句提交一个名为 `sensor_log_load_job1` 的【常规加载】作业，用于消费 Kafka 主题 `topic_1` 中的 Avro 消息并将数据加载到数据库 `sensor` 中的表 `sensor_log1` 中。

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

#### Avro 模式包含嵌套的记录类型字段

假设 Avro 模式包含嵌套的记录类型字段，并且您需要将嵌套的记录类型字段中的子字段加载到 StarRocks 中。

**准备数据集**

- **Avro 模式**

    1. 创建以下 Avro 模式文件 `avro_schema2.avsc`。外部 Avro 记录按顺序包括五个字段：`id`、`name`、`checked`、`sensor_type` 和 `data`。字段 `data` 中包含一个嵌套记录 `data_record`。

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

    2. 在[模式注册表](https://docs.confluent.io/platform/current/schema-registry/index.html)中注册 Avro 模式。

- **Avro 数据**

准备 Avro 数据并将其发送到 Kafka 主题 `topic_2`。

**目标数据库和表**

根据 Avro 数据的字段，在 StarRocks 集群中的目标数据库 `sensor` 中创建表 `sensor_log2`。

假设除了加载外部记录的字段 `id`、`name`、`checked` 和 `sensor_type` 外，还需要加载 Union 类型字段 `data` 中的元素 `data_record` 的子字段 `data_y`。

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

**常规加载作业**

提交加载作业，使用 `jsonpaths` 指定需要加载的 Avro 数据字段。注意，对于嵌套记录中的子字段 `data_y`，需要将其 `jsonpath` 指定为 `"$.data.data_y"`。

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

#### Avro 模式包含 Union 字段

**准备数据集**

假设 Avro 模式包含 Union 字段，并且您需要将 Union 字段加载到 StarRocks 中。

- **Avro 模式**

    1. 创建以下 Avro 模式文件 `avro_schema3.avsc`。外部 Avro 记录按顺序包括五个字段：`id`、`name`、`checked`、`sensor_type` 和 `data`。字段 `data` 是 Union 类型的，包括两个元素，`null` 和一个嵌套记录 `data_record`。

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

    2. 在[模式注册表](https://docs.confluent.io/platform/current/schema-registry/index.html)中注册 Avro 模式。

- **Avro 数据**

准备 Avro 数据并将其发送到 Kafka 主题 `topic_3`。

**目标数据库和表**

根据 Avro 数据的字段，在 StarRocks 集群中的目标数据库 `sensor` 中创建表 `sensor_log3`。

假设除了加载外部记录的字段 `id`、`name`、`checked` 和 `sensor_type` 外，还需要加载 Union 类型字段 `data` 中的元素 `data_record` 的字段 `data_y`。

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

提交加载作业，使用 `jsonpaths` 指定需要加载的 Avro 数据字段。注意，对于字段 `data_y`，需要将其 `jsonpath` 指定为 `"$.data.data_y"`。

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

当 Union 类型字段 `data` 的值为 `null` 时，在 StarRocks 表的 column `data_y` 中加载的值为 `null`。当 Union 类型字段 `data` 的值为数据记录时，在 column `data_y` 中加载的值将为 Long 类型。