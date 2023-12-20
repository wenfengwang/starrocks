---
displayed_sidebar: English
---

# 显示管道

## 描述

列出指定数据库或当前正在使用的数据库中存储的管道。

## 语法

```SQL
SHOW PIPES [FROM <db_name>]
[
   WHERE [ NAME { = "<pipe_name>" | LIKE "pipe_matcher" } ]
         [ [AND] STATE = { "SUSPENDED" | "RUNNING" | "ERROR" } ]
]
[ ORDER BY <field_name> [ ASC | DESC ] ]
[ LIMIT { [offset, ] limit | limit OFFSET offset } ]
```

## 参数

### FROM <数据库名称>

您想要查询管道的数据库名称。如果您没有指定此参数，系统将返回当前正在使用的数据库中的管道。

### WHERE

基于哪些条件来查询管道。

### ORDER BY <字段名称>

您希望根据哪个字段对返回的记录进行排序。

### LIMIT

您希望系统返回的记录的最大数量。

## 返回结果

命令输出包括以下字段。

|字段|描述|
|---|---|
|DATABASE_NAME|存储管道的数据库的名称。|
|PIPE_ID|管道的唯一ID。|
|PIPE_NAME|管道的名称。|
|TABLE_NAME|目标 StarRocks 表的名称。|
|STATE|管道的状态。有效值：正在运行、已完成、已暂停和错误。|
|LOAD_STATUS|要通过管道加载的数据文件的总体状态，包括以下子字段：loadedFiles：已加载的数据文件数量。loadedBytes：已加载的数据量，以字节为单位.LastLoadedTime：最后一次加载数据文件的日期和时间。格式：yyyy-MM-dd HH:mm:ss。示例：2023-07-24 14:58:58。|
|LAST_ERROR|有关管道执行期间发生的最后一个错误的详细信息。默认值：NULL。|
|CREATED_TIME|创建管道的日期和时间。格式：yyyy-MM-dd HH:mm:ss。示例：2023-07-24 14:58:58。|

## 示例

### 查询所有管道

切换到名为mydatabase的数据库，并显示其中的所有管道：

```SQL
USE mydatabase;
SHOW PIPES \G
```

### 查询指定管道

切换到名为mydatabase的数据库，并显示其中名为user_behavior_replica的管道：

```SQL
USE mydatabase;
SHOW PIPES WHERE NAME = 'user_behavior_replica' \G
```
