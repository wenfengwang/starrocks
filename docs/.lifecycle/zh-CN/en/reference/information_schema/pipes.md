---
displayed_sidebar: "Chinese"
---

# 管道

`pipes`提供有关存储在当前或指定数据库中的所有管道的信息。 从StarRocks v3.2起支持此视图。

在`pipes`提供以下字段：

| **字段**       | **描述**                                                              |
| ------------- | ------------------------------------------------------------ |
| DATABASE_NAME | 存储管道的数据库名称。                                                   |
| PIPE_ID       | 管道的唯一ID。                                                           |
| PIPE_NAME     | 管道的名称。                                                             |
| TABLE_NAME    | 目标表的名称。格式：`<database_name>.<table_name>`。                        |
| STATE         | 管道的状态。有效值：`RUNNING`，`FINISHED`，`SUSPENDED`和`ERROR`。      |
| LOAD_STATUS   | 通过管道加载的数据文件的整体状态，包括以下子字段：<ul><li>`loadedFiles`：已加载的数据文件数量。</li><li>`loadedBytes`：已加载的数据量，以字节为单位。</li><li>`loadingFiles`：正在加载的数据文件数量。</li></ul> |
| LAST_ERROR    | 管道执行期间发生的最后错误的详细信息。默认值：`NULL`。                     |
| CREATED_TIME  | 管道创建的日期和时间。格式：`yyyy-MM-dd HH:mm:ss`。 示例：`2023-07-24 14:58:58`。 |