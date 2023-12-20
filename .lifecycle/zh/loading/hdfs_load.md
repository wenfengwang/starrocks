---
displayed_sidebar: English
---

# 从 HDFS 加载数据

从 '../assets/commonMarkdown/loadMethodIntro.md' 导入 LoadMethodIntro

从 '../assets/commonMarkdown/insertPrivNote.md' 导入 InsertPrivNote

StarRocks 提供以下选项用于从 HDFS 加载数据：

<LoadMethodIntro />


## 开始之前

### 准备源数据

请确保您要加载到 StarRocks 中的源数据已正确存储在 HDFS 集群中。本主题假设您想要将 /user/amber/user_behavior_ten_million_rows.parquet 从 HDFS 加载到 StarRocks。

### 检查权限

<InsertPrivNote />


### 搜集连接详情

您可以使用简单认证方法来与您的 HDFS 集群建立连接。为使用简单认证，您需要搜集可以访问 HDFS 集群的 NameNode 的账户的用户名和密码。

## 使用 INSERT+FILES()

该方法从 v3.1 版本开始提供，目前仅支持 Parquet 和 ORC 文件格式。

### INSERT+FILES() 的优势

[`FILES()`](../sql-reference/sql-functions/table-functions/files.md) 可以基于您指定的与路径相关的属性读取存储于云存储中的文件，推断文件中数据的表结构，然后将文件中的数据作为数据行返回。

使用 FILES()，您可以：

