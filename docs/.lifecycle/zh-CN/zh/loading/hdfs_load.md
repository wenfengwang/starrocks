---
displayed_sidebar: "Chinese"
---

# 从 HDFS 导入

import LoadMethodIntro from '../assets/commonMarkdown/loadMethodIntro.md'

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocks 支持通过以下方式从 HDFS 导入数据：

<LoadMethodIntro />

## 准备工作

### 准备数据源

确保待导入数据已保存在 HDFS 集群。本文假设待导入的数据文件为 `/user/amber/user_behavior_ten_million_rows.parquet`。

### 查看权限

<InsertPrivNote />

### 获取资源访问配置

可以使用简单认证方式访问 HDFS，需要您提前获取用于访问 HDFS 集群中 NameNode 节点的用户名和密码。

## 通过 INSERT+FILES() 导入

该特性从 3.1 版本起支持。当前只支持 Parquet 和 ORC 文件格式。

### INSERT+FILES() 优势

`FILES()` 会根据给定的数据路径等参数读取数据，并自动根据数据文件的格式、列信息等推断出表结构，最终以数据行的形式返回文件中的数据。

通过 `FILES()`，您可以：

- 使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 语句直接从 HDFS 查询数据。
- 通过 [CREATE TABLE AS SELECT](../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)（简称 CTAS）语句实现自动建表和导入数据。
- 手动建表，然后通过 [INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md) 导入数据。

### 操作示例

#### 通过 SELECT 直接查询数据

您可以通过 SELECT+`FILES()` 直接查询 HDFS 里的数据，从而在建表前对待导入数据有一个整体的了解，其优势包括：

- 您不需要存储数据就可以对其进行查看。
- 您可以查看数据的最大值、最小值，并确定需要使用哪些数据类型。
- 您可以检查数据中是否包含 `NULL` 值。

例如，查询保存在 HDFS 中的数据文件 `/user/amber/user_behavior_ten_million_rows.parquet`：

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

系统返回如下查询结果：

```Plaintext
+--------+---------+------------+--------------+---------------------+
| UserID | ItemID  | CategoryID | BehaviorType | Timestamp           |
+--------+---------+------------+--------------+---------------------+
| 543711 |  829192 |    2355072 | pv           | 2017-11-27 08:22:37 |
| 543711 | 2056618 |    3645362 | pv           | 2017-11-27 10:16:46 |
| 543711 | 1165492 |    3645362 | pv           | 2017-11-27 10:17:00 |
+--------+---------+------------+--------------+---------------------+
```

> **说明**
>
> 以上返回结果中的列名是源 Parquet 文件中定义的列名。

#### 通过 CTAS 自动建表并导入数据

该示例是上一个示例的延续。该示例中，通过在 CREATE TABLE AS SELECT (CTAS) 语句中嵌套上一个示例中的 SELECT 查询，StarRocks 可以自动推断表结构、创建表、并把数据导入新建的表中。Parquet 格式的文件自带列名和数据类型，因此您不需要指定列名或数据类型。

> **说明**
>
> 使用表结构推断功能时，CREATE TABLE 语句不支持设置副本数，因此您需要在建表前把副本数设置好。例如，您可以通过如下命令设置副本数为 `1`：
>
> ```SQL
> ADMIN SET FRONTEND CONFIG ('default_replication_num' = "1");
> ```

通过如下语句创建数据库、并切换至该数据库：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

通过 CTAS 自动创建表、并把数据文件 `/user/amber/user_behavior_ten_million_rows.parquet` 中的数据导入到新建表中：

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

建表完成后，您可以通过 [DESCRIBE](../sql-reference/sql-statements/Utility/DESCRIBE.md) 查看新建表的表结构：

```SQL
DESCRIBE user_behavior_inferred;
```

系统返回如下查询结果：

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

将系统推断出来的表结构跟手动建表的表结构从以下几个方面进行对比：

- 数据类型
- 是否允许 `NULL` 值
- 定义为键的字段

在生产环境中，为更好地控制目标表的表结构、实现更高的查询性能，建议您手动创建表、指定表结构。

您可以查询新建表中的数据，验证数据已成功导入。例如：

```SQL
SELECT * from user_behavior_inferred LIMIT 3;
```

系统返回如下查询结果，表明数据已成功导入：

```Plaintext
+--------+--------+------------+--------------+---------------------+
| UserID | ItemID | CategoryID | BehaviorType | Timestamp           |
+--------+--------+------------+--------------+---------------------+
|     84 |  56257 |    1879194 | pv           | 2017-11-26 05:56:23 |
|     84 | 108021 |    2982027 | pv           | 2017-12-02 05:43:00 |
|     84 | 390657 |    1879194 | pv           | 2017-11-28 11:20:30 |
+--------+--------+------------+--------------+---------------------+
```

