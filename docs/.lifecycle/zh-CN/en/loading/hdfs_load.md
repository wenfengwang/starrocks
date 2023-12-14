---
displayed_sidebar: "Chinese"
---

# 从HDFS加载数据

从 `../assets/commonMarkdown/loadMethodIntro.md` 导入加载方法简介

从 `../assets/commonMarkdown/insertPrivNote.md` 导入插入私有说明

StarRocks 提供以下选项来从 HDFS 加载数据:

<LoadMethodIntro />

## 开始之前

### 准备源数据

确保要加载到 StarRocks 的源数据已经妥善存储在您的 HDFS 集群中。本主题假定您要将 HDFS 中的 `/user/amber/user_behavior_ten_million_rows.parquet` 加载到 StarRocks 中。

### 检查权限

<InsertPrivNote />

### 收集连接详细信息

您可以使用简单认证方法与您的 HDFS 集群建立连接。要使用简单认证，您需要收集可以用于访问 HDFS 集群的 NameNode 的帐户的用户名和密码。

## 使用 INSERT+FILES()

该方法从 v3.1 开始可用，目前仅支持 Parquet 和 ORC 文件格式。

### INSERT+FILES() 的优势

[`FILES()`](../sql-reference/sql-functions/table-functions/files.md) 可以基于您指定的与路径相关的属性读取存储在云存储中的文件，推断文件中的数据表结构，然后将文件数据作为数据行返回。

使用 `FILES()`，您可以：

