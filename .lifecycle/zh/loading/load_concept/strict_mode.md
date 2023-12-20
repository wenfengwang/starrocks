---
displayed_sidebar: English
---

# 严格模式

严格模式是您可以配置以用于数据加载的一个可选属性。它会影响加载行为以及最终加载的数据。

本主题将介绍什么是严格模式以及如何设置严格模式。

## 理解严格模式

在数据加载过程中，源列的数据类型可能与目标列的数据类型不完全一致。在这种情况下，StarRocks 对数据类型不一致的源列值进行转换。由于字段数据类型不匹配、字段长度溢出等问题，数据转换可能失败。未能正确转换的源列值称为不合格列值，包含不合格列值的源行称为“不合格行”。严格模式用于控制在数据加载过程中是否过滤掉这些不合格行。

严格模式的工作方式如下：

- 如果启用严格模式，StarRocks 只会加载合格的行。它会过滤掉不合格的行，并提供有关不合格行的详细信息。
- 如果禁用严格模式，StarRocks 会将不合格的列值转换成 NULL，并将包含这些 NULL 值的不合格行与合格行一起加载。

请注意以下几点：

- 在实际业务场景中，合格行和不合格行都可能包含 NULL 值。如果目标列不允许 NULL 值，StarRocks 将报告错误并过滤掉包含 NULL 值的行。

- 对于 [Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) 或 [Spark Load](../../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) 任务，可以过滤掉的不合格行的最大比例由可选的作业属性 `max_filter_ratio` 控制。[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md) 操作不支持设置 `max_filter_ratio` 属性。

例如，您想将一个 CSV 格式的数据文件中的四行数据，分别包含 \N（\N 代表 NULL 值）、abc、2000 和 1 这些值的列，加载到 StarRocks 表中，目标 StarRocks 表列的数据类型为 TINYINT [-128, 127]。

- 源列值 \N 转换为 TINYINT 类型时会被处理成 NULL。

    > **注意**
    > `\N` 在转换过程中总是会被处理成 `NULL`，无论目标数据类型为何。

- 源列值 abc 会被处理成 NULL，因为它的数据类型不是 TINYINT，转换失败。

- 源列值 2000 会被处理成 NULL，因为它超出了 TINYINT 支持的范围，转换失败。

- 源列值 1 可以正确地转换成 TINYINT 类型的值 1。

如果禁用严格模式，StarRocks 会加载所有四行。

如果启用严格模式，StarRocks 只会加载包含 \N 或 1 的行，并过滤掉包含 abc 或 2000 的行。被过滤掉的行将计入由 max_filter_ratio 参数指定的，由于数据质量不足而可被过滤掉的行的最大比例。

### 禁用严格模式时的最终加载数据

|源列值|转换为 TINYINT 时的列值|目标列允许 NULL 值时加载结果|目标列不允许 NULL 值时加载结果|
|---|---|---|---|
|\N|NULL|加载值 NULL。|报告错误。|
|abc|NULL|加载值 NULL。|报告错误。|
|2000|NULL|加载值NULL。|报告错误。|
|1|1|已加载值 1。|已加载值 1。|

### 启用严格模式时的最终加载数据

|源列值|转换为 TINYINT 时的列值|目标列允许 NULL 值时加载结果|目标列不允许 NULL 值时加载结果|
|---|---|---|---|
|\N|NULL|加载值 NULL。|报告错误。|
|abc|NULL|NULL 值不允许，因此被过滤掉。|报告错误。|
|2000|NULL|NULL 值不允许，因此被过滤掉。|报告错误。|
|1|1|已加载值 1。|已加载值 1。|

## 设置严格模式

如果您运行 [Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) 或 [Spark Load](../../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) 作业来加载数据，请使用 `strict_mode` 参数设置严格模式。有效值为 `true` 和 `false`。默认值为 `false`。值 `true` 表示启用严格模式，而值 `false` 表示禁用严格模式。

如果您执行 [INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md) 操作来加载数据，请使用 `enable_insert_strict` 会话变量来设置严格模式。有效值为 `true` 和 `false`。默认值为 `true`。值 `true` 表示启用严格模式，而值 `false` 表示禁用严格模式。

以下是一些示例：

### Stream Load

```Bash
curl --location-trusted -u <username>:<password> \
    -H "strict_mode: {true | false}" \
    -T <file_name> -XPUT \
    http://<fe_host>:<fe_http_port>/api/<database_name>/<table_name>/_stream_load
```

有关Stream Load的详细语法和参数，请参见[STREAM LOAD](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。

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

上述代码片段以 HDFS 为例。有关[BROKER LOAD](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)的详细语法和参数，请参见 BROKER LOAD。

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

上述代码片段以 Apache Kafka® 为例。有关 Routine Load 的详细语法和参数，请参见 [CREATE ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。

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

上述代码片段以 HDFS 为例。有关Spark Load的详细语法和参数，请参见[SPARK LOAD](../../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)。

### INSERT

```SQL
SET enable_insert_strict = {true | false};
INSERT INTO <table_name> ...
```

有关 INSERT 的详细语法和参数，请参见[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)。
