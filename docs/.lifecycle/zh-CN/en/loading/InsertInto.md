---
displayed_sidebar: "English"
---

# 使用INSERT加载数据

从../assets/commonMarkdown/insertPrivNote.md导入InsertPrivNote

本主题介绍了如何使用SQL语句- INSERT将数据加载到StarRocks中。

与MySQL和许多其他数据库管理系统类似，StarRocks支持使用INSERT将数据加载到内部表中。您可以使用VALUES子句直接插入一行或多行数据以测试功能或演示。您还可以通过查询结果插入一个内部表中的数据。从StarRocks v3.1开始，您可以使用INSERT命令和表函数FILES()直接从云存储中加载数据。

StarRocks v2.4进一步支持使用INSERT OVERWRITE将数据覆盖到表中。INSERT OVERWRITE语句集成了以下操作以实现覆盖功能：

1. 根据存储原始数据的分区创建临时分区。
2. 将数据插入临时分区。
3. 交换原始分区与临时分区。

> **注意**
>
> 如果需要在覆盖数据之前验证数据，而不是使用INSERT OVERWRITE，您可以在交换分区之前按照上述步骤覆盖数据并验证数据。

## 注意事项

- 您只能通过按下MySQL客户端中的**Ctrl**和**C**键来取消同步的INSERT事务。
- 您可以使用[SUBMIT TASK](../sql-reference/sql-statements/data-manipulation/SUBMIT_TASK.md)提交异步的INSERT任务。
- 对于当前版本的StarRocks，如果任何行的数据不符合表的架构，则INSERT事务默认失败。例如，如果任何行中某字段的长度超过表中映射字段的长度限制，则INSERT事务将失败。您可以将会话变量`enable_insert_strict`设置为`false`，以允许事务继续进行，通过过滤掉不匹配表的行。
- 如果您经常执行INSERT语句将小批量数据加载到StarRocks中，则会产生过多的数据版本。这严重影响查询性能。我们建议在生产中，您不要经常使用INSERT命令加载数据，也不要将其用作每日数据加载的例行程序。如果您的应用程序或分析场景需要单独加载流数据或小数据批次的解决方案，我们建议您使用Apache Kafka®作为您的数据源，并通过[例行加载](../loading/RoutineLoad.md)加载数据。
- 如果执行INSERT OVERWRITE语句，StarRocks将为存储原始数据的分区创建临时分区，将新数据插入临时分区，并[交换原始分区与临时分区](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md#use-a-temporary-partition-to-replace-current-partition)。所有这些操作在FE Leader节点上执行。因此，如果FE Leader节点在执行INSERT OVERWRITE命令时崩溃，整个加载事务将失败，临时分区将被截断。

## 准备工作

### 检查权限

<InsertPrivNote />

### 创建对象

创建名为`load_test`的数据库，并创建目标表`insert_wiki_edit`和源表`source_wiki_edit`。

> **注意**
>
> 本主题演示的示例基于表`insert_wiki_edit`和表`source_wiki_edit`。如果您希望使用自己的表和数据，可以跳过准备工作，直接转到下一步。

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
>
> 自v2.5.7以来，当您创建表或添加分区时，StarRocks可以自动设置桶的数量(BUCKETS)。您无需手动设置桶的数量。有关详细信息，请参见[determine the number of buckets](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

## 通过INSERT INTO VALUES插入数据

您可以使用INSERT INTO VALUES命令将一行或多行追加到特定表中。多行之间用逗号(,)分隔。有关详细说明和参数引用，请参见[SQL参考- INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)。

> **注意**
>
> 通过INSERT INTO VALUES插入数据仅适用于需要使用小数据集验证DEMO的情况。不建议用于大规模测试或生产环境。要加载大量数据到StarRocks，请参阅[Ingestion Overview](../loading/Loading_intro.md)以获得适合您场景的其他选项。

以下示例将两行插入数据源表`source_wiki_edit`，标签为`insert_load_wikipedia`。标签是数据库内每个数据加载事务的唯一标识标签。

```SQL
INSERT INTO source_wiki_edit
WITH LABEL insert_load_wikipedia
VALUES
    ("2015-09-12 00:00:00","#en.wikipedia","AustinFF",0,0,0,0,0,21,5,0),
    ("2015-09-12 00:00:00","#ca.wikipedia","helloSR",0,1,0,1,0,3,23,0);
```

## 通过INSERT INTO SELECT插入数据

您可以使用INSERT INTO SELECT命令将数据源表上的查询结果加载到目标表中。INSERT INTO SELECT命令在StarRocks中执行数据源表的ETL操作，并将数据加载到内部表中。数据源可以是一个或多个内部表或外部表，甚至云存储中的数据文件。目标表必须是StarRocks中的内部表。有关详细说明和参数引用，请参见[SQL参考- INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)。

### 从内部或外部表插入数据到内部表

> **注意**
>
> 从外部表插入数据与从内部表插入数据相同。为简单起见，以下示例仅演示如何从内部表插入数据。

- 以下示例将来自源表的数据插入目标表`insert_wiki_edit`。

```SQL
INSERT INTO insert_wiki_edit
WITH LABEL insert_load_wikipedia_1
SELECT * FROM source_wiki_edit;
```

- 以下示例将来自源表的数据插入目标表`insert_wiki_edit`的`p06`和`p12`分区。如果未指定分区，则数据将插入所有分区。否则，数据将仅插入指定的分区。

```SQL
INSERT INTO insert_wiki_edit PARTITION(p06, p12)
WITH LABEL insert_load_wikipedia_2
SELECT * FROM source_wiki_edit;
```

查询目标表以确保其中有数据。

```Plain text```
```SQL
TRUNCATE TABLE insert_wiki_edit PARTITION(p06, p12);
```
```
使用TRUNCATE TABLE insert_wiki_edit PARTITION(p06, p12)命令将`p06`和`p12`分区截断可能导致查询不返回数据。
```

```SQL
INSERT INTO insert_wiki_edit WITH LABEL insert_load_wikipedia_3 (event_time, channel) SELECT event_time, channel FROM source_wiki_edit;
```
```
以下示例将源表中的`event_time`和`channel`列插入到目标表`insert_wiki_edit`中。在这里没有指定的列中使用默认值。
```

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
```
从v3.1开始，StarRocks支持使用INSERT命令和[FILES()](../sql-reference/sql-functions/table-functions/files.md)函数直接从云存储中的文件加载数据，因此您无需首先创建外部目录或外部文件表。此外，FILES()可以自动推断文件的表模式，大大简化了数据加载过程。

以下示例将AWS S3存储桶`inserttest`中**parquet/insert_wiki_edit_append.parquet**中的Parquet文件中的数据行插入到表`insert_wiki_edit`中：
```

```SQL
INSERT OVERWRITE source_wiki_edit WITH LABEL insert_load_wikipedia_ow VALUES ("2015-09-12 00:00:00","#cn.wikipedia","GELongstreet",0,0,0,0,0,36,36,0), ("2015-09-12 00:00:00","#fr.wikipedia","PereBot",0,1,0,1,0,17,17,0);
```
```
通过使用INSERT OVERWRITE VALUES命令，您可以使用一个或多个行覆盖特定表。多行以逗号(,)分隔。有关详细的说明和参数参考，请参阅[SQL参考 - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)。

在执行INSERT OVERWRITE VALUES命令之前，通过查询源表和目标表来确保它们中有数据。
```

```Plain
MySQL > SELECT * FROM source_wiki_edit;
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| event_time          | channel       | user     | is_anonymous | is_minor | is_new | is_robot | is_unpatrolled | delta | added | deleted |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #cn.wikipedia | GELongstreet |            0 |        0 |      0 |        0 |              0 |    36 |    36 |       0 |
| 2015-09-12 00:00:00 | #fr.wikipedia | PereBot |            0 |        1 |      0 |        1 |              0 |    17 |    17 |       0 |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
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
```
以下示例通过两行数据覆盖源表`source_wiki_edit`。
```

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
```
通过使用INSERT OVERWRITE SELECT命令，您可以通过查询数据源表覆盖表的结果。INSERT OVERWRITE SELECT语句对内部或外部表的数据执行ETL操作，并使用内部表的数据覆盖内部表。有关详细的说明和参数参考，请参阅[SQL参考 - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)。

查询源表和目标表以确保它们持有不同的数据行。
```
- 以下示例会使用源表的数据覆盖表`insert_wiki_edit`。

```SQL
INSERT OVERWRITE insert_wiki_edit
WITH LABEL insert_load_wikipedia_ow_1
SELECT * FROM source_wiki_edit;
```

- 以下示例将覆盖表`insert_wiki_edit`的`p06`和`p12`分区，并使用源表的数据。

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
2 rows in set (0.01 sec)
```

如果你截断`p06`和`p12`分区，数据将不会在查询中返回。

```Plain
MySQL > TRUNCATE TABLE insert_wiki_edit PARTITION(p06, p12);
Query OK, 0 rows affected (0.01 sec)

MySQL > select * from insert_wiki_edit;
Empty set (0.00 sec)
```

- 以下示例会使用源表的`event_time`和`channel`列覆盖目标表`insert_wiki_edit`。默认值会被分配到没有被覆盖的列中。

```SQL
INSERT OVERWRITE insert_wiki_edit
WITH LABEL insert_load_wikipedia_ow_3 
(
    event_time, 
    channel
)
SELECT event_time, channel FROM source_wiki_edit;
```

## 将数据插入具有生成列的表中

生成列是特殊列，其值派生自预定义表达式或基于其他列的评估。 当您的查询请求涉及昂贵表达式的评估时，例如从JSON值查询某个字段，或计算ARRAY数据时，生成列尤其有用。StarRocks在表中加载数据时评估表达式并将结果存储在生成列中，从而避免在查询期间进行表达式评估，并提高查询性能。

您可以使用INSERT将数据加载到具有生成列的表中。

以下示例创建了一个名为`insert_generated_columns`的表，并向其中插入一行数据。该表包含两个生成列：`avg_array`和`get_string`。`avg_array`计算`data_array`中ARRAY数据的平均值，而`get_string`从`data_json`中的JSON路径`a`中提取字符串。

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

## 使用INSERT异步加载数据

使用INSERT加载数据会提交同步事务，可能会因为会话中断或超时而失败。 您可以使用[SUBMIT TASK](../sql-reference/sql-statements/data-manipulation/SUBMIT_TASK.md)异步提交INSERT事务。 该功能从StarRocks v2.5开始支持。

- 以下示例会异步将源表中的数据插入目标表`insert_wiki_edit`。

```SQL
SUBMIT TASK AS INSERT INTO insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

- 以下示例会异步将源表的数据覆盖表`insert_wiki_edit`。

```SQL
SUBMIT TASK AS INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

- 以下示例会异步将源表的数据覆盖表`insert_wiki_edit`，并使用提示将查询超时时间延长为`100000`秒。

```SQL
SUBMIT /*+set_var(query_timeout=100000)*/ TASK AS
INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

- 以下示例会异步将源表的数据覆盖表`insert_wiki_edit`，并将任务名称指定为`async`。

```SQL
SUBMIT TASK async
AS INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

您可以通过查询信息模式中的元数据视图`task_runs`来检查异步INSERT任务的状态。

以下示例检查名为`async`的INSERT任务的状态。

```SQL
SELECT * FROM information_schema.task_runs WHERE task_name = 'async';
```

## 检查INSERT作业状态

### 通过结果检查

同步INSERT事务根据事务的结果返回不同的状态。

- **事务成功**

如果事务成功，StarRocks返回以下内容：

```Plain
Query OK, 2 rows affected (0.05 sec)
{'label':'insert_load_wikipedia', 'status':'VISIBLE', 'txnId':'1006'}
```

- **事务失败**

如果所有数据行均未成功加载到目标表，则INSERT事务失败。 如果事务失败，StarRocks返回以下内容：

```Plain
ERROR 1064 (HY000): Insert has filtered data in strict mode, tracking_url=http://x.x.x.x:yyyy/api/_load_error_log?file=error_log_9f0a4fd0b64e11ec_906bbede076e9d08
```

您可以通过检查带有`tracking_url`标识的日志来定位问题。

### 通过信息模式检查

您可以使用[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)语句从`information_schema`数据库中的`loads`表中查询一个或多个加载作业的结果。 该功能从v3.1开始支持。

示例1：查询在`load_test`数据库上执行的加载作业的结果，以创建时间（`CREATE_TIME`）降序排序结果，并仅返回顶部结果。

```SQL
SELECT * FROM information_schema.loads
WHERE database_name = 'load_test'
ORDER BY create_time DESC
LIMIT 1\G
```

示例2：查询在`load_test`数据库上执行的加载作业（其标签为`insert_load_wikipedia`）的结果：

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


有关返回结果中字段的信息，请参阅[Information Schema > loads](../reference/information_schema/loads.md)。

### 通过 curl 命令检查

您可以使用 curl 命令来检查 INSERT 事务的状态。

打开终端，并执行以下命令:

```Bash
curl --location-trusted -u <username>:<password> \
  http://<fe_address>:<fe_http_port>/api/<db_name>/_load_info?label=<label_name>
```

以下示例检查带有标签 `insert_load_wikipedia` 的事务的状态。

```Bash
curl --location-trusted -u <username>:<password> \
  http://x.x.x.x:8030/api/load_test/_load_info?label=insert_load_wikipedia
```

> **注意**
>
> 如果您使用的帐户未设置密码，您只需输入 `<username>:`。

返回结果如下:

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

您可以为 INSERT 事务设置以下配置项:

- **前端配置**

| 前端配置                           | 描述                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| insert_load_default_timeout_second | INSERT 事务的默认超时时间。单位：秒。如果当前 INSERT 事务未在此参数设置的时间内完成，系统将取消该事务并且状态将变为 CANCELLED。就当前版本的 StarRocks 而言，您只能使用此参数为所有 INSERT 事务指定统一的超时时间，无法为特定 INSERT 事务设置不同的超时时间。默认值为 3600 秒 (1 小时)。如果指定时间内未完成 INSERT 事务，您可以通过调整此参数来延长超时时间。 |

- **会话变量**

| 会话变量              | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| enable_insert_strict | 开关值，用于控制 INSERT 事务是否容忍无效的数据行。当设置为 `true` 时，如果任何数据行无效，则事务将失败。当设置为 `false` 时，只要至少一行数据已正确加载，事务即成功，会返回标签。默认为 `true`。您可以使用 `SET enable_insert_strict = {true or false};` 命令设置此变量。 |
| query_timeout       | SQL 命令的超时时间。单位：秒。INSERT 作为 SQL 命令，也受此会话变量限制。您可以使用 `SET query_timeout = xxx;` 命令设置此变量。 |