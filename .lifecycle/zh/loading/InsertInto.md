---
displayed_sidebar: English
---

# 使用 INSERT 命令加载数据

从 '../assets/commonMarkdown/insertPrivNote.md' 导入 InsertPrivNote

本主题介绍如何使用 SQL 语句 - INSERT 将数据加载到 StarRocks 中。

与 MySQL 和许多其他数据库管理系统一样，StarRocks 支持使用 INSERT 将数据加载到内部表中。您可以使用 VALUES 子句直接插入一行或多行数据以测试功能或进行演示。您还可以将外部表的查询结果定义的数据插入到内部表中从[外部表](../data_source/External_table.md)。从 StarRocks v3.1 版本开始，您可以直接使用 INSERT 命令和表函数[FILES()](../sql-reference/sql-functions/files.md)从云存储上的文件加载数据。

StarRocks v2.4 版本进一步支持使用 INSERT OVERWRITE 命令覆盖表中的数据。INSERT OVERWRITE 语句集成了以下操作来实现覆盖功能：

1. 根据存储原始数据的分区创建临时分区。
2. 将数据插入临时分区。
3. 用临时分区替换原始分区。

> **注意**
> 如果您需要在覆盖数据之前验证数据，而不是使用 INSERT OVERWRITE，您可以按照上述步骤覆盖数据并在替换分区之前进行验证。

## 注意事项