#### 手动建表并通过 INSERT 导入数据

在实际业务场景中，您可能需要自定义目标表的表结构，包括：

- 各列的数据类型和默认值、以及是否允许 `NULL` 值
- 定义哪些列作为键、以及这些列的数据类型
- 数据分区分桶

> **说明**
>
> 要实现高效的表结构设计，您需要深度了解表中数据的用途、以及表中各列的内容。本文不对表设计做过多赘述，有关表设计的详细信息，参见[表设计](../table_design/StarRocks_table_design.md)。

该示例主要演示如何根据源 Parquet 格式文件中数据的特点、以及目标表未来的查询用途等对目标表进行定义和创建。在创建表之前，您可以先查看一下保存在 HDFS 中的源文件，从而了解源文件中数据的特点，例如：

- 源文件中包含一个数据类型为 `datetime` 的 `Timestamp` 列，因此建表语句中也应该定义这样一个数据类型为 `datetime` 的 `Timestamp` 列。
- 源文件中的数据中没有 `NULL` 值，因此建表语句中也不需要定义任何列为允许 `NULL` 值。
- 根据查询到的数据类型，可以在建表语句中定义 `UserID` 列为排序键和分桶键。根据实际业务场景需要，您还可以定义其他列比如 `ItemID` 或者定义 `UserID` 与其他列的组合作为排序键。

通过如下语句创建数据库，并切换至该数据库：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

通过如下语句手动创建表（建议表结构与您在 HDFS 存储的待导入数据结构一致）：

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

建表完成后，您可以通过 INSERT INTO SELECT FROM FILES() 向表内导入数据：

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

导入完成后，您可以查询新建表中的数据，验证数据已成功导入。例如：

```SQL
SELECT * from user_behavior_declared LIMIT 3;
```

系统返回如下查询结果，表明数据已成功导入：

```Plaintext
+--------+---------+------------+--------------+---------------------+
| UserID | ItemID  | CategoryID | BehaviorType | Timestamp           |
+--------+---------+------------+--------------+---------------------+
|    107 | 1568743 |    4476428 | pv           | 2017-11-25 14:29:53 |
|    107 |  470767 |    1020087 | pv           | 2017-11-25 14:32:31 |
|    107 |  358238 |    1817004 | pv           | 2017-11-25 14:43:23 |
+--------+---------+------------+--------------+---------------------+
```

#### 查看导入进度

通过 `information_schema.loads` 视图查看导入作业的进度。该功能自 3.1 版本起支持。例如：

```SQL
SELECT * FROM information_schema.loads ORDER BY JOB_ID DESC;
```

如果您提交了多个导入作业，您可以通过 `LABEL` 过滤出想要查看的作业。例如：

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

有关 `loads` 视图提供的字段详情，参见 [Information Schema](../reference/information_schema/loads.md)。

> **NOTE**
>
> 由于 INSERT 语句是一个同步命令，因此，如果作业还在运行当中，您需要打开另一个会话来查看 INSERT 作业的执行情况。

## 通过 Broker Load 导入

作为一种异步的导入方式，Broker Load 负责建立与 HDFS 的连接、拉取数据、并将数据存储到 StarRocks 中。

当前支持 Parquet、ORC、及 CSV 三种文件格式。

### Broker Load 优势

- Broker Load 支持在导入过程中执行[数据转换](../loading/Etl_in_loading.md)、以及 [UPSERT、DELETE 等数据变更操作](../loading/Load_to_Primary_Key_tables.md)。
- Broker Load 在后台运行，客户端不需要保持连接也能确保导入作业不中断。
- Broker Load 作业默认超时时间为 4 小时，适合导入数据较大、导入运行时间较长的场景。
- 除 Parquet 和 ORC 文件格式，Broker Load 还支持 CSV 文件格式。

### 工作原理

![Broker Load 原理图](../assets/broker_load_how-to-work_zh.png)

1. 用户创建导入作业。
2. FE 生成查询计划，然后把查询计划拆分并分分配给各个 BE 执行。
3. 各个 BE 从数据源拉取数据并把数据导入到 StarRocks 中。

### 操作示例

创建 StarRocks 表，启动导入作业从 HDFS 集群拉取数据文件 `/user/amber/user_behavior_ten_million_rows.parquet` 的数据，然后验证导入过程和结果是否成功。

#### 建库建表

通过如下语句创建数据库，并切换至该数据库：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

通过如下语句手动创建表（建议表结构与您在 HDFS 集群中存储的待导入数据结构一致）：

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

#### 提交导入作业

执行如下命令创建 Broker Load 作业，把数据文件 `/user/amber/user_behavior_ten_million_rows.parquet` 中的数据导入到表 `user_behavior`中：

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

