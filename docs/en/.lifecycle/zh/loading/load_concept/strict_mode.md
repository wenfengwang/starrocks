---
displayed_sidebar: English
---

# 严格模式

严格模式是一个可选属性，您可以为数据加载进行配置。它影响加载行为和最终加载的数据。

本节介绍什么是严格模式以及如何设置严格模式。

## 了解严格模式

在数据加载过程中，源列的数据类型可能与目标列的数据类型不完全一致。在这种情况下，StarRocks 对数据类型不一致的源列值执行转换。由于字段数据类型不匹配、字段长度溢出等各种问题，数据转换可能会失败。未能正确转换的源列值是不合格的列值，包含不合格列值的源行称为“不合格行”。严格模式用于控制数据加载时是否过滤掉不合格的行。

严格模式的工作原理如下：

- 如果启用严格模式，StarRocks 仅加载合格的行。它过滤掉不合格的行并返回有关不合格行的详细信息。
- 如果禁用严格模式，StarRocks 会将不合格的列值转换为 `NULL`，并将包含这些 `NULL` 值的不合格行与合格行一起加载。

请注意以下几点：

- 在实际业务场景中，合格行和不合格行都可能包含 `NULL` 值。如果目标列不允许 `NULL` 值，StarRocks 会报告错误并过滤掉包含 `NULL` 值的行。

- 可以为 [Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) 或 [Spark Load](../../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) 作业过滤掉的不合格行的最大百分比由可选作业属性 `max_filter_ratio` 控制。[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md) 不支持设置 `max_filter_ratio` 属性。

例如，您想要将 CSV 格式的数据文件中的四行分别包含 `\N`（`\N` 表示 `NULL` 值）、`abc`、`2000` 和 `1` 值的列加载到 StarRocks 表中，目标 StarRocks 表列的数据类型为 TINYINT [-128, 127]。

- 源列值 `\N` 在转换为 TINYINT 时被处理为 `NULL`。

    > **注意**
    > `\N` 在转换时始终被处理为 `NULL`，无论目标数据类型如何。

- 源列值 `abc` 被处理为 `NULL`，因为其数据类型不是 TINYINT，转换失败。

- 源列值 `2000` 被处理为 `NULL`，因为它超出了 TINYINT 支持的范围，转换失败。

- 源列值 `1` 可以正确转换为 TINYINT 类型值 `1`。

如果禁用严格模式，StarRocks 会加载所有四行。

如果启用严格模式，StarRocks 仅加载包含 `\N` 或 `1` 的行，并过滤掉包含 `abc` 或 `2000` 的行。过滤掉的行将根据 `max_filter_ratio` 参数指定的由于数据质量不足而可以过滤掉的行的最大百分比进行计数。

### 禁用严格模式的最终加载数据

|源列值|转换为 TINYINT 时的列值|目标列允许 NULL 值时的加载结果|目标列不允许 NULL 值时的加载结果|
|---|---|---|---|
|\N|NULL|加载值 `NULL`。|报告错误。|
|abc|NULL|加载值 `NULL`。|报告错误。|
|2000|NULL|加载值 `NULL`。|报告错误。|
|1|1|加载值 `1`。|加载值 `1`。|

### 启用严格模式后的最终加载数据

|源列值|转换为 TINYINT 时的列值|目标列允许 NULL 值时的加载结果|目标列不允许 NULL 值时的加载结果|
|---|---|---|---|
|\N|NULL|加载值 `NULL`。|报告错误。|
|abc|NULL|`NULL` 值不被允许，因此被过滤掉。|报告错误。|
|2000|NULL|`NULL` 值不被允许，因此被过滤掉。|报告错误。|
|1|1|加载值 `1`。|加载值 `1`。|

## 设置严格模式

如果您运行 [Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) 或 [Spark Load](../../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) 作业来加载数据，请使用 `strict_mode` 参数为加载作业设置严格模式。有效值为 `true` 和 `false`。默认值为 `false`。值 `true` 启用严格模式，值 `false` 禁用严格模式。

如果执行 [INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md) 来加载数据，请使用 `enable_insert_strict` 会话变量设置严格模式。有效值为 `true` 和 `false`。默认值是 `true`。值 `true` 启用严格模式，值 `false` 禁用严格模式。

示例如下：

### Stream Load

```Bash
curl --location-trusted -u <username>:<password> \
    -H "strict_mode: {true | false}" \
    -T <file_name> -XPUT \
    http://<fe_host>:<fe_http_port>/api/<database_name>/<table_name>/_stream_load
```

关于 Stream Load 的详细语法和参数，请参见 [STREAM LOAD](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。

### Broker Load

```SQL
LOAD LABEL [<database_name>.]<label_name>
(
    DATA INFILE ("<file_path>"[, "<file_path>" ...])
    INTO TABLE <table_name>
)
WITH BROKER
(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
)
PROPERTIES
(
    "strict_mode" = "{true | false}"
)
```

上述代码片段以 HDFS 为例。有关 Broker Load 的详细语法和参数，请参阅 [BROKER LOAD](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

### Routine Load

```SQL
CREATE ROUTINE LOAD [<database_name>.]<job_name> ON <table_name>
PROPERTIES
(
    "strict_mode" = "{true | false}"
) 
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>[,<kafka_broker2_ip>:<kafka_broker2_port>...]",
    "kafka_topic" = "<topic_name>"
)
```

上述代码片段以 Apache Kafka® 为例。有关 Routine Load 的详细语法和参数，请参阅 [CREATE ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。

### Spark Load

```SQL
LOAD LABEL [<database_name>.]<label_name>
(
    DATA INFILE ("<file_path>"[, "<file_path>" ...])
    INTO TABLE <table_name>
)
WITH RESOURCE <resource_name>
(
    "spark.executor.memory" = "3g",
    "broker.username" = "<hdfs_username>",
    "broker.password" = "<hdfs_password>"
)
PROPERTIES
(
    "strict_mode" = "{true | false}"   
)
```

上述代码片段以 HDFS 为例。有关 Spark Load 的详细语法和参数，请参阅 [SPARK LOAD](../../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)。

### INSERT

```SQL
SET enable_insert_strict = {true | false};
INSERT INTO <table_name> ...
```

关于 INSERT 的详细语法和参数，请参见 [INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)。