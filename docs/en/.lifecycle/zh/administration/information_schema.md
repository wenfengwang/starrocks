---
displayed_sidebar: English
---

# 信息模式

StarRocks 的 `information_schema` 是每个 StarRocks 实例内的数据库。`information_schema` 包含几个系统定义的只读表，这些表存储了 StarRocks 实例维护的所有对象的大量元数据信息。

从 v3.2 开始，StarRocks 支持通过 `information_schema` 查看外部目录元数据。

## 通过信息模式查看元数据

您可以通过查询 `information_schema` 中的表内容来查看 StarRocks 实例内的元数据信息。

以下示例通过查询 `tables` 表来查看 StarRocks 中名为 `sr_member` 的表的元数据信息。

```Plain
mysql> SELECT * FROM information_schema.tables WHERE TABLE_NAME like 'sr_member'\G
*************************** 1. row ***************************
  TABLE_CATALOG: def
   TABLE_SCHEMA: sr_hub
     TABLE_NAME: sr_member
     TABLE_TYPE: BASE TABLE
         ENGINE: StarRocks
        VERSION: NULL
     ROW_FORMAT: NULL
     TABLE_ROWS: 6
 AVG_ROW_LENGTH: 542
    DATA_LENGTH: 3255
MAX_DATA_LENGTH: NULL
   INDEX_LENGTH: NULL
      DATA_FREE: NULL
 AUTO_INCREMENT: NULL
    CREATE_TIME: 2022-11-17 14:32:30
    UPDATE_TIME: 2022-11-17 14:32:55
     CHECK_TIME: NULL
TABLE_COLLATION: utf8_general_ci
       CHECKSUM: NULL
 CREATE_OPTIONS: NULL
  TABLE_COMMENT: OLAP
1 row in set (1.04 sec)
```

## 信息模式表

StarRocks 优化了 `tables`、`tables_config` 和 `load_tracking_logs` 表提供的元数据信息，并从 v3.1 开始在 `information_schema` 中提供了 `loads` 表：

