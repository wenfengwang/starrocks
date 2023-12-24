---
displayed_sidebar: English
---

# 任务

`tasks` 提供有关异步任务的信息。

`tasks` 中提供了以下字段：

| **字段**   | **描述**                                              |
| ----------- | ------------------------------------------------------------ |
| TASK_NAME   | 任务的名称。                                            |
| CREATE_TIME | 任务创建的时间。                               |
| SCHEDULE    | 任务的计划。如果任务定期触发，则此字段显示 `START xxx EVERY xxx`。 |
| DATABASE    | 任务所属的数据库。                             |
| DEFINITION  | 任务的 SQL 定义。                                  |
| EXPIRE_TIME | 任务过期的时间。                                  |