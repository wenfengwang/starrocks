---
displayed_sidebar: English
---

# 使用 INSERT 加载数据

从 '../assets/commonMarkdown/insertPrivNote.md' 导入 InsertPrivNote。

本主题描述了如何使用 SQL 语句 - INSERT 将数据加载到 StarRocks 中。

与 MySQL 和许多其他数据库管理系统类似，StarRocks 支持使用 INSERT 将数据加载到内部表中。您可以使用 VALUES 子句直接插入一行或多行，以测试函数或 DEMO。您还可以将查询结果定义的数据插入到内部表中，从[外部表](../data_source/External_table.md)。从 StarRocks v3.1 开始，您可以使用 INSERT 命令和表函数 FILES() 直接从云存储中的文件加载数据[](../sql-reference/sql-functions/table-functions/files.md)。

StarRocks v2.4 进一步支持使用 INSERT OVERWRITE 将数据覆盖到表中。INSERT OVERWRITE 语句集成了以下操作来实现覆盖功能：

1. 根据存储原始数据的分区创建临时分区。
2. 将数据插入到临时分区中。
3. 将原始分区与临时分区交换。

> **注意**
>
> 如果需要在覆盖数据之前验证数据，可以按照上述过程覆盖数据并在交换分区之前对其进行验证，而不是使用 INSERT OVERWRITE。

## 注意事项