- `LOAD` statement: Includes job description information such as the URI of the source data file, the format of the source data file, and the name of the target table.
- `BROKER`: Connection authentication information configuration for connecting to the data source.
- `PROPERTIES`: Used to specify optional job properties such as timeout.

For detailed syntax and parameter descriptions, refer [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md).

#### View Import Progress

View the progress of the import job through the `information_schema.loads` view. This feature is supported from version 3.1 onwards.

```SQL
SELECT * FROM information_schema.loads;
```

For detailed information about the fields provided by the `loads` view, refer to [Information Schema](../reference/information_schema/loads.md).

If you have submitted multiple import jobs, you can filter out the jobs you want to view by using `LABEL`. For example:

```SQL
SELECT * FROM information_schema.loads WHERE LABEL = 'user_behavior';
```

For example, in the returned results below, there are two records related to the import job `user_behavior`:

- The first record shows the status of the import job as `CANCELLED`. Through the `ERROR_MSG` field in the record, it can be determined that the reason for the job failure is `listPath failed`.
- The second record shows the status of the import job as `FINISHED`, indicating a successful job.

```Plaintext
JOB_ID|LABEL                                      |DATABASE_NAME|STATE    |PROGRESS           |TYPE  |PRIORITY|SCAN_ROWS|FILTERED_ROWS|UNSELECTED_ROWS|SINK_ROWS|ETL_INFO|TASK_INFO                                           |CREATE_TIME        |ETL_START_TIME     |ETL_FINISH_TIME    |LOAD_START_TIME    |LOAD_FINISH_TIME   |JOB_DETAILS                                                                                                                                                                                                                                                    |ERROR_MSG                             |TRACKING_URL|TRACKING_SQL|REJECTED_RECORD_PATH|
------+-------------------------------------------+-------------+---------+-------------------+------+--------+---------+-------------+---------------+---------+--------+----------------------------------------------------+-------------------+-------------------+-------------------+-------------------+-------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------------+------------+------------+--------------------+
 10121|user_behavior                              |mydatabase   |CANCELLED|ETL:N/A; LOAD:N/A  |BROKER|NORMAL  |        0|            0|              0|        0|        |resource:N/A; timeout(s):72000; max_filter_ratio:0.0|2023-08-10 14:59:30|                   |                   |                   |2023-08-10 14:59:34|{"All backends":{},"FileNumber":0,"FileSize":0,"InternalTableLoadBytes":0,"InternalTableLoadRows":0,"ScanBytes":0,"ScanRows":0,"TaskNumber":0,"Unfinished backends":{}}                                                                                        |type:ETL_RUN_FAIL; msg:listPath failed|            |            |                    |
 10106|user_behavior                              |mydatabase   |FINISHED |ETL:100%; LOAD:100%|BROKER|NORMAL  | 86953525|            0|              0| 86953525|        |resource:N/A; timeout(s):72000; max_filter_ratio:0.0|2023-08-10 14:50:15|2023-08-10 14:50:19|2023-08-10 14:50:19|2023-08-10 14:50:19|2023-08-10 14:55:10|{"All backends":{"a5fe5e1d-d7d0-4826-ba99-c7348f9a5f2f":[10004]},"FileNumber":1,"FileSize":1225637388,"InternalTableLoadBytes":2710603082,"InternalTableLoadRows":86953525,"ScanBytes":1225637388,"ScanRows":86953525,"TaskNumber":1,"Unfinished backends":{"a5|                                      |            |            |                    |
```

After the import job is completed, you can query the table to verify whether the data has been successfully imported. For example:

```SQL
SELECT * from user_behavior LIMIT 3;
```

The system returns the following query results, indicating that the data has been successfully imported:

```Plaintext
+--------+--------+------------+--------------+---------------------+
| UserID | ItemID | CategoryID | BehaviorType | Timestamp           |
+--------+--------+------------+--------------+---------------------+
|     58 | 158350 |    2355072 | pv           | 2017-11-27 13:06:51 |
|     58 | 158590 |    3194735 | pv           | 2017-11-27 02:21:04 |
|     58 | 215073 |    3002561 | pv           | 2017-11-30 10:55:42 |
+--------+--------+------------+--------------+---------------------+
```

## Import via Pipe

Starting from version 3.2, StarRocks provides the Pipe importing method, which currently supports Parquet and ORC file formats.

### Advantages of Pipe

Pipe is suitable for scenarios where data needs to be imported in large-scale batches or continuously:

- **Large-scale batch import, reducing error retry costs.**

  When there are many data files to be imported with a large amount of data, Pipe will automatically split the files in the directory into multiple smaller serial import tasks based on the number or size of the files. Errors in individual files will not cause the entire import job to fail. In addition, Pipe will record the import status of each file. After the import is completed, you can repair the erroneous data file and then re-import the corrected data file. This helps to reduce the cost of error retry.

