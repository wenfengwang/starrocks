---
displayed_sidebar: English
---

# 管道

Pipes 提供当前或指定数据库中所有管道的信息。该视图从 StarRocks v3.2 版本开始支持。

管道视图包含以下字段：

|字段|描述|
|---|---|
|DATABASE_NAME|存储管道的数据库的名称。|
|PIPE_ID|管道的唯一ID。|
|PIPE_NAME|管道的名称。|
|TABLE_NAME|目标表的名称。格式：<数据库名称>.<表名称>.|
|STATE|管道的状态。有效值：正在运行、已完成、已暂停和错误。|
|LOAD_STATUS|要通过管道加载的数据文件的总体状态，包括以下子字段：loadedFiles：已加载的数据文件数量。loadedBytes：已加载的数据量，以字节为单位.loadingFiles：正在加载的数据文件的数量。|
|LAST_ERROR|有关管道执行期间发生的最后一个错误的详细信息。默认值：NULL。|
|CREATED_TIME|创建管道的日期和时间。格式：yyyy-MM-dd HH:mm:ss。示例：2023-07-24 14:58:58。|
