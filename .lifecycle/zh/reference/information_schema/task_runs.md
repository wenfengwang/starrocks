---
displayed_sidebar: English
---

# 任务运行情况

task_runs 提供了有关异步任务执行的信息。

在 task_runs 中，提供了以下字段：

|字段|描述|
|---|---|
|QUERY_ID|查询的 ID。|
|TASK_NAME|任务名称。|
|CREATE_TIME|创建任务的时间。|
|FINISH_TIME|任务完成的时间。|
|STATE|任务的状态。有效值：PENDING、RUNNING、FAILED 和 SUCCESS。|
|DATABASE|任务所属的数据库。|
|DEFINITION|任务的 SQL 定义。|
|EXPIRE_TIME|任务到期时间。|
|ERROR_CODE|任务的错误代码。|
|ERROR_MESSAGE|任务的错误消息。|
|PROGRESS|任务的进度。|
|EXTRA_MESSAGE|任务的额外消息，例如异步物化视图创建任务中的分区信息。|
