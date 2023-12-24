---
displayed_sidebar: English
---

# materialized_views

`materialized_views` 提供有关所有异步实例化视图的信息。

`materialized_views` 中提供了以下字段：

| **字段**                            | **描述**                                              |
| ------------------------------------ | ------------------------------------------------------------ |
| MATERIALIZED_VIEW_ID                 | 实例化视图的 ID。                                 |
| TABLE_SCHEMA                         | 实例化视图所在的数据库。             |
| TABLE_NAME                           | 实例化视图的名称。                               |
| REFRESH_TYPE                         | 实例化视图的刷新类型。有效值： `ROLLUP`、 `ASYNC`和 `MANUAL`。 |
| IS_ACTIVE                            | 指示实例化视图是否处于活动状态。非活动的实例化视图无法进行刷新或查询。 |
| INACTIVE_REASON                      | 实例化视图处于非活动状态的原因。           |
| PARTITION_TYPE                       | 实例化视图的分区策略类型。     |
| TASK_ID                              | 负责刷新实例化视图的任务的 ID。 |
| TASK_NAME                            | 负责刷新实例化视图的任务的名称。 |
| LAST_REFRESH_START_TIME              | 最近一次刷新任务的开始时间。                  |
| LAST_REFRESH_FINISHED_TIME           | 最近一次刷新任务的结束时间。                    |
| LAST_REFRESH_DURATION                | 最近一次刷新任务的持续时间。                    |
| LAST_REFRESH_STATE                   | 最近一次刷新任务的状态。                       |
| LAST_REFRESH_FORCE_REFRESH           | 指示最近一次刷新任务是否为强制刷新。 |
| LAST_REFRESH_START_PARTITION         | 最近一次刷新任务的起始分区。         |
| LAST_REFRESH_END_PARTITION           | 最近一次刷新任务的结束分区。           |
| LAST_REFRESH_BASE_REFRESH_PARTITIONS | 最近一次刷新任务涉及的基表分区。 |
| LAST_REFRESH_MV_REFRESH_PARTITIONS   | 最近一次刷新任务中刷新的实例化视图分区。 |
| LAST_REFRESH_ERROR_CODE              | 最近一次刷新任务的错误代码。                  |
| LAST_REFRESH_ERROR_MESSAGE           | 最近一次刷新任务的错误消息。               |
| TABLE_ROWS                           | 基于近似背景统计信息，实例化视图中的数据行数。 |
| MATERIALIZED_VIEW_DEFINITION         | 实例化视图的 SQL 定义。                     |
