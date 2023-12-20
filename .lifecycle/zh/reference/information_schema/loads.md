---
displayed_sidebar: English
---

# 负载作业结果

`loads`提供了负载作业的结果。该视图从StarRocks v3.1版本开始支持。目前，您仅能通过此视图查看[Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)和[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)作业的结果。

在负载作业结果视图中提供了以下字段：

|字段|描述|
|---|---|
|JOB_ID|StarRocks 分配的用于标识加载作业的唯一 ID。|
|LABEL|加载作业的标签。|
|DATABASE_NAME|目标 StarRocks 表所属的数据库的名称。|
|STATE|加载作业的状态。有效值：PENDING：加载作业已创建。QUEUEING：加载作业在队列中等待调度。LOADING：加载作业正在运行。PREPARED：事务已提交。FINISHED：加载作业成功。CANCELLED：加载作业失败。有关详细信息，请参阅异步加载。|
|PROGRESS|加载作业的 ETL 阶段和 LOADING 阶段的进度。|
|TYPE|加载作业的类型。对于 Broker Load，返回值为 BROKER。对于 INSERT，返回值为 INSERT。|
|优先级|加载作业的优先级。有效值：最高、最高、正常、最低和最低。|
|SCAN_ROWS|扫描的数据行数。|
|FILTERED_ROWS|由于数据质量不足而被过滤掉的数据行数。|
|UNSELECTED_ROWS|由于 WHERE 子句中指定的条件而过滤掉的数据行数。|
|SINK_ROWS|加载的数据行数。|
|ETL_INFO|加载作业的 ETL 详细信息。仅针对 Spark Load 返回非空值。对于任何其他类型的加载作业，将返回空值。|
|TASK_INFO|加载作业的任务执行详细信息，例如超时和 max_filter_ratio 设置。|
|CREATE_TIME|创建加载作业的时间。格式：yyyy-MM-dd HH:mm:ss。示例：2023-07-24 14:58:58。|
|ETL_START_TIME|加载作业的 ETL 阶段的开始时间。格式：yyyy-MM-dd HH:mm:ss。示例：2023-07-24 14:58:58。|
|ETL_FINISH_TIME|加载作业的 ETL 阶段的结束时间。格式：yyyy-MM-dd HH:mm:ss。示例：2023-07-24 14:58:58。|
|LOAD_START_TIME|加载作业的LOADING阶段的开始时间。格式：yyyy-MM-dd HH:mm:ss。示例：2023-07-24 14:58:58。|
|LOAD_FINISH_TIME|加载作业的 LOADING 阶段的结束时间。格式：yyyy-MM-dd HH:mm:ss。示例：2023-07-24 14:58:58。|
|JOB_DETAILS|加载数据的详细信息，例如字节数和文件数。|
|ERROR_MSG|加载作业的错误消息。如果加载作业没有遇到任何错误，则返回 NULL。|
|TRACKING_URL|您可以访问加载作业中检测到的不合格数据行样本的 URL。您可以使用curl或wget命令访问URL并获取不合格的数据行样本。如果没有检测到不合格的数据，则返回 NULL。|
|TRACKING_SQL|可用于查询加载作业跟踪日志的SQL语句。仅当加载作业涉及不合格数据行时才会返回 SQL 语句。如果加载作业不涉及任何不合格的数据行，则返回 NULL。|
|REJECTED_RECORD_PATH|您可以访问加载作业中过滤掉的所有不合格数据行的路径。记录的不合格数据行数由加载作业中配置的 log_rejected_record_num 参数确定。您可以使用 wget 命令来访问该路径。如果加载作业不涉及任何不合格的数据行，则返回 NULL。|
