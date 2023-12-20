---
displayed_sidebar: English
---

# task_runs

`task_runs` 提供有关异步任务执行的信息。

`task_runs` 中提供以下字段：

|**字段**|**描述**|
|---|---|
|QUERY_ID|查询 ID。|
|TASK_NAME|任务名称。|
|CREATE_TIME|任务创建时间。|
|FINISH_TIME|任务完成时间。|
|STATE|任务状态。有效值：`PENDING`、`RUNNING`、`FAILED` 和 `SUCCESS`。|
|DATABASE|任务所属数据库。|
|DEFINITION|任务的 SQL 定义。|
|EXPIRE_TIME|任务过期时间。|
|ERROR_CODE|任务错误代码。|
|ERROR_MESSAGE|任务错误信息。|
|PROGRESS|任务进度。|
|EXTRA_MESSAGE|任务额外信息，例如，在异步物化视图创建任务中的分区信息。|