|**信息模式表名称**|**描述**|
|---|---|
|[tables](#tables)|提供表的一般元数据信息。|
|[tables_config](#tables_config)|提供 StarRocks 独有的附加表元数据信息。|
|[load_tracking_logs](#load_tracking_logs)|提供加载作业的错误信息（如果有）。|
|[loads](#loads)|提供加载作业的结果。该表从 v3.1 开始支持。目前，您只能从此表中查看 [Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 和 [Insert](../sql-reference/sql-statements/data-manipulation/INSERT.md) 作业的结果。|

### loads

`loads` 表提供了以下字段：

|**字段**|**描述**|
|---|---|
|JOB_ID|StarRocks 分配的用于标识加载作业的唯一 ID。|
|LABEL|加载作业的标签。|
|DATABASE_NAME|目标 StarRocks 表所属的数据库名称。|
|STATE|加载作业的状态。有效值：<ul><li>`PENDING`：加载作业已创建。</li><li>`QUEUEING`：加载作业在队列中等待调度。</li><li>`LOADING`：加载作业正在运行。</li><li>`PREPARED`：事务已提交。</li><li>`FINISHED`：加载作业成功。</li><li>`CANCELLED`：加载作业失败。</li></ul>有关详细信息，请参阅[异步加载](../loading/Loading_intro.md#asynchronous-loading)。|
|PROGRESS|加载作业的 ETL 阶段和 LOADING 阶段的进度。|
|TYPE|加载作业的类型。对于 Broker Load，返回值为 `BROKER`。对于 INSERT，返回值为 `INSERT`。|
|PRIORITY|加载作业的优先级。有效值：`HIGHEST`、`HIGH`、`NORMAL`、`LOW` 和 `LOWEST`。|
|SCAN_ROWS|扫描的数据行数。|
|FILTERED_ROWS|由于数据质量不足而被过滤掉的数据行数。|
|UNSELECTED_ROWS|由于 WHERE 子句中指定的条件而过滤掉的数据行数。|
|SINK_ROWS|加载的数据行数。|
|ETL_INFO|加载作业的 ETL 详细信息。仅对于 Spark Load 返回非空值。对于其他类型的加载作业，返回空值。|
|TASK_INFO|加载作业的任务执行详细信息，例如 `timeout` 和 `max_filter_ratio` 设置。|
|CREATE_TIME|创建加载作业的时间。格式：`yyyy-MM-dd HH:mm:ss`。示例：`2023-07-24 14:58:58`。|
|ETL_START_TIME|加载作业的 ETL 阶段的开始时间。格式：`yyyy-MM-dd HH:mm:ss`。示例：`2023-07-24 14:58:58`。|
|ETL_FINISH_TIME|加载作业的 ETL 阶段的结束时间。格式：`yyyy-MM-dd HH:mm:ss`。示例：`2023-07-24 14:58:58`。|
|LOAD_START_TIME|加载作业的 LOADING 阶段的开始时间。格式：`yyyy-MM-dd HH:mm:ss`。示例：`2023-07-24 14:58:58`。|
|LOAD_FINISH_TIME|加载作业的 LOADING 阶段的结束时间。格式：`yyyy-MM-dd HH:mm:ss`。示例：`2023-07-24 14:58:58`。|
|JOB_DETAILS|加载数据的详细信息，例如字节数和文件数。|
|ERROR_MSG|加载作业的错误消息。如果加载作业没有遇到任何错误，则返回 `NULL`。|
|TRACKING_URL|您可以访问加载作业中检测到的不合格数据行样本的 URL。您可以使用 `curl` 或 `wget` 命令访问 URL 并获取不合格的数据行样本。如果没有检测到不合格的数据，则返回 `NULL`。|
|TRACKING_SQL|可用于查询加载作业跟踪日志的 SQL 语句。仅当加载作业涉及不合格数据行时才会返回 SQL 语句。如果加载作业不涉及任何不合格的数据行，则返回 `NULL`。|
|REJECTED_RECORD_PATH|您可以访问加载作业中过滤掉的所有不合格数据行的路径。记录的不合格数据行数由加载作业中配置的 `log_rejected_record_num` 参数确定。您可以使用 `wget` 命令来访问该路径。如果加载作业不涉及任何不合格的数据行，则返回 `NULL`。|

### tables

`tables` 表提供了以下字段：

|**字段**|**描述**|
|---|---|
|TABLE_CATALOG|存储表的目录名称。|
|TABLE_SCHEMA|存储表的数据库名称。|
|TABLE_NAME|表的名称。|
|TABLE_TYPE|表的类型。有效值：“BASE TABLE” 或 “VIEW”。|
|ENGINE|表的引擎类型。有效值：“StarRocks”、“MySQL”、“MEMORY” 或空字符串。|
|VERSION|适用于 StarRocks 中不可用的功能。|
|ROW_FORMAT|适用于 StarRocks 中不可用的功能。|
|TABLE_ROWS|表的行数。|
|AVG_ROW_LENGTH|表的平均行长度（大小）。它相当于 `DATA_LENGTH` / `TABLE_ROWS`。单位：字节。|
|DATA_LENGTH|表的数据长度（大小）。单位：字节。|
|MAX_DATA_LENGTH|适用于 StarRocks 中不可用的功能。|
|INDEX_LENGTH|适用于 StarRocks 中不可用的功能。|
|DATA_FREE|适用于 StarRocks 中不可用的功能。|
|AUTO_INCREMENT|适用于 StarRocks 中不可用的功能。|
|CREATE_TIME|创建表的时间。|
|UPDATE_TIME|上次更新表的时间。|
|CHECK_TIME|最后一次对表执行一致性检查的时间。|
|TABLE_COLLATION|表的默认排序规则。|
|CHECKSUM|适用于 StarRocks 中不可用的功能。|
|CREATE_OPTIONS|适用于 StarRocks 中不可用的功能。|
|TABLE_COMMENT|表的注释。|

### tables_config

`tables_config` 表提供了以下字段：

|**字段**|**描述**|
|---|---|
|TABLE_SCHEMA|存储表的数据库名称。|
|TABLE_NAME|表的名称。|
|TABLE_ENGINE|表的引擎类型。|
|TABLE_MODEL|表类型。有效值：“DUP_KEYS”、“AGG_KEYS”、“UNQ_KEYS” 或 “PRI_KEYS”。|
|PRIMARY_KEY|主键表或唯一键表的主键。如果表不是主键表或唯一键表，则返回空字符串。|
|PARTITION_KEY|表的分区列。|
|DISTRIBUTE_KEY|表的分桶列。|
|DISTRIBUTE_TYPE|表的数据分布方式。|
|DISTRIBUTE_BUCKET|表中存储桶的数量。|
|SORT_KEY|表的排序键。|
|PROPERTIES|表的属性。|
|TABLE_ID|表的 ID。|

## load_tracking_logs

此功能自 StarRocks v3.0 起支持。

`load_tracking_logs` 表提供了以下字段：

|**字段**|**描述**|
|---|---|
|JOB_ID|加载作业的 ID。|
|LABEL|加载作业的标签。|
|DATABASE_NAME|加载作业所属的数据库。|
|TRACKING_LOG|加载作业的错误日志（如果有）。|
|Type|加载作业的类型。有效值：`BROKER`、`INSERT`、`ROUTINE_LOAD` 和 `STREAM_LOAD`。|

## materialized_views

`materialized_views` 表提供了以下字段：

|**字段**|**描述**|
|---|---|
|MATERIALIZED_VIEW_ID|物化视图的 ID|
|TABLE_SCHEMA|物化视图所在的数据库|
|TABLE_NAME|物化视图的名称|
|REFRESH_TYPE|物化视图的刷新类型，包括 `ROLLUP`、`ASYNC` 和 `MANUAL`|
|IS_ACTIVE|指示物化视图是否处于活动状态。不活动的物化视图无法刷新或查询。|
|INACTIVE_REASON|物化视图不活动的原因|
|PARTITION_TYPE|物化视图的分区策略类型|
|TASK_ID|负责刷新物化视图的任务 ID|
|TASK_NAME|负责刷新物化视图的任务名称|
|LAST_REFRESH_START_TIME|最近一次刷新任务的开始时间|
|LAST_REFRESH_FINISHED_TIME|最近一次刷新任务的结束时间|
|LAST_REFRESH_DURATION|最近刷新任务的持续时间|
|LAST_REFRESH_STATE|最近刷新任务的状态|
|LAST_REFRESH_FORCE_REFRESH|指示最近的刷新任务是否为强制刷新|
|LAST_REFRESH_START_PARTITION|最近刷新任务的起始分区|
|LAST_REFRESH_END_PARTITION|最近刷新任务的结束分区|
|LAST_REFRESH_BASE_REFRESH_PARTITIONS|最近一次刷新任务涉及的基表分区|
|LAST_REFRESH_MV_REFRESH_PARTITIONS|在最近的刷新任务中刷新的物化视图分区|
|LAST_REFRESH_ERROR_CODE|最近一次刷新任务的错误代码|
|LAST_REFRESH_ERROR_MESSAGE|最近一次刷新任务的错误信息|
|TABLE_ROWS|物化视图中的数据行数，基于近似的后台统计|
|MATERIALIZED_VIEW_DEFINITION|物化视图的 SQL 定义|