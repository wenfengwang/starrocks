---
displayed_sidebar: "Chinese"
---

# 显示管道

## 功能

查看当前数据库或指定数据库下的管道。此命令从 3.2 版本开始受支持。

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

## 参数说明

### FROM `<db_name>`

指定要查询的数据库名称。如果不指定数据库，则默认查询当前数据库下的管道。

### WHERE

指定查询条件。

### ORDER BY `<field_name>`

指定结果记录的排序字段。

### LIMIT

指定系统返回的最大结果记录数。

## 返回结果

返回结果包括如下字段：

| **字段**      | **说明**                                                     |
| ------------- | ------------------------------------------------------------ |
| DATABASE_NAME | 管道所属数据库的名称。                                      |
| ID            | 管道的唯一 ID。                                             |
| NAME          | 管道的名称。                                                |
| TABLE_NAME    | StarRocks 目标表的名称。                                     |
| STATE         | 管道的状态，包括 `RUNNING`、`FINISHED`、`SUSPENDED`、`ERROR`。 |
| LOAD_STATUS   | 管道下待导入数据文件的整体状态，包括如下字段：<ul><li>`loadedFiles`：已导入的数据文件总个数。</li><li>`loadedBytes`：已导入的数据总量，单位为字节。</li><li>`LastLoadedTime`：最近一次执行导入的时间。格式：`yyyy-MM-dd HH:mm:ss`。例如，`2023-07-24 14:58:58`。</li></ul> |
| LAST_ERROR    | 管道执行过程中最近一次错误的详细信息。默认值为 `NULL`。     |
| CREATED_TIME  | 管道的创建时间。格式：`yyyy-MM-dd HH:mm:ss`。例如：`2023-07-24 14:58:58`。 |

## 示例

### 查询所有管道

进入数据库 `mydatabase`，然后查看该数据库下的所有管道：

```SQL
USE mydatabase;
SHOW PIPES \G
```

### 查询指定管道

进入数据库 `mydatabase`，然后查看该数据库下名称为 `user_behavior_replica` 的管道：

```SQL
USE mydatabase;
SHOW PIPES WHERE NAME = 'user_behavior_replica' \G
```