- 您可以通过在 MySQL 客户端按下 **Ctrl** 和 **C** 键来取消同步 INSERT 事务。
- 您可以使用 [SUBMIT TASK](../sql-reference/sql-statements/data-manipulation/SUBMIT_TASK.md) 命令提交异步 INSERT 任务。
- 对于 StarRocks 当前版本，如果任何行的数据不符合表的模式，则 INSERT 事务默认会失败。例如，如果任何行中的字段长度超出了表中相应字段的长度限制，INSERT 事务将失败。您可以将会话变量 enable_insert_strict 设置为 false，以允许事务继续执行，并过滤掉与表不匹配的行。
- 如果您频繁地执行 `INSERT` 语句以将小批量数据加载到 StarRocks 中，会产生过多的数据版本，这将严重影响查询性能。我们建议在生产环境中，不应过于频繁地使用 `INSERT` 命令加载数据，或将其作为日常数据加载的常规方法。如果您的应用程序或分析场景需要单独加载流式数据或小批量数据的解决方案，我们建议您使用 Apache Kafka® 作为数据源，并通过 [Routine Load](../loading/RoutineLoad.md) 功能加载数据。
- 如果执行`INSERT OVERWRITE`语句，StarRocks会为存储原始数据的分区创建临时分区，插入新数据到临时分区，并[将原始分区与临时分区交换](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md#use-a-temporary-partition-to-replace-current-partition)。所有这些操作都在FE Leader节点上执行。因此，如果FE Leader节点在执行`INSERT OVERWRITE`命令时崩溃，整个加载事务将失败，临时分区也将被删除。

## 准备工作

### 检查权限

<InsertPrivNote />


### 创建对象

创建一个名为 load_test 的数据库，并创建一个作为目标表的 insert_wiki_edit 表和一个作为源表的 source_wiki_edit 表。

> **注意**
> 本主题中展示的示例基于表 `insert_wiki_edit` 和表 `source_wiki_edit`。如果您更愿意使用自己的表和数据，可以跳过准备步骤，直接进入下一步。

```SQL
CREATE DATABASE IF NOT EXISTS load_test;
USE load_test;
CREATE TABLE insert_wiki_edit
(
    event_time      DATETIME,
    channel         VARCHAR(32)      DEFAULT '',
    user            VARCHAR(128)     DEFAULT '',
    is_anonymous    TINYINT          DEFAULT '0',
    is_minor        TINYINT          DEFAULT '0',
    is_new          TINYINT          DEFAULT '0',
    is_robot        TINYINT          DEFAULT '0',
    is_unpatrolled  TINYINT          DEFAULT '0',
    delta           INT              DEFAULT '0',
    added           INT              DEFAULT '0',
    deleted         INT              DEFAULT '0'
)
DUPLICATE KEY(
    event_time,
    channel,
    user,
    is_anonymous,
    is_minor,
    is_new,
    is_robot,
    is_unpatrolled
)
PARTITION BY RANGE(event_time)(
    PARTITION p06 VALUES LESS THAN ('2015-09-12 06:00:00'),
    PARTITION p12 VALUES LESS THAN ('2015-09-12 12:00:00'),
    PARTITION p18 VALUES LESS THAN ('2015-09-12 18:00:00'),
    PARTITION p24 VALUES LESS THAN ('2015-09-13 00:00:00')
)
DISTRIBUTED BY HASH(user);

CREATE TABLE source_wiki_edit
(
    event_time      DATETIME,
    channel         VARCHAR(32)      DEFAULT '',
    user            VARCHAR(128)     DEFAULT '',
    is_anonymous    TINYINT          DEFAULT '0',
    is_minor        TINYINT          DEFAULT '0',
    is_new          TINYINT          DEFAULT '0',
    is_robot        TINYINT          DEFAULT '0',
    is_unpatrolled  TINYINT          DEFAULT '0',
    delta           INT              DEFAULT '0',
    added           INT              DEFAULT '0',
    deleted         INT              DEFAULT '0'
)
DUPLICATE KEY(
    event_time,
    channel,user,
    is_anonymous,
    is_minor,
    is_new,
    is_robot,
    is_unpatrolled
)
PARTITION BY RANGE(event_time)(
    PARTITION p06 VALUES LESS THAN ('2015-09-12 06:00:00'),
    PARTITION p12 VALUES LESS THAN ('2015-09-12 12:00:00'),
    PARTITION p18 VALUES LESS THAN ('2015-09-12 18:00:00'),
    PARTITION p24 VALUES LESS THAN ('2015-09-13 00:00:00')
)
DISTRIBUTED BY HASH(user);
```

> **通知**
> 从 v2.5.7 版本开始，StarRocks在创建表或添加分区时可以自动设置桶（BUCKETS）的数量，您不再需要手动设置桶数。有关详细信息，请参阅[determine the number of buckets](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

## 通过 INSERT INTO VALUES 插入数据

您可以使用 `INSERT INTO VALUES` 命令向特定表追加一行或多行数据。多行数据之间用逗号 (,) 分隔。有关详细说明和参数参考，请参见 [SQL 参考 - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)。

> **警告**
> 使用 INSERT INTO VALUES 插入数据仅适用于需要用小数据集验证演示的情况。不建议在大规模测试或生产环境中使用。要将大量数据加载到 StarRocks，请参阅[数据摄取概览](../loading/Loading_intro.md)，了解适合您场景的其他选项。

以下示例将两行数据插入到带有标签 insert_load_wikipedia 的数据源表 source_wiki_edit 中。标签是数据库中每个数据加载事务的唯一标识。

```SQL
INSERT INTO source_wiki_edit
WITH LABEL insert_load_wikipedia
VALUES
    ("2015-09-12 00:00:00","#en.wikipedia","AustinFF",0,0,0,0,0,21,5,0),
    ("2015-09-12 00:00:00","#ca.wikipedia","helloSR",0,1,0,1,0,3,23,0);
```

## 通过 INSERT INTO SELECT 插入数据

您可以使用 INSERT INTO SELECT 命令将数据源表的查询结果加载到目标表中。INSERT INTO SELECT 命令执行ETL操作对数据源表的数据进行ETL操作，并将数据加载到StarRocks的内部表中。数据源可以是一个或多个内部或外部表，甚至是云存储上的数据文件。目标表**必须**是StarRocks中的内部表。有关详细说明和参数参考，请参见[SQL 参考 - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)。

### 将内部或外部表的数据插入到内部表中

> **注意**
> 从外部表插入数据的操作与从内部表插入数据的操作相同。为了简化，我们在以下示例中仅展示如何从内部表插入数据。

- 以下示例将数据从源表插入到目标表 insert_wiki_edit。

```SQL
INSERT INTO insert_wiki_edit
WITH LABEL insert_load_wikipedia_1
SELECT * FROM source_wiki_edit;
```

- 以下示例将数据从源表插入到目标表 insert_wiki_edit 的 p06 和 p12 分区中。如果没有指定分区，则数据将被插入到所有分区。否则，数据仅会被插入到指定的分区中。

```SQL
INSERT INTO insert_wiki_edit PARTITION(p06, p12)
WITH LABEL insert_load_wikipedia_2
SELECT * FROM source_wiki_edit;
```

查询目标表以确认其中有数据。

```Plain
MySQL > select * from insert_wiki_edit;
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| event_time          | channel       | user     | is_anonymous | is_minor | is_new | is_robot | is_unpatrolled | delta | added | deleted |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #en.wikipedia | AustinFF |            0 |        0 |      0 |        0 |              0 |    21 |     5 |       0 |
| 2015-09-12 00:00:00 | #ca.wikipedia | helloSR  |            0 |        1 |      0 |        1 |              0 |     3 |    23 |       0 |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 rows in set (0.00 sec)
```

如果您截断了 p06 和 p12 分区，查询将不会返回任何数据。

```Plain
MySQL > TRUNCATE TABLE insert_wiki_edit PARTITION(p06, p12);
Query OK, 0 rows affected (0.01 sec)

MySQL > select * from insert_wiki_edit;
Empty set (0.00 sec)
```

- 以下示例将源表的 event_time 和 channel 列的数据插入到目标表 insert_wiki_edit 中。未指定的列将使用默认值。

```SQL
INSERT INTO insert_wiki_edit
WITH LABEL insert_load_wikipedia_3 
(
    event_time, 
    channel
)
SELECT event_time, channel FROM source_wiki_edit;
```

### 使用 FILES() 从外部源文件直接加载数据

从 v3.1 版本开始，StarRocks 支持使用 INSERT 命令和 [FILES()](../sql-reference/sql-functions/table-functions/files.md) 函数直接从云存储中的文件加载数据，无需首先创建外部目录或文件外部表。此外，FILES() 可以自动推断文件的表结构，极大简化了数据加载过程。

以下示例将数据行从 Parquet 文件 **parquet/insert_wiki_edit_append.parquet** 插入到表 `insert_wiki_edit` 中，该文件位于 AWS S3 存储桶 `inserttest` 中：

```Plain
INSERT INTO insert_wiki_edit
    SELECT * FROM FILES(
        "path" = "s3://inserttest/parquet/insert_wiki_edit_append.parquet",
        "format" = "parquet",
        "aws.s3.access_key" = "XXXXXXXXXX",
        "aws.s3.secret_key" = "YYYYYYYYYY",
        "aws.s3.region" = "us-west-2"
);
```

## 通过 INSERT OVERWRITE VALUES 覆盖数据

您可以使用 INSERT OVERWRITE VALUES 命令覆盖特定表中的一行或多行数据。多行数据之间用逗号 (,) 分隔。有关详细说明和参数参见 [SQL 参考 - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)。

> **警告**
> 使用 INSERT OVERWRITE VALUES 覆盖数据仅适用于需要用小数据集验证演示的情况。不建议在大规模测试或生产环境中使用。要将大量数据加载到 StarRocks，请参阅[数据摄取概览](../loading/Loading_intro.md)，了解适合您场景的其他选项。

查询源表和目标表以确认其中有数据。

```Plain
MySQL > SELECT * FROM source_wiki_edit;
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| event_time          | channel       | user     | is_anonymous | is_minor | is_new | is_robot | is_unpatrolled | delta | added | deleted |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #ca.wikipedia | helloSR  |            0 |        1 |      0 |        1 |              0 |     3 |    23 |       0 |
| 2015-09-12 00:00:00 | #en.wikipedia | AustinFF |            0 |        0 |      0 |        0 |              0 |    21 |     5 |       0 |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 rows in set (0.02 sec)
 
MySQL > SELECT * FROM insert_wiki_edit;
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| event_time          | channel       | user     | is_anonymous | is_minor | is_new | is_robot | is_unpatrolled | delta | added | deleted |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #ca.wikipedia | helloSR  |            0 |        1 |      0 |        1 |              0 |     3 |    23 |       0 |
| 2015-09-12 00:00:00 | #en.wikipedia | AustinFF |            0 |        0 |      0 |        0 |              0 |    21 |     5 |       0 |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 rows in set (0.01 sec)
```

以下示例使用两个新行覆盖源表 source_wiki_edit。

```SQL
INSERT OVERWRITE source_wiki_edit
WITH LABEL insert_load_wikipedia_ow
VALUES
    ("2015-09-12 00:00:00","#cn.wikipedia","GELongstreet",0,0,0,0,0,36,36,0),
    ("2015-09-12 00:00:00","#fr.wikipedia","PereBot",0,1,0,1,0,17,17,0);
```

## 通过 INSERT OVERWRITE SELECT 覆盖数据

您可以使用 INSERT OVERWRITE SELECT 命令覆盖表中的数据，这些数据是基于数据源表的查询结果。INSERT OVERWRITE SELECT 语句对一个或多个内部或外部表的数据执行 ETL 操作，并用这些数据覆盖内部表。有关详细说明和参数参考，请参见 [SQL 参考 - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)。

> **注意**
> 从外部表加载数据的操作与从内部表加载数据的操作相同。为了简化，我们在以下示例中仅展示如何使用内部表的数据覆盖目标表。

查询源表和目标表以确认它们包含不同行的数据。

```Plain
MySQL > SELECT * FROM source_wiki_edit;
+---------------------+---------------+--------------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| event_time          | channel       | user         | is_anonymous | is_minor | is_new | is_robot | is_unpatrolled | delta | added | deleted |
+---------------------+---------------+--------------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #cn.wikipedia | GELongstreet |            0 |        0 |      0 |        0 |              0 |    36 |    36 |       0 |
| 2015-09-12 00:00:00 | #fr.wikipedia | PereBot      |            0 |        1 |      0 |        1 |              0 |    17 |    17 |       0 |
+---------------------+---------------+--------------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 rows in set (0.02 sec)
 
MySQL > SELECT * FROM insert_wiki_edit;
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| event_time          | channel       | user     | is_anonymous | is_minor | is_new | is_robot | is_unpatrolled | delta | added | deleted |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #en.wikipedia | AustinFF |            0 |        0 |      0 |        0 |              0 |    21 |     5 |       0 |
| 2015-09-12 00:00:00 | #ca.wikipedia | helloSR  |            0 |        1 |      0 |        1 |              0 |     3 |    23 |       0 |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 rows in set (0.01 sec)
```

- 以下示例使用源表的数据覆盖目标表 insert_wiki_edit。

```SQL
INSERT OVERWRITE insert_wiki_edit
WITH LABEL insert_load_wikipedia_ow_1
SELECT * FROM source_wiki_edit;
```

- 以下示例使用源表的数据覆盖目标表 insert_wiki_edit 的 p06 和 p12 分区。

```SQL
INSERT OVERWRITE insert_wiki_edit PARTITION(p06, p12)
WITH LABEL insert_load_wikipedia_ow_2
SELECT * FROM source_wiki_edit;
```

查询目标表以确认其中有数据。

```plain
MySQL > select * from insert_wiki_edit;
+---------------------+---------------+--------------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| event_time          | channel       | user         | is_anonymous | is_minor | is_new | is_robot | is_unpatrolled | delta | added | deleted |
+---------------------+---------------+--------------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #fr.wikipedia | PereBot      |            0 |        1 |      0 |        1 |              0 |    17 |    17 |       0 |
| 2015-09-12 00:00:00 | #cn.wikipedia | GELongstreet |            0 |        0 |      0 |        0 |              0 |    36 |    36 |       0 |
+---------------------+---------------+--------------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 rows in set (0.01 sec)
```

如果您截断了 p06 和 p12 分区，查询将不会返回任何数据。

```Plain
MySQL > TRUNCATE TABLE insert_wiki_edit PARTITION(p06, p12);
Query OK, 0 rows affected (0.01 sec)

MySQL > select * from insert_wiki_edit;
Empty set (0.00 sec)
```

- 以下示例将源表的 event_time 和 channel 列的数据覆盖到目标表 insert_wiki_edit 中，未覆盖的列将被赋予默认值。

```SQL
INSERT OVERWRITE insert_wiki_edit
WITH LABEL insert_load_wikipedia_ow_3 
(
    event_time, 
    channel
)
SELECT event_time, channel FROM source_wiki_edit;
```

## 将数据插入带有生成列的表中

生成列是一种特殊的列，其值是基于其他列的预定义表达式或计算得出的。当您的查询涉及昂贵的表达式计算时，例如，从 JSON 值中查询特定字段或计算 ARRAY 数据，生成列尤其有用。当数据被加载到表中时，StarRocks 计算表达式并将结果存储在生成列中，从而避免了在查询过程中进行表达式计算，提高了查询性能。

您可以使用 INSERT 命令将数据加载到包含生成列的表中。

以下示例创建了一个包含两个生成列 avg_array 和 get_string 的表 insert_generated_columns，并向其中插入了一行数据。avg_array 计算 data_array 中 ARRAY 数据的平均值，get_string 从 data_json 中的 JSON 路径 a 提取字符串。

```SQL
CREATE TABLE insert_generated_columns (
  id           INT(11)           NOT NULL    COMMENT "ID",
  data_array   ARRAY<INT(11)>    NOT NULL    COMMENT "ARRAY",
  data_json    JSON              NOT NULL    COMMENT "JSON",
  avg_array    DOUBLE            NULL 
      AS array_avg(data_array)               COMMENT "Get the average of ARRAY",
  get_string   VARCHAR(65533)    NULL 
      AS get_json_string(json_string(data_json), '$.a') COMMENT "Extract JSON string"
) ENGINE=OLAP 
PRIMARY KEY(id)
DISTRIBUTED BY HASH(id);

INSERT INTO insert_generated_columns 
VALUES (1, [1,2], parse_json('{"a" : 1, "b" : 2}'));
```

> **注意**
> 不支持直接将数据加载到**生成列**中。

您可以查询表来检查其中的数据。

```Plain
mysql> SELECT * FROM insert_generated_columns;
+------+------------+------------------+-----------+------------+
| id   | data_array | data_json        | avg_array | get_string |
+------+------------+------------------+-----------+------------+
|    1 | [1,2]      | {"a": 1, "b": 2} |       1.5 | 1          |
+------+------------+------------------+-----------+------------+
1 row in set (0.02 sec)
```

## 使用 INSERT 异步加载数据

使用 INSERT 加载数据会提交同步事务，可能会因会话中断或超时而失败。您可以提交一个异步的 INSERT 事务，使用[SUBMIT TASK](../sql-reference/sql-statements/data-manipulation/SUBMIT_TASK.md)。这项功能自 StarRocks v2.5 版本起提供支持。

- 以下示例将源表中的数据异步插入到目标表 insert_wiki_edit 中。

```SQL
SUBMIT TASK AS INSERT INTO insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

- 以下示例使用源表中的数据异步覆盖目标表 insert_wiki_edit。

```SQL
SUBMIT TASK AS INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

- 以下示例使用源表中的数据异步覆盖目标表 insert_wiki_edit，并使用提示将查询超时设置为 100000 秒。

```SQL
SUBMIT /*+set_var(query_timeout=100000)*/ TASK AS
INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

- 以下示例使用源表中的数据异步覆盖目标表 insert_wiki_edit，并将任务名称指定为 async。

```SQL
SUBMIT TASK async
AS INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

您可以通过查询信息模式（Information Schema）中的元数据视图 task_runs 来检查异步 INSERT 任务的状态。

以下示例检查异步 INSERT 任务的状态。

```SQL
SELECT * FROM information_schema.task_runs WHERE task_name = 'async';
```

## 检查 INSERT 作业状态

### 通过结果检查

同步 INSERT 事务根据事务的结果返回不同的状态。

- **交易成功**

如果交易成功，StarRocks 将返回以下内容：

```Plain
Query OK, 2 rows affected (0.05 sec)
{'label':'insert_load_wikipedia', 'status':'VISIBLE', 'txnId':'1006'}
```

- **交易失败**

如果所有数据行都无法加载到目标表中，INSERT 事务将失败。如果交易失败，StarRocks 将返回以下内容：

```Plain
ERROR 1064 (HY000): Insert has filtered data in strict mode, tracking_url=http://x.x.x.x:yyyy/api/_load_error_log?file=error_log_9f0a4fd0b64e11ec_906bbede076e9d08
```

您可以通过 tracking_url 查看日志来定位问题。

### 通过信息模式（Information Schema）检查

您可以使用[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)语句从`information_schema`数据库的`loads`表中查询一个或多个加载作业的结果。这项功能自v3.1版本起提供支持。

示例 1：查询在 load_test 数据库上执行的加载作业的结果，按创建时间（CREATE_TIME）降序排序，仅返回最新的结果。

```SQL
SELECT * FROM information_schema.loads
WHERE database_name = 'load_test'
ORDER BY create_time DESC
LIMIT 1\G
```

示例 2：查询在 load_test 数据库上执行的、标签为 insert_load_wikipedia 的加载作业的结果：

```SQL
SELECT * FROM information_schema.loads
WHERE database_name = 'load_test' and label = 'insert_load_wikipedia'\G
```

返回结果如下：

```Plain
*************************** 1. row ***************************
              JOB_ID: 21319
               LABEL: insert_load_wikipedia
       DATABASE_NAME: load_test
               STATE: FINISHED
            PROGRESS: ETL:100%; LOAD:100%
                TYPE: INSERT
            PRIORITY: NORMAL
           SCAN_ROWS: 0
       FILTERED_ROWS: 0
     UNSELECTED_ROWS: 0
           SINK_ROWS: 2
            ETL_INFO: 
           TASK_INFO: resource:N/A; timeout(s):300; max_filter_ratio:0.0
         CREATE_TIME: 2023-08-09 10:42:23
      ETL_START_TIME: 2023-08-09 10:42:23
     ETL_FINISH_TIME: 2023-08-09 10:42:23
     LOAD_START_TIME: 2023-08-09 10:42:23
    LOAD_FINISH_TIME: 2023-08-09 10:42:24
         JOB_DETAILS: {"All backends":{"5ebf11b5-365e-11ee-9e4a-7a563fb695da":[10006]},"FileNumber":0,"FileSize":0,"InternalTableLoadBytes":175,"InternalTableLoadRows":2,"ScanBytes":0,"ScanRows":0,"TaskNumber":1,"Unfinished backends":{"5ebf11b5-365e-11ee-9e4a-7a563fb695da":[]}}
           ERROR_MSG: NULL
        TRACKING_URL: NULL
        TRACKING_SQL: NULL
REJECTED_RECORD_PATH: NULL
1 row in set (0.01 sec)
```

有关返回结果中字段的详细信息，请参阅[信息模式 \"> loads\"](../reference/information_schema/loads.md)。

### 通过 curl 命令检查

您可以使用 curl 命令检查 INSERT 事务的状态。

打开终端，执行以下命令：

```Bash
curl --location-trusted -u <username>:<password> \
  http://<fe_address>:<fe_http_port>/api/<db_name>/_load_info?label=<label_name>
```

以下示例检查带有标签 insert_load_wikipedia 的事务的状态。

```Bash
curl --location-trusted -u <username>:<password> \
  http://x.x.x.x:8030/api/load_test/_load_info?label=insert_load_wikipedia
```

> **注意**
> 如果您使用的账户没有设置密码，您只需输入 `
<用户名>`:

返回结果如下：

```Plain
{
   "jobInfo":{
      "dbName":"load_test",
      "tblNames":[
         "source_wiki_edit"
      ],
      "label":"insert_load_wikipedia",
      "state":"FINISHED",
      "failMsg":"",
      "trackingUrl":""
   },
   "status":"OK",
   "msg":"Success"
}
```

## 配置

您可以为 INSERT 事务设置以下配置项：

- **FE 配置**

|FE配置|说明|
|---|---|
|insert_load_default_timeout_second|INSERT 事务的默认超时。单位：第二。如果当前INSERT事务没有在该参数设置的时间内完成，则会被系统取消，状态为CANCELLED。对于当前版本的StarRocks，您只能使用该参数为所有INSERT事务指定统一的超时时间，并且不能为特定的INSERT事务设置不同的超时时间。默认值为 3600 秒（1 小时）。如果INSERT事务无法在指定时间内完成，可以通过调整该参数来延长超时时间。|

- **会话变量**

|会话变量|描述|
|---|---|
|enable_insert_strict|开关值来控制 INSERT 事务是否容忍无效数据行。当它设置为 true 时，如果任何数据行无效，则事务将失败。当设置为 false 时，至少一行数据正确加载后事务成功，并返回标签。默认为 true。您可以使用 SET enable_insert_strict = {true 或 false}; 设置此变量命令。|
|query_timeout|SQL 命令超时。单位：第二。 INSERT 作为 SQL 命令，也受到此会话变量的限制。您可以使用 SET query_timeout = xxx; 设置此变量。命令。|
