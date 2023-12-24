---
displayed_sidebar: English
---

# task_runs

`task_runs` 提供有关异步任务执行的信息。

以下是 `task_runs` 中提供的字段：

| **字段**     | **描述**                                              |
| ------------- | ------------------------------------------------------------ |
| QUERY_ID      | 查询的ID。                                             |
| TASK_NAME     | 任务的名称。                                            |
| CREATE_TIME   | 任务创建的时间。                               |
| FINISH_TIME   | 任务完成的时间。                                 |
| STATE         | 任务的状态。有效值： `PENDING`、 `RUNNING`、 `FAILED`和 `SUCCESS`。 |
| DATABASE      | 任务所属的数据库。                             |
| DEFINITION    | 任务的SQL定义。                                  |
| EXPIRE_TIME   | 任务到期的时间。                                  |
| ERROR_CODE    | 任务的错误代码。                                      |
| ERROR_MESSAGE | 任务的错误消息。                                   |
| PROGRESS      | 任务的进度。                                    |
| EXTRA_MESSAGE | 任务的额外消息，例如异步物化视图创建任务中的分区信息。 |
