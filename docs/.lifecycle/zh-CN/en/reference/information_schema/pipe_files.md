---
displayed_sidebar: "Chinese"
---

# pipe_files

`pipe_files`提供通过指定的pipe加载的数据文件的状态。此视图从StarRocks v3.2开始支持。

在`pipe_files`中提供以下字段：

| **字段**           | **描述**                                                     |
| ---------------- | ------------------------------------------------------------ |
| DATABASE_NAME    | 存储pipe的数据库的名称。                                        |
| PIPE_ID          | pipe的唯一ID。                                                  |
| PIPE_NAME        | pipe的名称。                                                    |
| FILE_NAME        | 数据文件的名称。                                                 |
| FILE_VERSION     | 数据文件的摘要。                                                 |
| FILE_SIZE        | 数据文件的大小。单位：字节。                                       |
| LAST_MODIFIED    | 数据文件上次修改的时间。格式：`yyyy-MM-dd HH:mm:ss`。例如：`2023-07-24 14:58:58`。 |
| LOAD_STATE       | 数据文件的加载状态。有效值：`UNLOADED`、`LOADING`、`FINISHED`和`ERROR`。 |
| STAGED_TIME      | 数据文件首次被pipe记录的日期和时间。格式：`yyyy-MM-dd HH:mm:ss`。例如：`2023-07-24 14:58:58`。 |
| START_LOAD_TIME  | 数据文件加载开始的日期和时间。格式：`yyyy-MM-dd HH:mm:ss`。例如：`2023-07-24 14:58:58`。 |
| FINISH_LOAD_TIME | 数据文件加载完成的日期和时间。格式：`yyyy-MM-dd HH:mm:ss`。例如：`2023-07-24 14:58:58`。 |
| ERROR_MSG        | 数据文件加载错误的详细信息。                                       |