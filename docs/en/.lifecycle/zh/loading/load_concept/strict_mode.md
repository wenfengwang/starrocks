---
displayed_sidebar: English
---

# 严格模式

严格模式是您可以为数据加载配置的可选属性。它会影响加载行为和最终加载的数据。

本主题介绍了严格模式是什么，以及如何设置严格模式。

## 了解严格模式

在数据加载过程中，源列的数据类型可能与目标列的数据类型不完全一致。在这种情况下，StarRocks会对数据类型不一致的源列值进行转换。由于各种问题，例如字段数据类型不匹配和字段长度溢出，数据转换可能会失败。无法正确转换的源列值为非限定列值，包含非限定列值的源行称为"非限定行"。严格模式用于控制在数据加载过程中是否过滤掉不合格的行。

严格模式的工作原理如下：

- 如果启用了严格模式，StarRocks将只加载符合条件的行。它会筛选掉不合格的行，并返回有关不合格行的详细信息。
- 如果禁用严格模式，StarRocks会将不合格的列值转换为`NULL`，并将包含这些`NULL`值的不合格行与合格行一起加载。

请注意以下几点：

- 在实际业务场景中，合格行和不合格行都可能包含`NULL`值。如果目标列不允许`NULL`值，StarRocks会报错并过滤掉包含`NULL`值的行。

- 可以通过可选作业属性`max_filter_ratio`来控制[Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)或[Spark Load](../../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)作业中可以筛选掉的不合格行的最大百分比。[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)不支持设置`max_filter_ratio`属性。

例如，您希望将 CSV 格式的数据文件中的四行数据加载到StarRocks表中，这四行分别包含`\N`（`\N`表示`NULL`值）、`abc`、`2000`和`1`。目标StarRocks表列的数据类型为TINYINT [-128, 127]。

- 源列值`\N`在转换为TINYINT时被处理成`NULL`。

  > **注意**
  >
  > 无论目标数据类型如何，`\N`在转换时始终被处理成`NULL`。

- 源列值`abc`被处理成`NULL`，因为它的数据类型不是TINYINT，转换失败。

- 源列值`2000`被处理成`NULL`，因为它超出了TINYINT支持的范围，转换失败。

- 源列值`1`可以正确转换为TINYINT类型的值`1`。

如果禁用严格模式，StarRocks会加载所有四行。

如果启用了严格模式，StarRocks将只加载包含`\N`或`1`的行，并筛选掉包含`abc`或`2000`的行。筛选出的行将计入由`max_filter_ratio`参数指定的数据质量不足而可以筛选出的行的最大百分比。

### 禁用严格模式的最终加载数据

| 源列值 | 转换为TINYINT时的列值 | 当目标列允许NULL值时加载结果 | 当目标列不允许NULL值时加载结果 |
| ------- | ----------------------- | ---------------------------- | ------------------------------ |
| \N      | NULL                   | 加载值`NULL`                | 报错                           |
| abc     | NULL                   | 加载值`NULL`                | 报错                           |
| 2000    | NULL                   | 加载值`NULL`                | 报错                           |
| 1       | 1                      | 加载值`1`                   | 加载值`1`                      |

### 启用严格模式的最终加载数据

| 源列值 | 转换为TINYINT时的列值 | 当目标列允许NULL值时加载结果 | 当目标列不允许NULL值时加载结果 |
| ------- | ----------------------- | ---------------------------- | ------------------------------ |
| \N      | NULL                   | 加载值`NULL`                | 报错                           |
| abc     | NULL                   | 该值`NULL`是不允许的，因此被过滤掉 | 报错                           |
| 2000    | NULL                   | 该值`NULL`是不允许的，因此被过滤掉 | 报错                           |
| 1       | 1                      | 加载值`1`                   | 加载值`1`                      |

## 设置严格模式

如果要运行[Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)或[Spark Load](../../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)作业来加载数据，请使用`strict_mode`参数为加载作业设置严格模式。有效值为`true`和`false`。默认值为`false`。值`true`启用严格模式，值`false`禁用严格模式。

如果要执行[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)加载数据，请使用`enable_insert_strict`会话变量设置严格模式。有效值为`true`和`false`。默认值为`true`。值`true`启用严格模式，值`false`禁用严格模式。

以下是示例：

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

上述代码片段以HDFS为例。有关Broker Load的详细语法和参数，请参见[BROKER LOAD](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

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

上述代码片段以Apache Kafka®为例。有关Routine Load的详细语法和参数，请参见[CREATE ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。

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

上述代码片段以HDFS为例。有关Spark Load的详细语法和参数，请参见[SPARK LOAD](../../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)。

### INSERT

```SQL
SET enable_insert_strict = {true | false};
INSERT INTO <table_name> ...
```

有关INSERT的详细语法和参数，请参见[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)。