- **Continuous import without interruption, reducing manual operation costs.**

  When new or changed data files need to be written to a specific folder, and the new data needs to be continuously imported into StarRocks, you only need to create a continuous import job based on Pipe (specify `"AUTO_INGEST" = "TRUE"` in the statement). This Pipe will continuously monitor the changes in the data files under the specified path in the job and automatically import the new or modified data files into the target table in StarRocks.

In addition, Pipe also supports file uniqueness verification to avoid duplicate data import. During the import process, Pipe will determine whether the data file is duplicated based on the file name and the corresponding digest value of the file. If the file name and file digest value have been processed in the same Pipe import job, later imports will automatically skip the files that have already been processed. Note that HDFS uses `LastModifiedTime` as the file digest.

The file status during the import process will be recorded in the `information_schema.pipe_files` view. You can use this view to check the import status of files under the Pipe import job. If the Pipe job associated with this view is deleted, the related records in this view will also be cleaned up synchronously.

### Working Principle

![Pipe Working Principle](../assets/pipe_data_flow.png)

### Difference Between Pipe and INSERT+FILES()

The Pipe import operation will split into one or more transactions based on the size and number of rows contained in each data file, and the intermediate results of the import process will be visible to the user. The INSERT+`FILES()` import operation is a single transaction, and the data is not visible to the user during the import process.

### File Import Order

The Pipe import operation maintains a file queue internally, and fetches the corresponding files from the queue for import in batches. Pipe does not guarantee that the import order of the files is consistent with the file upload order, so it may happen that new data is imported before old data.

### Operation Example

#### Database and Table Creation

Create a database and switch to the database using the following statement:

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

Manually create the table using the following statement (it is recommended that the table structure matches the structure of the data to be imported on HDFS):

```SQL
CREATE TABLE user_behavior_replica
(
    UserID int(11),
```
      + ItemID int(11),
      + CategoryID int(11),
    + BehaviorType varchar(65533),
  + Timestamp datetime
)
ENGINE = OLAP 
DUPLICATE KEY(UserID)
DISTRIBUTED BY HASH(UserID)
PROPERTIES
(
    "replication_num" = "1"
);
```

#### Submit Import Job

Execute the following command to create a Pipe job and import data from the data file `/user/amber/user_behavior_ten_million_rows.parquet` into the table `user_behavior_replica`:

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

The import statement contains four parts:

- `pipe_name`: The name of the Pipe. This name must be unique within the database where the Pipe resides.
- `INSERT_SQL`: The INSERT INTO SELECT FROM FILES statement is used to import data from the specified source data file into the target table.
- `PROPERTIES`: Used to control some parameters for Pipe execution, such as `AUTO_INGEST`, `POLL_INTERVAL`, `BATCH_SIZE`, and `BATCH_FILES`, in the format of `"key" = "value"`.

For detailed syntax and parameter descriptions, refer to [CREATE PIPE](../sql-reference/sql-statements/data-manipulation/CREATE_PIPE.md).

#### Check Import Progress

- Use [SHOW PIPES](../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md) to view the import jobs in the current database.

  ```SQL
  SHOW PIPES;
  ```

  If you have submitted multiple import jobs, you can use `NAME` to filter the specific import job you want to view. For example:

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

- Use the [`information_schema.pipes`](../reference/information_schema/pipes.md) view to check the import jobs in the current database.

  ```SQL
  SELECT * FROM information_schema.pipes;
  ```

  If you have submitted multiple import jobs, you can use `PIPE_NAME` to filter the specific import job you want to view. For example:

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

#### Check Imported File Information

You can use the [`information_schema.pipe_files`](../reference/information_schema/pipe_files.md) view to check the information of the imported files.

```SQL
SELECT * FROM information_schema.pipe_files;
```

If you have submitted multiple import jobs, you can use `PIPE_NAME` to filter the specific job for which you want to view the file information. For example:

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

#### Manage Import Jobs

After creating Pipe import jobs, you can modify, suspend or resume, delete, query, and attempt re-importation of these jobs as needed. For details, see [ALTER PIPE](../sql-reference/sql-statements/data-manipulation/ALTER_PIPE.md), [SUSPEND or RESUME PIPE](../sql-reference/sql-statements/data-manipulation/SUSPEND_or_RESUME_PIPE.md), [DROP PIPE](../sql-reference/sql-statements/data-manipulation/DROP_PIPE.md), [SHOW PIPES](../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md), [RETRY FILE](../sql-reference/sql-statements/data-manipulation/RETRY_FILE.md).