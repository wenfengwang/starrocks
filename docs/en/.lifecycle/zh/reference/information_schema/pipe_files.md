---
displayed_sidebar: English
---

# pipe_files

`pipe_files` 提供了通过指定管道加载的数据文件的状态。该视图从 StarRocks v3.2 版本开始支持。

`pipe_files` 中提供了以下字段：

|**字段**|**描述**|
|---|---|
|DATABASE_NAME|存储管道的数据库名称。|
|PIPE_ID|管道的唯一标识符。|
|PIPE_NAME|管道的名称。|
|FILE_NAME|数据文件的名称。|
|FILE_VERSION|数据文件的摘要。|
|FILE_SIZE|数据文件的大小。单位：字节。|
|LAST_MODIFIED|数据文件最后修改的时间。格式：`yyyy-MM-dd HH:mm:ss`。示例：`2023-07-24 14:58:58`。|
|LOAD_STATE|数据文件的加载状态。有效值：`UNLOADED`、`LOADING`、`FINISHED` 和 `ERROR`。|
|STAGED_TIME|管道首次记录数据文件的日期和时间。格式：`yyyy-MM-dd HH:mm:ss`。示例：`2023-07-24 14:58:58`。|
|START_LOAD_TIME|开始加载数据文件的日期和时间。格式：`yyyy-MM-dd HH:mm:ss`。示例：`2023-07-24 14:58:58`。|
|FINISH_LOAD_TIME|数据文件加载完成的日期和时间。格式：`yyyy-MM-dd HH:mm:ss`。示例：`2023-07-24 14:58:58`。|
|ERROR_MSG|关于数据文件加载错误的详细信息。|