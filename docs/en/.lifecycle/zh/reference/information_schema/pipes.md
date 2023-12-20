---
displayed_sidebar: English
---

# pipes

`pipes` 提供有关存储在当前或指定数据库中的所有管道的信息。此视图从 StarRocks v3.2 起得到支持。

`pipes` 中提供了以下字段：

|**字段**|**描述**|
|---|---|
|DATABASE_NAME|存储管道的数据库名称。|
|PIPE_ID|管道的唯一 ID。|
|PIPE_NAME|管道的名称。|
|TABLE_NAME|目标表的名称。格式：`<database_name>.<table_name>`。|
|STATE|管道的状态。有效值：`RUNNING`（运行中）、`FINISHED`（已完成）、`SUSPENDED`（已暂停）和 `ERROR`（错误）。|
|LOAD_STATUS|通过管道加载的数据文件的整体状态，包括以下子字段：<ul><li>`loadedFiles`：已加载的数据文件数量。</li><li>`loadedBytes`：已加载的数据量，以字节为单位。</li><li>`loadingFiles`：正在加载的数据文件数量。</li></ul>|
|LAST_ERROR|管道执行期间发生的最后一个错误的详细信息。默认值：`NULL`。|
|CREATED_TIME|创建管道的日期和时间。格式：`yyyy-MM-dd HH:mm:ss`。示例：`2023-07-24 14:58:58`。|