- 您只能通过在 MySQL 客户端按下 **Ctrl** 和 **C** 键来取消同步 INSERT 事务。
- 您可以使用 [SUBMIT TASK](../sql-reference/sql-statements/data-manipulation/SUBMIT_TASK.md) 提交异步 INSERT 任务。
- 对于 StarRocks 的当前版本，默认情况下，如果任何行的数据不符合表的 schema，则 INSERT 事务将失败。例如，如果任何行中字段的长度超过表中映射字段的长度限制，则 INSERT 事务将失败。您可以将会话变量 `enable_insert_strict` 设置为 `false`，以允许事务继续，通过过滤掉与表不匹配的行。
- 如果您频繁执行 INSERT 语句，将小批量数据加载到 StarRocks 中，将会生成过多的数据版本。这严重影响查询性能。我们建议在生产环境中，不要过于频繁地使用 INSERT 命令加载数据，也不要将其用作每天加载数据的例程。如果您的应用程序或分析场景需要单独加载流数据或小数据批次的解决方案，我们建议您使用 Apache Kafka® 作为数据源，并通过[例程加载](../loading/RoutineLoad.md)加载数据。
- 如果执行 INSERT OVERWRITE 语句，StarRocks 会为存放原始数据的分区创建临时分区，在临时分区中插入新数据，并将原始分区[与临时分区进行交换](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md#use-a-temporary-partition-to-replace-current-partition)。所有这些操作都在 FE Leader 节点中执行。因此，如果 FE Leader 节点在执行 INSERT OVERWRITE 命令时崩溃，则整个加载事务将失败，临时分区将被截断。

## 准备工作

### 检查权限

<InsertPrivNote />

### 创建对象

创建名为 `load_test` 的数据库，并创建一个名为 `insert_wiki_edit` 的表作为目标表，以及一个名为 `source_wiki_edit` 的表作为源表。

> **注意**
>
> 本主题中演示的示例基于表 `insert_wiki_edit` 和表 `source_wiki_edit`。如果您更喜欢使用自己的表和数据，可以跳过准备工作并继续下一步。

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

> **注意**
>
> 从 v2.5.7 开始，StarRocks 可以在您创建表或添加分区时自动设置 BUCKET 数量。您不再需要手动设置存储桶数量。有关详细信息，请参阅 [确定存储桶数量](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

## 通过 INSERT INTO VALUES 插入数据

您可以使用 INSERT INTO VALUES 命令将一行或多行追加到特定表中。多行之间用逗号（,）分隔。有关详细说明和参数参考，请参阅 [SQL 参考 - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)。

> **注意**
>
> 通过 INSERT INTO VALUES 插入数据仅适用于需要使用小型数据集验证 DEMO 的情况。不建议将其用于大规模测试或生产环境。如需将海量数据加载到 StarRocks 中，请参阅[Ingestion Overview](../loading/Loading_intro.md)，了解适合您场景的其他选项。

下面的示例使用标签 `insert_load_wikipedia`，将两行数据插入到数据源表 `source_wiki_edit` 中。

```SQL
INSERT INTO source_wiki_edit
WITH LABEL insert_load_wikipedia
VALUES
    ("2015-09-12 00:00:00","#en.wikipedia","AustinFF",0,0,0,0,0,21,5,0),
    ("2015-09-12 00:00:00","#ca.wikipedia","helloSR",0,1,0,1,0,3,23,0);
```

## 通过 INSERT INTO SELECT 插入数据

您可以使用 INSERT INTO SELECT 命令将数据源表的查询结果加载到目标表中。INSERT INTO SELECT 命令对数据源表中的数据进行 ETL 操作，并将数据加载到 StarRocks 的内部表中。数据源可以是一个或多个内部或外部表，甚至可以是云存储上的数据文件。目标表必须是 StarRocks 中的内部表。有关详细说明和参数参考，请参阅 [SQL 参考 - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)。

### 将数据从内部表或外部表插入到内部表中

> **注意**
>
> 从外部表插入数据与从内部表插入数据相同。为简单起见，我们仅在以下示例中演示如何从内部表插入数据。

- 以下示例将源表中的数据插入到目标表 `insert_wiki_edit` 中。

```SQL
INSERT INTO insert_wiki_edit
WITH LABEL insert_load_wikipedia_1
SELECT * FROM source_wiki_edit;
```

- 以下示例将源表中的数据插入到目标表 `insert_wiki_edit` 的 `p06` 和 `p12` 分区中。如果未指定分区，则数据将插入到所有分区中。否则，数据将仅插入到指定的分区中。

```SQL
INSERT INTO insert_wiki_edit PARTITION(p06, p12)
WITH LABEL insert_load_wikipedia_2
SELECT * FROM source_wiki_edit;
```

查询目标表，确保其中有数据。

```Plain text
MySQL > select * from insert_wiki_edit;
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| event_time          | channel       | user     | is_anonymous | is_minor | is_new | is_robot | is_unpatrolled | delta | added | deleted |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+

| 2015-09-12 00:00:00 | #en.wikipedia | AustinFF |            0 |        0 |      0 |        0 |              0 |    21 |     5 |       0 |
| 2015-09-12 00:00:00 | #ca.wikipedia | helloSR  |            0 |        1 |      0 |        1 |              0 |     3 |    23 |       0 |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 行 (0.00 秒)
```

如果截断 `p06` 和 `p12` 分区，则查询中将不会返回数据。

```Plain
MySQL > TRUNCATE TABLE insert_wiki_edit PARTITION(p06, p12);
查询 OK，0 行受影响 (0.01 秒)

MySQL > select * from insert_wiki_edit;
空集 (0.00 秒)
```

- 以下示例将源表中的 `event_time` 和 `channel` 列插入目标表 `insert_wiki_edit`。未在此处指定的列将使用默认值。

```SQL
INSERT INTO insert_wiki_edit
WITH LABEL insert_load_wikipedia_3 
(
    event_time, 
    channel
)
SELECT event_time, channel FROM source_wiki_edit;
```

### 使用 FILES() 从外部源中直接插入数据

从 v3.1 开始，StarRocks 支持使用 INSERT 命令和 [FILES()](../sql-reference/sql-functions/table-functions/files.md) 函数直接从云存储中的文件加载数据，无需首先创建外部目录或外部文件表。此外，FILES() 可以自动推断文件的表模式，大大简化了数据加载过程。

以下示例将 AWS S3 存储桶中的 Parquet 文件 **parquet/insert_wiki_edit_append.parquet** 中的数据行插入到表 `insert_wiki_edit` 中：

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

您可以使用 INSERT OVERWRITE VALUES 命令用一行或多行覆盖特定表。多行之间用逗号（,）分隔。有关详细说明和参数，请参见[SQL 参考 - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)。

> **注意**
>
> 通过 INSERT OVERWRITE VALUES 仅适用于需要使用小型数据集验证 DEMO 的情况。不建议在大规模测试或生产环境中使用。要将大量数据加载到 StarRocks 中，请参见[数据加载概述](../loading/Loading_intro.md)，了解适合您场景的其他选项。

查询源表和目标表，确保其中有数据。

```Plain
MySQL > SELECT * FROM source_wiki_edit;
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| event_time          | channel       | user     | is_anonymous | is_minor | is_new | is_robot | is_unpatrolled | delta | added | deleted |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #ca.wikipedia | helloSR  |            0 |        1 |      0 |        1 |              0 |     3 |    23 |       0 |
| 2015-09-12 00:00:00 | #en.wikipedia | AustinFF |            0 |        0 |      0 |        0 |              0 |    21 |     5 |       0 |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 行 (0.02 秒)
 
MySQL > SELECT * FROM insert_wiki_edit;
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| event_time          | channel       | user     | is_anonymous | is_minor | is_new | is_robot | is_unpatrolled | delta | added | deleted |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #ca.wikipedia | helloSR  |            0 |        1 |      0 |        1 |              0 |     3 |    23 |       0 |
| 2015-09-12 00:00:00 | #en.wikipedia | AustinFF |            0 |        0 |      0 |        0 |              0 |    21 |     5 |       0 |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 行 (0.01 秒)
```

以下示例用两个新行覆盖源表 `source_wiki_edit`。

```SQL
INSERT OVERWRITE source_wiki_edit
WITH LABEL insert_load_wikipedia_ow
VALUES
    ("2015-09-12 00:00:00","#cn.wikipedia","GELongstreet",0,0,0,0,0,36,36,0),
    ("2015-09-12 00:00:00","#fr.wikipedia","PereBot",0,1,0,1,0,17,17,0);
```

## 通过 INSERT OVERWRITE SELECT 覆盖数据

您可以通过 INSERT OVERWRITE SELECT 命令使用对数据源表的查询结果覆盖表。INSERT OVERWRITE SELECT 语句对一个或多个内部或外部表中的数据执行 ETL 操作，并用数据覆盖内部表。有关详细说明和参数，请参见[SQL 参考 - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)。

> **注意**
>
> 从外部表加载数据与从内部表加载数据相同。为简单起见，在以下示例中，我们仅演示如何使用内部表中的数据覆盖目标表。

查询源表和目标表，确保它们包含不同的数据行。

```Plain
MySQL > SELECT * FROM source_wiki_edit;
+---------------------+---------------+--------------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| event_time          | channel       | user         | is_anonymous | is_minor | is_new | is_robot | is_unpatrolled | delta | added | deleted |
+---------------------+---------------+--------------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #cn.wikipedia | GELongstreet |            0 |        0 |      0 |        0 |              0 |    36 |    36 |       0 |
| 2015-09-12 00:00:00 | #fr.wikipedia | PereBot      |            0 |        1 |      0 |        1 |              0 |    17 |    17 |       0 |
+---------------------+---------------+--------------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 行 (0.02 秒)
 
MySQL > SELECT * FROM insert_wiki_edit;
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| event_time          | channel       | user     | is_anonymous | is_minor | is_new | is_robot | is_unpatrolled | delta | added | deleted |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #en.wikipedia | AustinFF |            0 |        0 |      0 |        0 |              0 |    21 |     5 |       0 |
| 2015-09-12 00:00:00 | #ca.wikipedia | helloSR  |            0 |        1 |      0 |        1 |              0 |     3 |    23 |       0 |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 行 (0.01 秒)
```

以下示例使用源表中的数据覆盖表 `insert_wiki_edit`。

```SQL
INSERT OVERWRITE insert_wiki_edit
WITH LABEL insert_load_wikipedia_ow_1
SELECT * FROM source_wiki_edit;
```

以下示例使用源表中的数据覆盖表 `insert_wiki_edit` 的 `p06` 和 `p12` 分区。

```SQL
INSERT OVERWRITE insert_wiki_edit PARTITION(p06, p12)
WITH LABEL insert_load_wikipedia_ow_2
SELECT * FROM source_wiki_edit;
```

查询目标表以确保其中有数据。

```plain text
MySQL > select * from insert_wiki_edit;
+---------------------+---------------+--------------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| event_time          | channel       | user         | is_anonymous | is_minor | is_new | is_robot | is_unpatrolled | delta | added | deleted |
+---------------------+---------------+--------------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #fr.wikipedia | PereBot      |            0 |        1 |      0 |        1 |              0 |    17 |    17 |       0 |
| 2015-09-12 00:00:00 | #cn.wikipedia | GELongstreet |            0 |        0 |      0 |        0 |              0 |    36 |    36 |       0 |
+---------------------+---------------+--------------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 行 (0.01 秒)
如果截断 `p06` 和 `p12` 分区，则查询中将不会返回数据。

```Plain
MySQL > TRUNCATE TABLE insert_wiki_edit PARTITION(p06, p12);
Query OK, 0 rows affected (0.01 sec)

MySQL > select * from insert_wiki_edit;
Empty set (0.00 sec)
```

- 以下示例将使用源表的 `event_time` 和 `channel` 列覆盖目标表 `insert_wiki_edit`。对于未覆盖任何数据的列，将分配默认值。

```SQL
INSERT OVERWRITE insert_wiki_edit
WITH LABEL insert_load_wikipedia_ow_3 
(
    event_time, 
    channel
)
SELECT event_time, channel FROM source_wiki_edit;
```

## 将数据插入到具有生成列的表中

生成列是一种特殊列，其值是根据预定义表达式或基于其他列的计算得出的。当您的查询请求涉及对昂贵表达式的计算时（例如，从 JSON 值查询特定字段或计算 ARRAY 数据），生成列特别有用。StarRocks 在加载数据到表中时会评估表达式并将结果存储在生成的列中，从而避免在查询期间进行表达式评估，并提高查询性能。

您可以使用 INSERT 将数据加载到具有生成列的表中。

以下示例创建了一个表 `insert_generated_columns` 并向其中插入了一行数据。该表包含两个生成列：`avg_array` 和 `get_string`。`avg_array` 计算了 `data_array` 中 ARRAY 数据的平均值，并且 `get_string` 从 `data_json` 的 `a` JSON 路径中提取了字符串。

```SQL
CREATE TABLE insert_generated_columns (
  id           INT(11)           NOT NULL    COMMENT "ID",
  data_array   ARRAY<INT(11)>    NOT NULL    COMMENT "ARRAY",
  data_json    JSON              NOT NULL    COMMENT "JSON",
  avg_array    DOUBLE            NULL 
      AS array_avg(data_array)               COMMENT "获取 ARRAY 的平均值",
  get_string   VARCHAR(65533)    NULL 
      AS get_json_string(json_string(data_json), '$.a') COMMENT "提取 JSON 字符串"
) ENGINE=OLAP 
PRIMARY KEY(id)
DISTRIBUTED BY HASH(id);

INSERT INTO insert_generated_columns 
VALUES (1, [1,2], parse_json('{"a" : 1, "b" : 2}'));
```

> **注意**
>
> 不支持直接将数据加载到生成列中。

您可以查询表以检查其中的数据。

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

使用 INSERT 加载数据会提交同步事务，该事务可能会因会话中断或超时而失败。您可以使用 [SUBMIT TASK](../sql-reference/sql-statements/data-manipulation/SUBMIT_TASK.md) 提交异步 INSERT 事务。此功能从 StarRocks v2.5 开始支持。

- 以下示例异步将数据从源表插入到目标表 `insert_wiki_edit`。

```SQL
SUBMIT TASK AS INSERT INTO insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

- 以下示例异步使用源表的数据覆盖表 `insert_wiki_edit`。

```SQL
SUBMIT TASK AS INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

- 以下示例异步使用源表的数据覆盖表 `insert_wiki_edit`，并使用提示将查询超时延长到 `100000` 秒。

```SQL
SUBMIT /*+set_var(query_timeout=100000)*/ TASK AS
INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

- 以下示例异步使用源表的数据覆盖表 `insert_wiki_edit`，并将任务名称指定为 `async`。

```SQL
SUBMIT TASK async
AS INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

您可以通过查询信息架构中的 `task_runs` 元数据视图来检查异步 INSERT 任务的状态。

以下示例检查异步 INSERT 任务 `async` 的状态。

```SQL
SELECT * FROM information_schema.task_runs WHERE task_name = 'async';
```

## 检查 INSERT 作业状态

### 通过结果检查

同步 INSERT 事务根据事务的结果返回不同的状态。

- **事务成功**

如果事务成功，StarRocks 将返回以下结果：

```Plain
Query OK, 2 rows affected (0.05 sec)
{'label':'insert_load_wikipedia', 'status':'VISIBLE', 'txnId':'1006'}
```

- **事务失败**

如果所有数据行都无法加载到目标表中，则 INSERT 事务将失败。如果事务失败，StarRocks 将返回以下结果：

```Plain
ERROR 1064 (HY000): Insert has filtered data in strict mode, tracking_url=http://x.x.x.x:yyyy/api/_load_error_log?file=error_log_9f0a4fd0b64e11ec_906bbede076e9d08
```

您可以通过检查带有 `tracking_url` 的日志来定位问题。

### 通过信息架构进行检查

可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 语句从 `information_schema` 数据库中的 `loads` 表中查询一个或多个加载作业的结果。此功能从 v3.1 开始支持。

示例 1：查询在 `load_test` 数据库上执行的加载作业的结果，按创建时间（`CREATE_TIME`）降序排序，并仅返回顶部结果。

```SQL
SELECT * FROM information_schema.loads
WHERE database_name = 'load_test'
ORDER BY create_time DESC
LIMIT 1\G
```

示例 2：查询在 `load_test` 数据库上执行的加载作业（其标签为 `insert_load_wikipedia`）的结果。

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

有关返回结果中的字段的信息，请参阅 [Information Schema > loads](../reference/information_schema/loads.md)。

### 通过 curl 命令进行检查

您可以使用 curl 命令检查 INSERT 事务的状态。

启动终端，并执行以下命令：

```Bash
curl --location-trusted -u <username>:<password> \
  http://<fe_address>:<fe_http_port>/api/<db_name>/_load_info?label=<label_name>
```

以下示例使用标签检查事务的状态 `insert_load_wikipedia`。

```Bash
curl --location-trusted -u <username>:<password> \
  http://x.x.x.x:8030/api/load_test/_load_info?label=insert_load_wikipedia
```

> **注意**
>
> 如果您使用未设置密码的帐户，则只需输入 `<username>:`。

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

| FE 配置                   | 描述                                                  |
| ---------------------------------- | ------------------------------------------------------------ |
| insert_load_default_timeout_second | INSERT 事务的默认超时。单位：秒。如果当前 INSERT 事务未在该参数设置的时间内完成，则系统将取消该事务，状态为 CANCELLED。对于当前版本的 StarRocks，您只能使用该参数为所有 INSERT 事务指定统一的超时时间，不能为特定的 INSERT 事务设置不同的超时时间。默认值为 3600 秒（1 小时）。如果在指定时间内无法完成 INSERT 事务，可以通过调整该参数来延长超时时间。 |

- **会话变量**

| 会话变量     | 描述                                                  |
| -------------------- | ------------------------------------------------------------ |

| enable_insert_strict | 切换值，控制是否容忍无效数据行的 INSERT 事务。当设置为 `true` 时，如果任何数据行无效，事务将失败。当设置为 `false` 时，只要至少一行数据已正确加载，事务即成功，并返回标签。默认值为 `true`。您可以使用命令 `SET enable_insert_strict = {true or false};` 来设置此变量。 |
| query_timeout        | SQL 命令的超时时间。单位：秒。INSERT 作为 SQL 命令，也受此会话变量的限制。您可以使用命令 `SET query_timeout = xxx;` 来设置此变量。 |