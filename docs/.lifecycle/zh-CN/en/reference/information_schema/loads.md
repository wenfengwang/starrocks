---
displayed_sidebar: "中文"
---

# 装载

`loads` 提供装载作业的结果。从 StarRocks v3.1 开始支持此视图。目前，您只能从此视图查看[Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)和[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)作业的结果。

`loads` 提供以下字段：

| **字段**              | **描述**                                                      |
| -------------------- | ------------------------------------------------------------ |
| JOB_ID               | StarRocks 分配的唯一 ID，用于标识装载作业。             |
| LABEL                | 装载作业的标签。                                             |
| DATABASE_NAME        | 目标 StarRocks 表所属数据库的名称。                         |
| STATE                | 装载作业的状态。有效值：<ul><li>`PENDING`：作业已创建。</li><li>`QUEUEING`：作业在队列中等待调度。</li><li>`LOADING`：作业正在运行。</li><li>`PREPARED`：事务已提交。</li><li>`FINISHED`：作业成功。</li><li>`CANCELLED`：作业失败。</li></ul>详情请参见[异步装载](../../loading/Loading_intro.md#asynchronous-loading)。 |
| PROGRESS             | 装载作业的 ETL 阶段和 LOADING 阶段的进度。                   |
| TYPE                 | 装载作业的类型。对于 Broker Load，返回值为 `BROKER`。对于 INSERT，返回值为 `INSERT`。     |
| PRIORITY             | 装载作业的优先级。有效值：`HIGHEST`、`HIGH`、`NORMAL`、`LOW`和`LOWEST`。  |
| SCAN_ROWS            | 扫描到的数据行数。                                           |
| FILTERED_ROWS        | 由于数据质量不足而被过滤掉的数据行数。                     |
| UNSELECTED_ROWS      | 由于 WHERE 子句中指定的条件而被过滤掉的数据行数。           |
| SINK_ROWS            | 装载的数据行数。                                             |
| ETL_INFO             | 装载作业的 ETL 详情。仅当为 Spark Load 时返回非空值。对于其他类型的装载作业，则返回空值。 |
| TASK_INFO            | 装载作业的任务执行详情，如 `timeout` 和 `max_filter_ratio` 设置。 |
| CREATE_TIME          | 装载作业的创建时间。格式：`yyyy-MM-dd HH:mm:ss`。示例：`2023-07-24 14:58:58`。 |
| ETL_START_TIME       | 装载作业的 ETL 阶段的开始时间。格式：`yyyy-MM-dd HH:mm:ss`。示例：`2023-07-24 14:58:58`。 |
| ETL_FINISH_TIME      | 装载作业的 ETL 阶段的结束时间。格式：`yyyy-MM-dd HH:mm:ss`。示例：`2023-07-24 14:58:58`。 |
| LOAD_START_TIME      | 装载作业的 LOADING 阶段的开始时间。格式：`yyyy-MM-dd HH:mm:ss`。示例：`2023-07-24 14:58:58`。 |
| LOAD_FINISH_TIME     | 装载作业的 LOADING 阶段的结束时间。格式：`yyyy-MM-dd HH:mm:ss`。示例：`2023-07-24 14:58:58`。 |
| JOB_DETAILS          | 装载的数据详情，如字节数量和文件数量。                      |
| ERROR_MSG            | 装载作业的错误消息。如果装载作业未遇到任何错误，则返回 `NULL`。 |
| TRACKING_URL         | 您可以访问到在装载作业中检测到的不合格数据行样本的 URL。您可以使用 `curl` 或 `wget` 命令访问此 URL，并获取不合格的数据行样本。如果没有检测到不合格数据，返回 `NULL`。 |
| TRACKING_SQL         | 用于查询装载作业跟踪日志的 SQL 语句。仅当装载作业涉及不合格数据行时返回 SQL 语句。如果装载作业不涉及任何不合格数据行，则返回 `NULL`。 |
| REJECTED_RECORD_PATH | 您可以获取到在装载作业中被过滤掉的所有不合格数据行的路径。记录的不合格数据行数量由装载作业中配置的 `log_rejected_record_num` 参数决定。您可以使用 `wget` 命令访问此路径。如果装载作业不涉及任何不合格数据行，则返回 `NULL`。 |