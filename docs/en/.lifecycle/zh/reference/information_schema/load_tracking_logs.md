---
displayed_sidebar: English
---

# load_tracking_logs

`load_tracking_logs` 提供加载作业的错误日志。该视图从 StarRocks v3.0 版本开始支持。

`load_tracking_logs` 中提供了以下字段：

| **字段**     | **描述**                            |
| ------------- | ------------------------------------------ |
| JOB_ID        | 加载作业的ID。                    |
| LABEL         | 加载作业的标签。                 |
| DATABASE_NAME | 加载作业所属的数据库。 |
| TRACKING_LOG  | 加载作业的错误（如果有）。           |
