---
displayed_sidebar: "Chinese"
---

# 严格模式

严格模式是您可以为数据加载配置的可选属性。它影响加载行为和最终加载的数据。

本主题介绍了严格模式是什么以及如何设置严格模式。

## 理解严格模式

在数据加载过程中，源列的数据类型可能与目标列的数据类型不完全一致。在这种情况下，StarRocks会对数据类型不一致的源列值进行转换。由于各种问题（例如不匹配的字段数据类型和字段长度溢出）导致数据转换失败。未能成功转换的源列值是不合格列值，包含不合格列值的源行被称为“不合格行”。严格模式用于控制在数据加载过程中是否过滤掉不合格行。

严格模式的工作方式如下：

- 如果启用了严格模式，StarRocks仅加载合格行。它会过滤掉不合格行并返回有关不合格行的详细信息。
- 如果禁用了严格模式，StarRocks将不合格列值转换为 `NULL`，并加载包含这些 `NULL` 值的不合格行以及合格行。

请注意以下几点：

- 在实际业务场景中，合格行和不合格行都可能包含 `NULL` 值。如果目标列不允许 `NULL` 值，StarRocks会报告错误并过滤掉包含 `NULL` 值的行。

- 对于 [Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) 或 [Spark Load](../../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) 作业，可以通过可选的作业属性 `max_filter_ratio` 控制可过滤掉的不合格行的最大百分比。[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md) 不支持设置 `max_filter_ratio` 属性。

例如，您要从 CSV 格式的数据文件中加载四行数据，其中一个列包含 `\N`（`\N` 表示 `NULL` 值）、`abc`、`2000` 和 `1` 的值，加载到 StarRocks 表中，并且目标 StarRocks 表列的数据类型是 TINYINT [-128, 127]。

- 源列值 `\N` 在转换为 TINYINT 时被处理为 `NULL`。

  > **注意**
  >
  > 无论目标数据类型如何，`\N`在转换时始终被处理为 `NULL`。

- 源列值 `abc` 被处理为 `NULL`，因为它的数据类型不是 TINYINT，转换失败。

- 源列值 `2000` 被处理为 `NULL`，因为它超出了 TINYINT 支持的范围，转换失败。

- 源列值 `1` 可以被正确转换为 TINYINT 类型的值 `1`。

如果禁用了严格模式，StarRocks将加载所有四行数据。

如果启用了严格模式，StarRocks仅加载包含 `\N` 或 `1` 的行，并过滤掉包含 `abc` 或 `2000` 的行。被过滤掉的行会计入由 `max_filter_ratio` 参数指定的不合格数据质量导致的最大可过滤掉的行百分比。

### 禁用严格模式的最终加载数据

| 源列值 | 转换为 TINYINT 后的列值 | 目标列允许 NULL 值的加载结果 | 目标列不允许 NULL 值的加载结果 |
| ------- | -------------------------- | ----------------------------- | ------------------------- |
| \N      | NULL                       | 加载值 `NULL`。              | 报告错误。                |
| abc     | NULL                       | 加载值 `NULL`。              | 报告错误。                |
| 2000    | NULL                       | 加载值 `NULL`。              | 报告错误。                |
| 1       | 1                          | 加载值 `1`。                 | 加载值 `1`。                |

### 启用严格模式的最终加载数据

| 源列值 | 转换为 TINYINT 后的列值 | 目标列允许 NULL 值的加载结果 | 目标列不允许 NULL 值的加载结果 |
| ------- | -------------------------- | ----------------------------- | ------------------------- |
| \N      | NULL                       | 加载值 `NULL`。              | 报告错误。                |
| abc     | NULL                       | 不允许加载值 `NULL`，因此被过滤掉。 | 报告错误。                |
| 2000    | NULL                       | 不允许加载值 `NULL`，因此被过滤掉。 | 报告错误。                |
| 1       | 1                          | 加载值 `1`。                 | 加载值 `1`。                |

## 设置严格模式

如果要运行 [Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) 或 [Spark Load](../../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) 作业加载数据，可以使用 `strict_mode` 参数为加载作业设置严格模式。有效值为 `true` 和 `false`。默认值为 `false`。值 `true` 启用严格模式，值 `false` 禁用严格模式。

如果要执行 [INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md) 加载数据，可以使用 `enable_insert_strict` 会话变量为设置严格模式。有效值为 `true` 和 `false`。默认值为 `true`。值 `true` 启用严格模式，值 `false` 禁用严格模式。

以下是示例：

### Stream Load

```Bash
curl --location-trusted -u <username>:<password> \
    -H "strict_mode: {true | false}" \
    -T <file_name> -XPUT \
    http://<fe_host>:<fe_http_port>/api/<database_name>/<table_name>/_stream_load
```

有关 Stream Load 的详细语法和参数，请参见 [STREAM LOAD](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。

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

上述代码片段使用 HDFS 作为示例。有关 Broker Load 的详细语法和参数，请参见 [BROKER LOAD](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

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

上述代码片段使用 Apache Kafka® 作为示例。有关 Routine Load 的详细语法和参数，请参见 [CREATE ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。

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

上述代码片段使用 HDFS 作为示例。有关 Spark Load 的详细语法和参数，请参见 [SPARK LOAD](../../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)。

### INSERT

```SQL
SET enable_insert_strict = {true | false};
INSERT INTO <table_name> ...
```

有关 INSERT 的详细语法和参数，请参见 [INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)。