- 通过[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)直接从 HDFS 查询数据。
- 使用[CREATE TABLE AS SELECT](../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md) (CTAS)创建并加载表。
- 使用 [INSERT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 将数据加载进已有表。

### 典型示例

#### 直接使用 SELECT 从 HDFS 查询

使用 SELECT+FILES() 直接从 HDFS 查询可以在创建表之前预览数据集内容。例如：

- 在不存储数据的情况下预览数据集。
- 查询最小值和最大值来决定使用哪些数据类型。
- 检查 NULL 值。

以下示例查询了存储在 HDFS 集群中的数据文件 /user/amber/user_behavior_ten_million_rows.parquet：

```SQL
SELECT * FROM FILES
(
    "path" = "hdfs://<hdfs_ip>:<hdfs_port>/user/amber/user_behavior_ten_million_rows.parquet",
    "format" = "parquet",
    "hadoop.security.authentication" = "simple",
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
)
LIMIT 3;
```

系统返回以下查询结果：

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
> 请注意，上述返回的列名是由 Parquet 文件提供的。

#### 使用 CTAS 创建并加载表

这是前一个示例的延续。先前的查询被包裹在 CREATE TABLE AS SELECT (CTAS) 中，以自动利用架构推断来创建表。这意味着 StarRocks 会推断表的架构，创建您所需的表，然后加载数据。使用 FILES() 表函数与 Parquet 文件时，创建表不需要指定列名和类型，因为 Parquet 格式已包含列名。

> **注意**
> 使用架构推断的 CREATE TABLE 语法不允许设置**副本数量**，请在创建表之前设置。以下示例适用于**单副本**系统：
```SQL
ADMIN SET FRONTEND CONFIG ('default_replication_num' = "1");
```

创建数据库并切换：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

使用 CTAS 创建表，并将数据文件 /user/amber/user_behavior_ten_million_rows.parquet 的数据加载进表中：

```SQL
CREATE TABLE user_behavior_inferred AS
SELECT * FROM FILES
(
    "path" = "hdfs://<hdfs_ip>:<hdfs_port>/user/amber/user_behavior_ten_million_rows.parquet",
    "format" = "parquet",
    "hadoop.security.authentication" = "simple",
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

创建表后，您可以使用 [DESCRIBE](../sql-reference/sql-statements/Utility/DESCRIBE.md) 查看其架构：

```SQL
DESCRIBE user_behavior_inferred;
```

系统返回以下查询结果：

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

比较推断出的架构与手动创建的架构：

- 数据类型
- 是否可为空
- 关键字段

为了更好地控制目标表的架构并提升查询性能，我们建议您在生产环境中手动指定表架构。

查询表以确认数据已被加载。例如：

```SQL
SELECT * from user_behavior_inferred LIMIT 3;
```

返回以下查询结果，表明数据已成功加载：

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

您可能希望自定义您正在插入的表，例如：

- 列数据类型、可空设置或默认值
- 键类型和列
- 数据分区和桶排序

> **注意**
> 创建最高效的表结构需要了解数据的使用方式和列的内容。本主题不涵盖表设计。有关表设计的信息，请参见[表类型](../table_design/StarRocks_table_design.md)。

在这个示例中，我们根据表将如何被查询以及 Parquet 文件中的数据知识创建了一个表。可以通过直接在 HDFS 中查询文件来获得 Parquet 文件中数据的知识。

- 由于在 HDFS 中查询数据集显示 Timestamp 列包含与 datetime 数据类型匹配的数据，因此在以下 DDL 中指定了列类型。
- 通过查询 HDFS 中的数据，我们发现数据集中没有 NULL 值，因此 DDL 没有将任何列设置为可空。
- 基于对预期查询类型的了解，将排序键和桶列设置为 UserID 列。您的用例可能会有所不同，因此您可能会决定使用 ItemID 作为排序键，或者与 UserID 一起使用。

创建数据库并切换：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

手动创建一个表（我们建议该表的架构与您想从 HDFS 加载的 Parquet 文件相同）：

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

创建表后，您可以使用 INSERT INTO SELECT FROM FILES() 来加载数据：

```SQL
INSERT INTO user_behavior_declared
SELECT * FROM FILES
(
    "path" = "hdfs://<hdfs_ip>:<hdfs_port>/user/amber/user_behavior_ten_million_rows.parquet",
    "format" = "parquet",
    "hadoop.security.authentication" = "simple",
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

加载完成后，您可以查询表以确认数据已被加载。例如：

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

您可以通过 information_schema.loads 视图查询 INSERT 作业的进度。该功能从 v3.1 版本开始支持。例如：

```SQL
SELECT * FROM information_schema.loads ORDER BY JOB_ID DESC;
```

如果您提交了多个加载作业，您可以根据作业关联的 LABEL 过滤。例如：

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

有关 `loads` 视图中提供的字段信息，请参见 [Information Schema](../reference/information_schema/loads.md)。

> **注意**
> INSERT 是一个同步命令。如果一个 INSERT 作业仍在进行中，您需要打开另一个会话来检查其执行状态。

## 使用 Broker Load

异步的 Broker Load 过程负责建立与 HDFS 的连接、拉取数据并将数据存储到 StarRocks 中。

该方法支持 Parquet、ORC 和 CSV 文件格式。

### Broker Load 的优势

- Broker Load支持在加载过程中进行[数据转换](../loading/Etl_in_loading.md)和[数据变更](../loading/Load_to_Primary_Key_tables.md)操作，比如UPSERT和DELETE。
- Broker Load 在后台运行，客户端无需保持连接即可完成作业。
- Broker Load 更适合长时间运行的作业，默认的超时时间为 4 小时。
- 除了 Parquet 和 ORC 文件格式，Broker Load 还支持 CSV 文件。

### 数据流程

![Workflow of Broker Load](../assets/broker_load_how-to-work_en.png)

1. 用户创建加载作业。
2. 前端 (FE) 创建查询计划并将计划分发给后端节点 (BE)。
3. BE 从源头拉取数据并将数据加载到 StarRocks 中。

### 典型示例

创建数据库和表，启动一个加载过程，从 HDFS 拉取数据文件 /user/amber/user_behavior_ten_million_rows.parquet，并验证数据加载的进度和成功情况。

#### 创建数据库和表

创建数据库并切换：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

手动创建一个表（我们建议该表的架构与您想从 HDFS 加载的 Parquet 文件相同）：

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

#### 启动 Broker Load

运行以下命令来启动一个 Broker Load 作业，从数据文件 /user/amber/user_behavior_ten_million_rows.parquet 加载数据到 user_behavior 表中：

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

此作业包含四个主要部分：

- LABEL：在查询加载作业状态时使用的字符串标识。
- LOAD 声明：指定源 URI、源数据格式和目标表名。
- BROKER：源的连接详情。
- PROPERTIES：超时值和其他适用于加载作业的属性。

有关详细语法和参数说明，请参阅 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

#### 检查加载进度

您可以从 information_schema.loads 视图查询 Broker Load 作业的进度。该功能从 v3.1 版本开始支持。

```SQL
SELECT * FROM information_schema.loads;
```

有关 `loads` 视图中提供的字段信息，请参见 [Information Schema](../reference/information_schema/loads.md)。

如果您提交了多个加载作业，您可以根据作业关联的 LABEL 过滤。例如：

```SQL
SELECT * FROM information_schema.loads WHERE LABEL = 'user_behavior';
```

在下方输出中，有两条关于加载作业 user_behavior 的记录：

- 第一条记录显示状态为 CANCELLED。滚动到 ERROR_MSG，您可以看到作业因 listPath 失败而未成功。
- 第二条记录显示状态为 FINISHED，意味着作业已成功完成。

```Plaintext
JOB_ID|LABEL                                      |DATABASE_NAME|STATE    |PROGRESS           |TYPE  |PRIORITY|SCAN_ROWS|FILTERED_ROWS|UNSELECTED_ROWS|SINK_ROWS|ETL_INFO|TASK_INFO                                           |CREATE_TIME        |ETL_START_TIME     |ETL_FINISH_TIME    |LOAD_START_TIME    |LOAD_FINISH_TIME   |JOB_DETAILS                                                                                                                                                                                                                                                    |ERROR_MSG                             |TRACKING_URL|TRACKING_SQL|REJECTED_RECORD_PATH|
------+-------------------------------------------+-------------+---------+-------------------+------+--------+---------+-------------+---------------+---------+--------+----------------------------------------------------+-------------------+-------------------+-------------------+-------------------+-------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------------+------------+------------+--------------------+
 10121|user_behavior                              |mydatabase   |CANCELLED|ETL:N/A; LOAD:N/A  |BROKER|NORMAL  |        0|            0|              0|        0|        |resource:N/A; timeout(s):72000; max_filter_ratio:0.0|2023-08-10 14:59:30|                   |                   |                   |2023-08-10 14:59:34|{"All backends":{},"FileNumber":0,"FileSize":0,"InternalTableLoadBytes":0,"InternalTableLoadRows":0,"ScanBytes":0,"ScanRows":0,"TaskNumber":0,"Unfinished backends":{}}                                                                                        |type:ETL_RUN_FAIL; msg:listPath failed|            |            |                    |
 10106|user_behavior                              |mydatabase   |FINISHED |ETL:100%; LOAD:100%|BROKER|NORMAL  | 86953525|            0|              0| 86953525|        |resource:N/A; timeout(s):72000; max_filter_ratio:0.0|2023-08-10 14:50:15|2023-08-10 14:50:19|2023-08-10 14:50:19|2023-08-10 14:50:19|2023-08-10 14:55:10|{"All backends":{"a5fe5e1d-d7d0-4826-ba99-c7348f9a5f2f":[10004]},"FileNumber":1,"FileSize":1225637388,"InternalTableLoadBytes":2710603082,"InternalTableLoadRows":86953525,"ScanBytes":1225637388,"ScanRows":86953525,"TaskNumber":1,"Unfinished backends":{"a5|                                      |            |            |                    |
```

确认加载作业完成后，您可以检查目标表的部分数据以确认数据是否已成功加载。例如：

```SQL
SELECT * from user_behavior LIMIT 3;
```

返回以下查询结果，表明数据已成功加载：

```Plaintext
+--------+---------+------------+--------------+---------------------+
| UserID | ItemID  | CategoryID | BehaviorType | Timestamp           |
+--------+---------+------------+--------------+---------------------+
|    142 | 2869980 |    2939262 | pv           | 2017-11-25 03:43:22 |
|    142 | 2522236 |    1669167 | pv           | 2017-11-25 15:14:12 |
|    142 | 3031639 |    3607361 | pv           | 2017-11-25 15:19:25 |
+--------+---------+------------+--------------+---------------------+
```

## 使用 Pipe

从 v3.2 版本开始，StarRocks 提供了 Pipe 加载方法，目前仅支持 Parquet 和 ORC 文件格式。

### Pipe 的优势

Pipe 非常适合连续数据加载和大规模数据加载：

- **大规模数据的微批量加载**有助于减少因数据错误导致的重试成本。

  借助 Pipe，StarRocks 能够高效加载大量的数据文件，总体数据量巨大。Pipe 会自动根据文件数量或大小分割文件，把加载作业拆分成更小的连续任务。这种方法确保单个文件的错误不会影响整个加载作业。Pipe 会记录每个文件的加载状态，方便您识别并修复包含错误的文件。通过尽量减少因数据错误而需要重试的情况，这种方法有助于降低成本。

- **连续数据加载**减少了人力需求。

  Pipe 帮助您将新的或更新的数据文件写入特定位置，并持续从这些文件中加载新数据到 StarRocks。创建了指定 "AUTO_INGEST" = "TRUE" 的 Pipe 作业后，它会不断监控指定路径中存储的数据文件的变化，并自动从这些数据文件中加载新的或更新的数据到目标 StarRocks 表中。

此外，Pipe 还执行文件唯一性检查，以防止重复加载数据。在加载过程中，Pipe 会根据文件名和摘要来检查每个数据文件的唯一性。如果某个特定文件名和摘要的文件已经被 Pipe 作业处理过，则 Pipe 作业将跳过所有具有相同文件名和摘要的后续文件。请注意，HDFS 使用 LastModifiedTime 作为文件摘要。

每个数据文件的加载状态都被记录并保存到 information_schema.pipe_files 视图中。删除与视图关联的 Pipe 作业后，该作业中加载的文件的记录也会被删除。

### 数据流程

![Pipe data flow](../assets/pipe_data_flow.png)

### Pipe 与 INSERT+FILES() 之间的区别

Pipe 作业根据每个数据文件的大小和行数分为一个或多个事务。用户可以在加载过程中查询中间结果。相比之下，INSERT+FILES() 作业作为单个事务进行处理，用户无法在加载过程中看到数据。

### 文件加载顺序

对于每个 Pipe 作业，StarRocks 维护一个文件队列，从队列中按微批次提取并加载数据文件。Pipe 不保证数据文件的加载顺序与它们上传的顺序相同。因此，较新的数据可能会先于较旧的数据被加载。

### 典型示例

#### 创建数据库和表

创建数据库并切换：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

手动创建一个表（我们建议该表的架构与您想从 HDFS 加载的 Parquet 文件相同）：

```SQL
CREATE TABLE user_behavior_replica
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

#### 启动 Pipe 作业

运行以下命令来启动一个 Pipe 作业，从数据文件 /user/amber/user_behavior_ten_million_rows.parquet 加载数据到 user_behavior_replica 表中：

```SQL
CREATE PIPE user_behavior_replica
PROPERTIES
(
    "AUTO_INGEST" = "TRUE"
)
AS
INSERT INTO user_behavior_replica
SELECT * FROM FILES
(
    "path" = "hdfs://<hdfs_ip>:<hdfs_port>/user/amber/user_behavior_ten_million_rows.parquet",
    "format" = "parquet",
    "hadoop.security.authentication" = "simple",
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
); 
```

此作业包含四个主要部分：

- pipe_name：Pipe 的名称。Pipe 名称在其所属数据库中必须唯一。
- INSERT_SQL：用于将数据从特定的源数据文件加载到目标表的 INSERT INTO SELECT FROM FILES 语句。
- PROPERTIES：一组可选参数，指定执行 Pipe 的方式。其中包括 AUTO_INGEST、POLL_INTERVAL、BATCH_SIZE 和 BATCH_FILES。以 "key" = "value" 格式指定这些属性。

有关详细语法和参数说明，请参阅[CREATE PIPE](../sql-reference/sql-statements/data-manipulation/CREATE_PIPE.md)。

#### 检查加载进度

- 使用[SHOW PIPES](../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md)查询 Pipe 作业的进度。

  ```SQL
  SHOW PIPES;
  ```

  如果您提交了多个加载作业，可以根据与作业关联的 NAME 过滤。例如：

  ```SQL
  SHOW PIPES WHERE NAME = 'user_behavior_replica' \G
  *************************** 1. row ***************************
  DATABASE_NAME: mydatabase
        PIPE_ID: 10252
      PIPE_NAME: user_behavior_replica
          STATE: RUNNING
     TABLE_NAME: mydatabase.user_behavior_replica
    LOAD_STATUS: {"loadedFiles":1,"loadedBytes":132251298,"loadingFiles":0,"lastLoadedTime":"2023-11-17 16:13:22"}
     LAST_ERROR: NULL
   CREATED_TIME: 2023-11-17 16:13:15
  1 row in set (0.00 sec)
  ```

- 从 [`information_schema.pipes`](../reference/information_schema/pipes.md) 视图中查询 Pipe 作业的进度。

  ```SQL
  SELECT * FROM information_schema.pipes;
  ```

  如果您提交了多个加载作业，可以根据与作业关联的 PIPE_NAME 过滤。例如：

  ```SQL
  SELECT * FROM information_schema.pipes WHERE pipe_name = 'user_behavior_replica' \G
  *************************** 1. row ***************************
  DATABASE_NAME: mydatabase
        PIPE_ID: 10252
      PIPE_NAME: user_behavior_replica
          STATE: RUNNING
     TABLE_NAME: mydatabase.user_behavior_replica
    LOAD_STATUS: {"loadedFiles":1,"loadedBytes":132251298,"loadingFiles":0,"lastLoadedTime":"2023-11-17 16:13:22"}
     LAST_ERROR:
   CREATED_TIME: 2023-11-17 16:13:15
  1 row in set (0.00 sec)
  ```

#### 检查文件状态

您可以通过[`information_schema.pipe_files`](../reference/information_schema/pipe_files.md)视图查询已加载文件的加载状态。

```SQL
SELECT * FROM information_schema.pipe_files;
```

如果您提交了多个加载作业，可以根据与作业关联的 PIPE_NAME 过滤。例如：

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

#### 管理 Pipe

您可以更改、暂停或恢复、删除或查询您创建的 Pipe，并重试加载特定数据文件。有关详细信息，请参见 [ALTER PIPE](../sql-reference/sql-statements/data-manipulation/ALTER_PIPE.md)、[SUSPEND 或 RESUME PIPE](../sql-reference/sql-statements/data-manipulation/SUSPEND_or_RESUME_PIPE.md)、[DROP PIPE](../sql-reference/sql-statements/data-manipulation/DROP_PIPE.md)、[SHOW PIPES](../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md) 和 [RETRY FILE](../sql-reference/sql-statements/data-manipulation/RETRY_FILE.md)。
