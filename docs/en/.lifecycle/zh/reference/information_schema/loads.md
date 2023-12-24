---
displayed_sidebar: English
---

# 负载

`loads` 提供加载作业的结果。该视图从 StarRocks v3.1 版本开始支持。目前，您只能从此视图查看 [Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 和 [INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md) 作业的结果。

`loads` 中提供了以下字段：

| **字段**            | **描述**                                              |
| -------------------- | ------------------------------------------------------------ |
| JOB_ID               | StarRocks 分配的唯一 ID，用于标识加载作业。 |
| LABEL                | 加载作业的标签。                                   |
| DATABASE_NAME        | 目标 StarRocks 表所属的数据库名称。 |
| STATE                | 加载作业的状态。有效值：<ul><li>`PENDING`：加载作业已创建。</li><li>`QUEUEING`：加载作业在队列中等待调度。</li><li>`LOADING`：加载作业正在运行。</li><li>`PREPARED`：事务已提交。</li><li>`FINISHED`：加载作业成功。</li><li>`CANCELLED`：加载作业失败。</li></ul>更多信息，请参阅 [异步加载](../../loading/Loading_intro.md#asynchronous-loading)。 |
| PROGRESS             | 加载作业的 ETL 阶段和 LOADING 阶段的进度。 |
| TYPE                 | 加载作业的类型。对于 Broker Load，返回值为 `BROKER`。对于 INSERT，返回值为 `INSERT`。 |
| PRIORITY             | 加载作业的优先级。有效值：`HIGHEST`、`HIGH`、`NORMAL`、`LOW` 和 `LOWEST`。 |
| SCAN_ROWS            | 扫描的数据行数。                    |
| FILTERED_ROWS        | 由于数据质量不足而被过滤掉的数据行数。 |
| UNSELECTED_ROWS      | 由于 WHERE 子句中指定的条件而筛选出的数据行数。 |
| SINK_ROWS            | 加载的数据行数。                     |
| ETL_INFO             | 加载作业的 ETL 详细信息。仅针对 Spark Load 返回非空值。对于其他类型的加载作业，返回空值。 |
| TASK_INFO            | 加载作业的任务执行详细信息，例如 `timeout` 和 `max_filter_ratio` 设置。 |
| CREATE_TIME          | 创建加载作业的时间。格式： `yyyy-MM-dd HH:mm:ss`。示例： `2023-07-24 14:58:58`。 |
| ETL_START_TIME       | 加载作业的 ETL 阶段的开始时间。格式： `yyyy-MM-dd HH:mm:ss`。示例： `2023-07-24 14:58:58`。 |
| ETL_FINISH_TIME      | 加载作业的 ETL 阶段的结束时间。格式： `yyyy-MM-dd HH:mm:ss`。示例： `2023-07-24 14:58:58`。 |
| LOAD_START_TIME      | 加载作业的 LOADING 阶段的开始时间。格式： `yyyy-MM-dd HH:mm:ss`。示例： `2023-07-24 14:58:58`。 |
| LOAD_FINISH_TIME     | 加载作业的 LOADING 阶段的结束时间。格式： `yyyy-MM-dd HH:mm:ss`。示例： `2023-07-24 14:58:58`。 |
| JOB_DETAILS          | 有关加载的数据的详细信息，例如字节数和文件数。 |
| ERROR_MSG            | 加载作业的错误消息。如果加载作业未遇到任何错误，`NULL` 则返回。 |
| TRACKING_URL         | 可从中访问在加载作业中检测到的不合格数据行示例的 URL。您可以使用 `curl` 或 `wget` 命令访问 URL 并获取不合格数据行示例。如果未检测到不合格的数据，`NULL` 则返回。 |
| TRACKING_SQL         | 可用于查询加载作业跟踪日志的 SQL 语句。仅当加载作业涉及不合格数据行时，才会返回 SQL 语句。如果加载作业不涉及任何不合格数据行，`NULL` 则返回。 |
| REJECTED_RECORD_PATH | 您可以从中访问在加载作业中筛选出的所有不合格数据行的路径。记录的不合格数据行数由加载作业中配置的 `log_rejected_record_num` 参数决定。您可以使用 `wget` 命令访问路径。如果加载作业不涉及任何不合格数据行，`NULL` 则返回。 |