- 使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 直接从 HDFS 查询数据。
- 使用 [CREATE TABLE AS SELECT](../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md) (CTAS) 创建和加载表。
- 使用 [INSERT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 将数据加载到现有表中。

### 典型示例

#### 使用 SELECT 直接从 HDFS 查询

使用 SELECT+`FILES()` 直接从 HDFS 查询可以很好地预览数据集的内容，然后再创建表。例如：

- 获取数据预览而无需存储数据。
- 查询最小值和最大值，并决定使用什么数据类型。
- 检查 `NULL` 值。

下面的示例查询存储在 HDFS 集群中的数据文件 `/user/amber/user_behavior_ten_million_rows.parquet`:

```SQL
SELECT * FROM FILES
(
    "path" = "hdfs://<hdfs_ip>:<hdfs_port>/user/amber/user_behavior_ten_million_rows.parquet",
    "format" = "parquet",
    "hadoop.security.authentication" = "simple",
    "username" = "<hdfs用户名>",
    "password" = "<hdfs密码>"
)
LIMIT 3;
```

系统返回以下查询结果:

```Plaintext
+--------+---------+------------+--------------+---------------------+
| UserID | ItemID  | CategoryID | BehaviorType | Timestamp           |
+--------+---------+------------+--------------+---------------------+
| 543711 |  829192 |    2355072 | pv           | 2017-11-27 08:22:37 |
| 543711 | 2056618 |    3645362 | pv           | 2017-11-27 10:16:46 |
| 543711 | 1165492 |    3645362 | pv           | 2017-11-27 10:17:00 |
+--------+---------+------------+--------------+---------------------+
```

> **注意**
>
> 请注意上述返回的列名是由 Parquet 文件提供的。

#### 使用 CTAS 创建和加载表

这是前面示例的延续。将上一个查询包装在 CREATE TABLE AS SELECT (CTAS) 中，以自动化使用模式推断来创建表。这意味着 StarRocks 将推断表模式，创建您想要的表，然后将数据加载到该表中。使用 Parquet 文件的 `FILES()` 表函数时，不需要指定表的列名和类型，因为 Parquet 格式包含列名。

> **注意**
>
> 在使用模式推断时，CREATE TABLE 的语法不允许设置副本数量，因此在创建表之前请先设置它。下面的示例适用于副本数为1的系统:
>
> ```SQL
> ADMIN SET FRONTEND CONFIG ('default_replication_num' = "1");
> ```

创建一个数据库并切换到它:

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

使用 CTAS 创建一个表并将数据文件 `/user/amber/user_behavior_ten_million_rows.parquet` 中的数据加载到该表中:

```SQL
CREATE TABLE user_behavior_inferred AS
SELECT * FROM FILES
(
    "path" = "hdfs://<hdfs_ip>:<hdfs_port>/user/amber/user_behavior_ten_million_rows.parquet",
    "format" = "parquet",
    "hadoop.security.authentication" = "simple",
    "username" = "<hdfs用户名>",
    "password" = "<hdfs密码>"
);
```

创建表后，您可以使用 [DESCRIBE](../sql-reference/sql-statements/Utility/DESCRIBE.md) 查看其模式:

```SQL
DESCRIBE user_behavior_inferred;
```

系统返回以下查询结果:

```Plaintext
+--------------+------------------+------+-------+---------+-------+
| Field        | Type             | Null | Key   | Default | Extra |
+--------------+------------------+------+-------+---------+-------+
| UserID       | bigint           | YES  | true  | NULL    |       |
| ItemID       | bigint           | YES  | true  | NULL    |       |
| CategoryID   | bigint           | YES  | true  | NULL    |       |
| BehaviorType | varchar(1048576) | YES  | false | NULL    |       |
| Timestamp    | varchar(1048576) | YES  | false | NULL    |       |
+--------------+------------------+------+-------+---------+-------+
```

将推断的模式与手动创建的模式进行比较:

- 数据类型
- 可为空
- 关键字段

为了更好地控制目标表的模式并获得更好的查询性能，我们建议您在生产环境中手动指定表模式。

查询表以验证数据是否已加载到其中。示例:

```SQL
SELECT * from user_behavior_inferred LIMIT 3;
```

返回以下查询结果，指示数据已成功加载:

```Plaintext
+--------+--------+------------+--------------+---------------------+
| UserID | ItemID | CategoryID | BehaviorType | Timestamp           |
+--------+--------+------------+--------------+---------------------+
|     84 |  56257 |    1879194 | pv           | 2017-11-26 05:56:23 |
|     84 | 108021 |    2982027 | pv           | 2017-12-02 05:43:00 |
|     84 | 390657 |    1879194 | pv           | 2017-11-28 11:20:30 |
+--------+--------+------------+--------------+---------------------+
```

#### 使用 INSERT 将数据加载到现有表中

您可能希望自定义要插入的表，例如:

- 列数据类型、可为空设置或默认值
- 键类型和列
- 数据分区和桶分配

> **注意**
>
> 创建最有效的表结构需要了解数据将如何使用以及列的内容。本主题不涵盖表设计。有关表设计的信息，请参阅 [表类型](../table_design/StarRocks_table_design.md)。

在本示例中，我们根据对将查询表以及 Parquet 文件中的数据的了解来创建表。

- 由于查询 HDFS 中的数据集表明 `Timestamp` 列包含与 `datetime` 数据类型匹配的数据，所以下面的 DDL 中指定了列类型。
- 通过查询 HDFS 中的数据，您可以发现数据集中没有 `NULL` 值，因此 DDL 不将任何列设置为可为空。
- 基于对期望查询类型的了解，将排序键和桶列设置为列 `UserID`。对于此数据，您可能会针对此数据采用不同的用例，因此您可能决定使用 `UserID` 之外的列或代替 `UserID` 作为排序键。

创建一个数据库并切换到它:

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

手动创建一个表(我们建议该表与您要从 HDFS 加载的 Parquet 文件具有相同的模式):

```SQL
CREATE TABLE user_behavior_declared
(
    UserID int(11),
    ItemID int(11),
    CategoryID int(11),
    BehaviorType varchar(65533),
    Timestamp datetime
)
ENGINE = OLAP 
DUPLICATE KEY(UserID)
DISTRIBUTED BY HASH(UserID)
PROPERTIES
(
    "replication_num" = "1"
);
```

创建表后，您可以使用 INSERT INTO SELECT FROM FILES() 将其加载:

```SQL
INSERT INTO user_behavior_declared
```
SELECT * FROM FILES
(
    "path" = "hdfs://<hdfs_ip>:<hdfs_port>/user/amber/user_behavior_ten_million_rows.parquet",
    "format" = "parquet",
    "hadoop.security.authentication" = "simple",
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);

```

加载完成后，您可以查询表格以验证数据是否已经加载。例如：

```SQL

SELECT * from user_behavior_declared LIMIT 3;

```

返回以下查询结果，表明数据已成功加载：

```Plaintext
+--------+---------+------------+--------------+---------------------+
| UserID | ItemID  | CategoryID | BehaviorType | Timestamp           |
+--------+---------+------------+--------------+---------------------+
|    107 | 1568743 |    4476428 | pv           | 2017-11-25 14:29:53 |

|    107 |  470767 |    1020087 | pv           | 2017-11-25 14:32:31 |

|    107 |  358238 |    1817004 | pv           | 2017-11-25 14:43:23 |

+--------+---------+------------+--------------+---------------------+
```


#### 检查加载进度

您可以从`information_schema.loads`视图中查询INSERT任务的进度。该功能从v3.1开始支持。例如:

```SQL
SELECT * FROM information_schema.loads ORDER BY JOB_ID DESC;
```

如果您已经提交了多个加载任务，您可以根据与作业关联的`LABEL`进行过滤. 例如：

```SQL
SELECT * FROM information_schema.loads WHERE LABEL = 'insert_0d86c3f9-851f-11ee-9c3e-00163e044958' \G
*************************** 1. row ***************************
              JOB_ID: 10214
               LABEL: insert_0d86c3f9-851f-11ee-9c3e-00163e044958
       DATABASE_NAME: mydatabase
               STATE: FINISHED
            PROGRESS: ETL:100%; LOAD:100%
                TYPE: INSERT
            PRIORITY: NORMAL
           SCAN_ROWS: 10000000
       FILTERED_ROWS: 0
     UNSELECTED_ROWS: 0
           SINK_ROWS: 10000000
            ETL_INFO:
           TASK_INFO: resource:N/A; timeout(s):300; max_filter_ratio:0.0
         CREATE_TIME: 2023-11-17 15:58:14
      ETL_START_TIME: 2023-11-17 15:58:14
     ETL_FINISH_TIME: 2023-11-17 15:58:14

     LOAD_START_TIME: 2023-11-17 15:58:14

    LOAD_FINISH_TIME: 2023-11-17 15:58:18
         JOB_DETAILS: {"All backends":{"0d86c3f9-851f-11ee-9c3e-00163e044958":[10120]},"FileNumber":0,"FileSize":0,"InternalTableLoadBytes":311710786,"InternalTableLoadRows":10000000,"ScanBytes":581574034,"ScanRows":10000000,"TaskNumber":1,"Unfinished backends":{"0d86c3f9-851f-11ee-9c3e-00163e044958":[]}}
           ERROR_MSG: NULL

        TRACKING_URL: NULL

        TRACKING_SQL: NULL

REJECTED_RECORD_PATH: NULL

```

有关`loads`视图提供的字段的信息，请参见[信息模式](../reference/information_schema/loads.md)。

> **注意**
>

> INSERT是一个同步命令。如果INSERT任务仍在运行，您需要打开另一个会话来查询它的执行状态。

## 使用Broker加载

异步的Broker加载过程处理与HDFS的连接，数据的拉取以及数据存储在StarRocks中。

此方法支持Parquet、ORC和CSV文件格式。

### Broker加载的优势

- Broker加载支持[数据转换](../loading/Etl_in_loading.md)和[数据更改](../loading/Load_to_Primary_Key_tables.md)，比如加载期间的UPSERT和DELETE操作。

- Broker加载在后台运行，客户端不需要保持连接才能继续作业。

- 对于长时间运行的作业，Broker加载是首选，其默认超时时间为4小时。

- 除了Parquet和ORC文件格式，Broker加载还支持CSV文件。

### 数据流


![Broker加载工作流程](../assets/broker_load_how-to-work_en.png)

1. 用户创建加载作业。
2. 前端（FE）创建查询计划并将计划分发到后端节点（BE）。
3. 后端从源中拉取数据并将数据加载到StarRocks中。

### 典型示例

创建一个表格，启动一个加载进程，从HDFS中的数据文件`/user/amber/user_behavior_ten_million_rows.parquet`，并验证数据加载的进度和成功。

#### 创建数据库和表格

创建一个数据库并切换到它：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```


手动创建一个表（我们建议表与您要从HDFS加载的Parquet文件具有相同的模式）：

```SQL

CREATE TABLE user_behavior
(
    UserID int(11),
    ItemID int(11),
    CategoryID int(11),
    BehaviorType varchar(65533),
    Timestamp datetime
)
ENGINE = OLAP 
DUPLICATE KEY(UserID)
DISTRIBUTED BY HASH(UserID)
PROPERTIES
(
    "replication_num" = "1"
);
```

#### 启动Broker加载

运行以下命令，开始一个Broker加载作业，将数据文件`/user/amber/user_behavior_ten_million_rows.parquet`的数据加载到`user_behavior`表格中：

```SQL
LOAD LABEL user_behavior
(
    DATA INFILE("hdfs://<hdfs_ip>:<hdfs_port>/user/amber/user_behavior_ten_million_rows.parquet")

    INTO TABLE user_behavior

    FORMAT AS "parquet"

 )

 WITH BROKER
(
    "hadoop.security.authentication" = "simple",

    "username" = "<hdfs_username>",

    "password" = "<hdfs_password>"

)
PROPERTIES
(

    "timeout" = "72000"

);
```

这个任务有四个主要部分：

- `LABEL`：查询加载作业状态时使用的字符串。
10121|user_behavior                              |mydatabase   |CANCELLED|ETL:N/A; LOAD:N/A  |BROKER|NORMAL  |        0|            0|              0|        0|        |resource:N/A; timeout(s):72000; max_filter_ratio:0.0|2023-08-10 14:59:30|                   |                   |                   |2023-08-10 14:59:34|{"所有后端":{},"文件编号":0,"文件大小":0,"内部表加载字节":0,"内部表加载行数":0,"扫描字节":0,"扫描行数":0,"任务编号":0,"未完成的后端":{}}                                                                                        |type:ETL_RUN_FAIL; msg:listPath failed|            |            |                    |
10106|user_behavior                              |mydatabase   |FINISHED |ETL:100%; LOAD:100%|BROKER|NORMAL  | 86953525|            0|              0| 86953525|        |resource:N/A; timeout(s):72000; max_filter_ratio:0.0|2023-08-10 14:50:15|2023-08-10 14:50:19|2023-08-10 14:50:19|2023-08-10 14:50:19|2023-08-10 14:55:10|{"所有后端":{"a5fe5e1d-d7d0-4826-ba99-c7348f9a5f2f":[10004]},"文件编号":1,"文件大小":1225637388,"内部表加载字节":2710603082,"内部表加载行数":86953525,"扫描字节":1225637388,"扫描行数":86953525,"任务编号":1,"未完成的后端":{"a5|                                      |            |            |                    |
```json
    LOAD_STATUS: {"loadedFiles":1,"loadedBytes":132251298,"loadingFiles":0,"lastLoadedTime":"2023-11-17 16:13:22"}
    LAST_ERROR:
  CREATED_TIME: 2023-11-17 16:13:15
1 row in set (0.00 sec)
```

#### 检查文件状态

您可以查询从 [`information_schema.pipe_files`](../reference/information_schema/pipe_files.md) 视图加载的文件的加载状态。

```SQL
SELECT * FROM information_schema.pipe_files;
```

如果您提交了多个加载作业，您可以按与作业关联的`PIPE_NAME`进行筛选。例如：

```SQL
SELECT * FROM information_schema.pipe_files WHERE pipe_name = 'user_behavior_replica' \G
*************************** 1. row ***************************
   DATABASE_NAME: mydatabase
         PIPE_ID: 10252
       PIPE_NAME: user_behavior_replica
       FILE_NAME: hdfs://172.26.195.67:9000/user/amber/user_behavior_ten_million_rows.parquet
    FILE_VERSION: 1700035418838
       FILE_SIZE: 132251298
   LAST_MODIFIED: 2023-11-15 08:03:38
      LOAD_STATE: FINISHED
     STAGED_TIME: 2023-11-17 16:13:16
 START_LOAD_TIME: 2023-11-17 16:13:17
FINISH_LOAD_TIME: 2023-11-17 16:13:22
       ERROR_MSG:
1 row in set (0.02 sec)
```

#### 管理数据管道

您可以修改、挂起或恢复、删除或查询您已创建的数据管道，并重试加载特定数据文件。有关更多信息，请参阅 [ALTER PIPE](../sql-reference/sql-statements/data-manipulation/ALTER_PIPE.md), [SUSPEND or RESUME PIPE](../sql-reference/sql-statements/data-manipulation/SUSPEND_or_RESUME_PIPE.md), [DROP PIPE](../sql-reference/sql-statements/data-manipulation/DROP_PIPE.md), [SHOW PIPES](../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md), 和 [RETRY FILE](../sql-reference/sql-statements/data-manipulation/RETRY_FILE.md)。