---
displayed_sidebar: "中文"
---

# 显示LOAD

## 描述

显示数据库中所有加载作业或给定加载作业的信息。此语句只能显示通过[Broker Load](../data-manipulation/BROKER_LOAD.md)、[INSERT](./INSERT.md)或[Spark Load](../data-manipulation/SPARK_LOAD.md)创建的加载作业。您还可以通过`curl`命令查看加载作业的信息。从v3.1开始，我们建议您使用[SELECT](../data-manipulation/SELECT.md)语句从`information_schema`数据库中的[`loads`](../../../reference/information_schema/loads.md)表查询Broker Load或Insert作业的结果。有关更多信息，请参见[从HDFS加载数据](../../../loading/hdfs_load.md)、[从云存储加载数据](../../../loading/cloud_storage_load.md)、[使用INSERT加载数据](../../../loading/InsertInto.md)和[使用Apache Spark™进行批量加载](../../../loading/SparkLoad.md)。

除了上述加载方法之外，StarRocks还支持使用Stream Load和Routine Load加载数据。Stream Load是同步操作，将直接返回Stream Load作业的信息。Routine Load是一种异步操作，您可以使用[SHOW ROUTINE LOAD](../data-manipulation/SHOW_ROUTINE_LOAD.md)语句显示Routine Load作业的信息。

## 语法

```SQL
SHOW LOAD [ FROM db_name ]
[
   WHERE [ LABEL { = "label_name" | LIKE "label_matcher" } ]
         [ [AND] STATE = { "PENDING" | "ETL" | "LOADING" | "FINISHED" | "CANCELLED" } ]
]
[ ORDER BY field_name [ ASC | DESC ] ]
[ LIMIT { [offset, ] limit | limit OFFSET offset } ]
```

> **说明**
>
> 您可以在语句中添加`\G`选项（例如`SHOW LOAD WHERE LABEL = "label1"\G;`）以垂直显示输出，而不是通常的水平表格格式。有关更多信息，请参见[示例1](#示例)。

## 参数

| **参数**                          | **必需** | **描述**                                                     |
| --------------------------------- | -------- | ------------------------------------------------------------ |
| db_name                           | 否       | 数据库名称。如果未指定此参数，则默认使用当前数据库。            |
| LABEL = "label_name"              | 否       | 加载作业的标签。                                              |
| LABEL LIKE "label_matcher"        | 否       | 如果指定了此参数，则返回包含`label_matcher`标签的加载作业的信息。 |
| AND                               | 否       | <ul><li>如果在WHERE子句中只指定了一个过滤条件，请不要指定此关键字。例如：`WHERE STATE = "PENDING"`。</li><li>如果在WHERE子句中指定了两个或三个过滤条件，您必须指定此关键字。例如：`WHERE LABEL = "label_name" AND STATE = "PENDING"`。</li></ul> |
| STATE                             | 否       | 加载作业的状态。加载方法不同时，状态也不同。<ul><li>Broker Load<ul><li>`PENDING`：已创建加载作业。</li><li>`QUEUEING`：加载作业在队列中等待调度。</li><li>`LOADING`：加载作业正在运行。</li><li>`PREPARED`：事务已提交。</li><li>`FINISHED`：加载作业成功。</li><li>`CANCELLED`：加载作业失败。</li></ul></li><li>Spark Load<ul><li>`PENDING`：您的StarRocks集群正在准备与ETL相关的配置，然后提交一个ETL作业到您的Apache Spark™集群。</li><li>`ETL`：您的Spark集群正在执行ETL作业，然后将数据写入相应的HDFS集群。</li><li>`LOADING`：HDFS集群中的数据正在加载到您的StarRocks集群，这意味着加载作业正在运行。</li><li>`PREPARED`：事务已提交。</li><li>`FINISHED`：加载作业成功。</li><li>`CANCELLED`：加载作业失败。</li></ul></li><li>INSERT<ul><li>`FINISHED`：加载作业成功。</li><li>`CANCELLED`：加载作业失败。</li></ul></li></ul>未指定`STATE`参数时，默认返回所有状态的加载作业信息。如果指定了`STATE`参数，则仅返回给定状态的加载作业信息。例如，`STATE = "PENDING"`返回`PENDING`状态的加载作业信息。 |
| ORDER BY field_name [ASC \| DESC] | 否       | 如果指定了此参数，则根据字段以升序或降序对输出进行排序。支持以下字段：`JobId`、`Label`、`State`、`Progress`、`Type`、`EtlInfo`、`TaskInfo`、`ErrorMsg`、    `CreateTime`、`EtlStartTime`、`EtlFinishTime`、`LoadStartTime`、`LoadFinishTime`、`URL`和`JobDetails`。<ul><li>要按升序对输出进行排序，请指定`ORDER BY field_name ASC`。</li><li>要按降序对输出进行排序，请指定`ORDER BY field_name DESC`。</li></ul>如果未指定字段和排序顺序，输出将默认按`JobId`的升序排序。 |
| LIMIT limit                       | 否       | 允许显示的加载作业数量。如果未指定此参数，则显示与过滤条件匹配的所有加载作业的信息。如果指定了此参数，例如`LIMIT 10`，则仅返回匹配过滤条件的10个加载作业的信息。 |
| OFFSET offset                     | 否       | `offset`参数定义要跳过的加载作业数量。例如，`OFFSET 5`跳过前5个加载作业并返回其余的部分。`offset`参数的值默认为`0`。 |

## 输出

```SQL
+-------+-------+-------+----------+------+---------+----------+----------+------------+--------------+---------------+---------------+----------------+-----+------------+
| JobId | Label | State | Progress | Type | Priority | EtlInfo | TaskInfo | ErrorMsg | CreateTime | EtlStartTime | EtlFinishTime | LoadStartTime | LoadFinishTime | URL | JobDetails |
+-------+-------+-------+----------+------+---------+----------+----------+------------+--------------+---------------+---------------+----------------+-----+------------+
```

该语句的输出根据加载方法不同而变化。

| **字段**   | **Broker Load**                                              | **Spark Load**                                               | **INSERT**                                                   |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| JobId      | StarRocks分配的用于识别StarRocks集群中加载作业的唯一ID。        | 与Broker Load作业中的含义相同。                                     | 与Broker Load作业中的含义相同。                                     |
| Label      | 加载作业的标签。在数据库中，加载作业的标签是唯一的，但在不同数据库中可能重复。 | 与Broker Load作业中的含义相同。                                     | 与Broker Load作业中的含义相同。                                     |
| State      | 加载作业的状态。<ul><li>`PENDING`：已创建加载作业。</li><li>`QUEUEING`：加载作业在队列中等待调度。</li><li>`LOADING`：加载作业正在运行。</li><li>`PREPARED`：事务已提交。</li><li>`FINISHED`：加载作业成功。</li><li>`CANCELLED`：加载作业失败。</li></ul> | 加载作业的状态。<ul><li>`PENDING`：StarRocks集群正在准备与ETL相关的配置，然后提交一个ETL作业到Spark集群。</li><li>`ETL`：Spark集群正在执行ETL作业，然后将数据写入相应的HDFS集群。</li><li>`LOADING`：HDFS集群中的数据正在加载到StarRocks集群，这意味着加载作业正在运行。</li><li>`PREPARED`：事务已提交。</li><li>`FINISHED`：加载作业成功。</li><li>`CANCELLED`：加载作业失败。</li></ul> | 加载作业的状态。<ul><li>`FINISHED`：加载作业成功。</li><li>`CANCELLED`：加载作业失败。</li></ul> |
| Progress       | The stage of the load job. A Broker Load job only has the `LOAD` stage, which ranges from 0% to 100% to describe the progress of the stage. When the load job enters the `LOAD` stage, `LOADING` is returned for the `State` parameter. A Broker Load job does not have the `ETL` stage. The `ETL` parameter is valid only for a Spark Load job.<br />**Note**<ul><li>The formula to calculate the progress of the `LOAD` stage: Number of StarRocks tables that complete data loading/Number of StarRocks tables that you plan to load data into * 100%.</li><li>When all data is loaded into StarRocks, `99%` is returned for the `LOAD` parameter. Then, loaded data starts taking effect in StarRocks. After the data takes effect, `100%` is returned for the `LOAD` parameter.</li><li>The progress of the `LOAD` stage is not linear. Therefore, the value of the `LOAD` parameter may not change over a period of time even if data loading is still ongoing.</li></ul> | The stage of the load job. A Spark Load job has two stages:<ul><li>`ETL`: ranges from 0% to 100% to describe the progress of the `ETL` stage.</li><li>`LOAD`: ranges from 0% to 100% to describe the progress of the `Load` stage. </li></ul>When the load job enters the `ETL` stage, `ETL` is returned for the `State` parameter. When the load job moves to the `LOAD` stage, `LOADING` is returned for the `State` parameter. <br />The **Note** is the same as those for Broker Load. | The stage of the load job. An INSERT job only has the `LOAD` stage, which ranges from 0% to 100% to describe the progress of the stage. When the load job enters the `LOAD` stage, `LOADING` is returned for the `State` parameter. An INSERT job does not have the `ETL` stage. The `ETL` parameter is valid only for a Spark Load job.<br />The **Note** is the same as those for Broker Load. |
| Type           | The method of the load job. The value of this parameter defaults to `BROKER`. | The method of the load job. The value of this parameter defaults to `SPARK`. | The method of the load job. The value of this parameter defaults to `INSERT`. |
| Priority       | The priority of the load job. Valid values: LOWEST, LOW, NORMAL, HIGH, and HIGHEST. | - | - |
| EtlInfo        | The metrics related to ETL.<ul><li>`unselected.rows`: The number of rows that are filtered out by the WHERE clause.</li><li>`dpp.abnorm.ALL`: The number of rows that are filtered out due to data quality issues, which refers to mismatches between source tables and StarRocks tables in, for example, the data type and the number of columns.</li><li>`dpp.norm.ALL`: The number of rows that are loaded into your StarRocks cluster.</li></ul>The sum of the preceding metrics is the total number of rows of raw data. You can use the following formula to calculate whether the percentage of unqualified data exceeds the value of the `max-filter-ratio` parameter:`dpp.abnorm.ALL`/(`unselected.rows` + `dpp.abnorm.ALL` + `dpp.norm.ALL`). | The field has the same meaning in a Spark Load job as it does in a Broker Load job. | The metrics related to ETL. An INSERT job does not have the `ETL` stage. Therefore, `NULL` is returned. |
| TaskInfo       | The parameters that are specified when you create the load job.<ul><li>`resource`: This parameter is valid only in a Spark Load job.</li><li>`timeout`: The time period that a load job is allowed to run. Unit: seconds.</li><li>`max-filter-ratio`: The largest percentage of rows that are filtered out due to data quality issues.</li></ul>For more information, see [BROKER LOAD](../data-manipulation/BROKER_LOAD.md). | The parameters that are specified when you create the load job.<ul><li>`resource`: The resource name.</li><li>`timeout`: The time period that a load job is allowed to run. Unit: seconds.</li><li>`max-filter-ratio`: The largest percentage of rows that are filtered out due to data quality issues.</li></ul>For more information, see [SPARK LOAD](../data-manipulation/SPARK_LOAD.md). | The parameters that are specified when you create the load job.<ul><li>`resource`: This parameter is valid only in a Spark Load job.</li><li>`timeout`: The time period that a load job is allowed to run. Unit: seconds.</li><li>`max-filter-ratio`: The largest percentage of rows that are filtered out due to data quality issues.</li></ul>For more information, see [INSERT](./INSERT.md). |
| ErrorMsg       | The error message returned when the load job fails. When the state of the loading job is `PENDING`, `LOADING`, or `FINISHED`, `NULL` is returned for the `ErrorMsg` field. When the state of the loading job is `CANCELLED`, the value returned for the `ErrorMsg` field consists of two parts: `type` and `msg`.<ul><li>The `type` part can be any of the following values:<ul><li>`USER_CANCEL`: The load job was manually canceled.</li><li>`ETL_SUBMIT_FAIL`: The load job failed to be submitted.</li><li>`ETL-QUALITY-UNSATISFIED`: The load job failed because the percentage of unqualified data exceeds the value of the `max-filter-ratio` parameter.</li><li>`LOAD-RUN-FAIL`: The load job failed in the `LOAD` stage.</li><li>`TIMEOUT`: The load job failed to finish within the specified timeout period.</li><li>`UNKNOWN`: The load job failed due to an unknown error.</li></ul></li><li>The `msg` part provides the detailed cause of the load failure.</li></ul> | The error message returned when the load job fails. When the state of the loading job is `PENDING`, `LOADING`, or `FINISHED`, `NULL` is returned for the `ErrorMsg` field. When the state of the loading job is `CANCELLED`, the value returned for the `ErrorMsg` field consists of two parts: `type` and `msg`.<ul><li>The `type` part can be any of the following values:<ul><li>`USER_CANCEL`: The load job was manually canceled.</li><li>`ETL_SUBMIT_FAIL`: StarRocks failed to submit an ETL job to Spark.</li><li>`ETL-RUN-FAIL`: Spark failed to execute the ETL job. </li><li>`ETL-QUALITY-UNSATISFIED`: The load job failed because the percentage of unqualified data exceeds the value of the `max-filter-ratio` parameter.</li><li>`LOAD-RUN-FAIL`: The load job failed in the `LOAD` stage.</li><li>`TIMEOUT`: The load job failed to finish within the specified timeout period.</li><li>`UNKNOWN`: The load job failed due to an unknown error.</li></ul></li><li>The `msg` part provides the detailed cause of the load failure.</li></ul> | The error message returned when the load job fails. When the state of the loading job is `FINISHED`, `NULL` is returned for the `ErrorMsg` field. When the state of the loading job is `CANCELLED`, the value returned for the `ErrorMsg` field consists of two parts: `type` and `msg`.<ul><li>The `type` part can be any of the following values:<ul><li>`USER_CANCEL`: The load job was manually canceled.</li><li>`ETL_SUBMIT_FAIL`: The load job failed to be submitted.</li><li>`ETL_RUN_FAIL`: The load job failed to run.</li><li>`ETL_QUALITY_UNSATISFIED`: The load job failed due to quality issues of raw data.</li><li>`LOAD-RUN-FAIL`: The load job failed in the `LOAD` stage.</li><li>`TIMEOUT`: The load job failed to finish within the specified timeout period.</li><li>`UNKNOWN`: The load job failed due to an unknown error.</li><li>`TXN_UNKNOWN`: The load job failed because the state of the transaction of the load job is unknown.</li></ul></li><li>The `msg` part provides the detailed cause of the load failure.</li></ul> |
| CreateTime     | The time at which the load job was created.                  | The field has the same meaning in a Spark Load job as it does in a Broker Load job. | The field has the same meaning in a INSERT job as it does in a Broker Load job. |
| EtlStartTime   | 经纪人加载作业没有`ETL`阶段。因此，该字段的值与`LoadStartTime`字段的值相同。 | `ETL`阶段开始的时间。                      | 插入作业没有`ETL`阶段。因此，该字段的值与`LoadStartTime`字段的值相同。 |
| EtlFinishTime  | 经纪人加载作业没有`ETL`阶段。因此，该字段的值与`LoadStartTime`字段的值相同。 | `ETL`阶段完成的时间。                      | 插入作业没有`ETL`阶段。因此，该字段的值与`LoadStartTime`字段的值相同。 |
| LoadStartTime  | `LOAD`阶段开始的时间。                   | 在 Spark 加载作业中，该字段的含义与在经纪人加载作业中相同。 | 在插入作业中，该字段的含义与在经纪人加载作业中相同。 |
| LoadFinishTime | 加载作业完成的时间。                     | 在 Spark 加载作业中，该字段的含义与在经纪人加载作业中相同。 | 在插入作业中，该字段的含义与在经纪人加载作业中相同。 |
| URL            | 用于访问加载作业中检测到的不合格数据的 URL。您可以使用`curl`或`wget`命令访问该 URL 并获取不合格数据。如果未检测到不合格数据，则返回`NULL`。 | 在 Spark 加载作业中，该字段的含义与在经纪人加载作业中相同。 | 在插入作业中，该字段的含义与在经纪人加载作业中相同。 |
| JobDetails     | 与加载作业相关的其他信息。<ul><li>`Unfinished backends`：未完成数据加载的 BE 的 ID。</li><li>`ScannedRows`：加载到 StarRocks 中的总行数以及被筛除的行数。</li><li>`TaskNumber`：加载作业可以分成一个或多个并发运行的任务。该字段指示加载任务的数量。</li><li>`All backends`：正在执行数据加载的 BE 的 ID。</li><li>`FileNumber`：源数据文件的数量。</li><li>`FileSize`：源数据文件的数据量。单位：字节。</li></ul> | 在 Spark 加载作业中，该字段的含义与在经纪人加载作业中相同。 | 在插入作业中，该字段的含义与在经纪人加载作业中相同。 |

## 用法说明

- `SHOW LOAD` 语句返回的信息在加载作业的`LoadFinishTime`后的 3 天内有效。3 天后，将无法显示该信息。您可以使用`label_keep_max_second`参数修改默认有效期。

    ```SQL
    ADMIN SET FRONTEND CONFIG ("label_keep_max_second" = "value");
    ```

- 如果`LoadStartTime`字段的值长时间为`N/A`，则表示加载作业严重积压。建议您减少创建加载作业的频率。
- 加载作业消耗的总时间段 = `LoadFinishTime`-`CreateTime`。
- 加载作业在`LOAD`阶段消耗的总时间段 = `LoadFinishTime`-`LoadStartTime`。

## 示例

示例 1：垂直显示当前数据库中的所有加载作业。

```plaintext
SHOW LOAD\G;
*************************** 1. row ***************************
         JobId: 976331
         Label: duplicate_table_with_null
         State: FINISHED
      Progress: ETL:100%; LOAD:100%
          Type: BROKER
      Priority: NORMAL
       EtlInfo: unselected.rows=0; dpp.abnorm.ALL=0; dpp.norm.ALL=65546
      TaskInfo: resource:N/A; timeout(s):300; max_filter_ratio:0.0
      ErrorMsg: NULL
    CreateTime: 2022-10-17 19:35:00
  EtlStartTime: 2022-10-17 19:35:04
 EtlFinishTime: 2022-10-17 19:35:04
 LoadStartTime: 2022-10-17 19:35:04
LoadFinishTime: 2022-10-17 19:35:06
           URL: NULL
    JobDetails: {"Unfinished backends":{"b90a703c-6e5a-4fcb-a8e1-94eca5be0b8f":[]},"ScannedRows":65546,"TaskNumber":1,"All backends":{"b90a703c-6e5a-4fcb-a8e1-94eca5be0b8f":[10004]},"FileNumber":1,"FileSize":548622}
```

示例 2：显示当前数据库中标签包含`null`字符串的两个加载作业。

```plaintext
SHOW LOAD 
WHERE LABEL LIKE "null" 
LIMIT 2;

+-------+---------------------------+----------+---------------------+--------+---------------------------------------------------------+---------------------------------------------------------------------------------------------------------+----------+---------------------+---------------------+---------------------+---------------------+---------------------+--------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| JobId | Label                     | State    | Progress            | Type   | EtlInfo                                                 | TaskInfo                                                                                                | ErrorMsg | CreateTime          | EtlStartTime        | EtlFinishTime       | LoadStartTime       | LoadFinishTime      | URL                                                                            | JobDetails                                                                                                                                                                                              |
+-------+---------------------------+----------+---------------------+--------+---------------------------------------------------------+---------------------------------------------------------------------------------------------------------+----------+---------------------+---------------------+---------------------+---------------------+---------------------+--------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 10082 | duplicate_table_with_null | FINISHED | ETL:100%; LOAD:100% | BROKER | unselected.rows=0; dpp.abnorm.ALL=0; dpp.norm.ALL=65546 | resource:N/A; timeout(s):300; max_filter_ratio:0.0                                                      | NULL     | 2022-08-02 14:53:27 | 2022-08-02 14:53:30 | 2022-08-02 14:53:30 | 2022-08-02 14:53:30 | 2022-08-02 14:53:31 | NULL                                                                           | {"Unfinished backends":{"4393c992-5da1-4e9f-8b03-895dc0c96dbc":[]},"ScannedRows":65546,"TaskNumber":1,"All backends":{"4393c992-5da1-4e9f-8b03-895dc0c96dbc":[10002]},"FileNumber":1,"FileSize":548622} |
| 10103 | unique_table_with_null    | FINISHED | ETL:100%; LOAD:100% | SPARK  | unselected.rows=0; dpp.abnorm.ALL=0; dpp.norm.ALL=65546 | resource:test_spark_resource_07af473a_1230_11ed_b483_00163e0e550b; timeout(s):300; max_filter_ratio:0.0 | NULL     | 2022-08-02 14:56:06 | 2022-08-02 14:56:19 | 2022-08-02 14:56:41 | 2022-08-02 14:56:41 | 2022-08-02 14:56:44 | http://emr-header-1.cluster-49091:20888/proxy/application_1655710334658_26391/ | {"Unfinished backends":{"00000000-0000-0000-0000-000000000000":[]},"ScannedRows":65546,"TaskNumber":1,"All backends":{"00000000-0000-0000-0000-000000000000":[-1]},"FileNumber":1,"FileSize":8790855}   |
+-------+---------------------------+----------+---------------------+--------+---------------------------------------------------------+---------------------------------------------------------------------------------------------------------+----------+---------------------+---------------------+---------------------+---------------------+---------------------+--------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+


示例 3：显示`example_db`中标签包含`table`字符串的加载作业。此外，返回的加载作业以`LoadStartTime`字段的降序方式显示。

```plaintext
SHOW LOAD FROM example_db 
WHERE LABEL LIKE "table" 
ORDER BY LoadStartTime DESC;

+-------+---------------------------+----------+---------------------+--------+---------------------------------------------------------+---------------------------------------------------------------------------------------------------------+----------+---------------------+---------------------+---------------------+---------------------+---------------------+--------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| JobId | Label                     | State    | Progress            | Type   | EtlInfo                                                 | TaskInfo                                                                                                | ErrorMsg | CreateTime          | EtlStartTime        | EtlFinishTime       | LoadStartTime       | LoadFinishTime      | URL                                                                            | JobDetails                                                                                                                                                                                              |
+-------+---------------------------+----------+---------------------+--------+---------------------------------------------------------+---------------------------------------------------------------------------------------------------------+----------+---------------------+---------------------+---------------------+---------------------+---------------------+--------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```plaintext
| 10103 | unique_table_with_null    | FINISHED | ETL:100%; LOAD:100% | SPARK  | unselected.rows=0; dpp.abnorm.ALL=0; dpp.norm.ALL=65546 | resource:test_spark_resource_07af473a_1230_11ed_b483_00163e0e550b; timeout(s):300; max_filter_ratio:0.0 | NULL     | 2022-08-02 14:56:06 | 2022-08-02 14:56:19 | 2022-08-02 14:56:41 | 2022-08-02 14:56:41 | 2022-08-02 14:56:44 | http://emr-header-1.cluster-49091:20888/proxy/application_1655710334658_26391/ | {"Unfinished backends":{"00000000-0000-0000-0000-000000000000":[]},"ScannedRows":65546,"TaskNumber":1,"All backends":{"00000000-0000-0000-0000-000000000000":[-1]},"FileNumber":1,"FileSize":8790855}   |
| 10082 | duplicate_table_with_null | FINISHED | ETL:100%; LOAD:100% | BROKER | unselected.rows=0; dpp.abnorm.ALL=0; dpp.norm.ALL=65546 | resource:N/A; timeout(s):300; max_filter_ratio:0.0                                                      | NULL     | 2022-08-02 14:53:27 | 2022-08-02 14:53:30 | 2022-08-02 14:53:30 | 2022-08-02 14:53:30 | 2022-08-02 14:53:31 | NULL                                                                           | {"Unfinished backends":{"4393c992-5da1-4e9f-8b03-895dc0c96dbc":[]},"ScannedRows":65546,"TaskNumber":1,"All backends":{"4393c992-5da1-4e9f-8b03-895dc0c96dbc":[10002]},"FileNumber":1,"FileSize":548622} |
+-------+---------------------------+----------+---------------------+--------+---------------------------------------------------------+----------------------------------------------------+----------+---------------------+---------------------+---------------------+---------------------+---------------------+--------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

例4：显示标签为`duplicate_table_with_null`且状态为`FINISHED`的加载作业在`example_db`中。

SHOW LOAD FROM example_db 
WHERE LABEL = "duplicate_table_with_null" AND STATE = "FINISHED";


+-------+---------------------------+----------+---------------------+--------+---------------------------------------------------------+----------------------------------------------------+----------+---------------------+---------------------+---------------------+---------------------+---------------------+------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| JobId | Label                     | State    | Progress            | Type   | EtlInfo                                                 | TaskInfo                                           | ErrorMsg | CreateTime          | EtlStartTime        | EtlFinishTime       | LoadStartTime       | LoadFinishTime      | URL  | JobDetails                                                                                                                                                                                              |
+-------+---------------------------+----------+---------------------+--------+---------------------------------------------------------+----------------------------------------------------+----------+---------------------+---------------------+---------------------+---------------------+---------------------+------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 10082 | duplicate_table_with_null | FINISHED | ETL:100%; LOAD:100% | BROKER | unselected.rows=0; dpp.abnorm.ALL=0; dpp.norm.ALL=65546 | resource:N/A; timeout(s):300; max_filter_ratio:0.0 | NULL     | 2022-08-02 14:53:27 | 2022-08-02 14:53:30 | 2022-08-02 14:53:30 | 2022-08-02 14:53:30 | 2022-08-02 14:53:31 | NULL | {"Unfinished backends":{"4393c992-5da1-4e9f-8b03-895dc0c96dbc":[]},"ScannedRows":65546,"TaskNumber":1,"All backends":{"4393c992-5da1-4e9f-8b03-895dc0c96dbc":[10002]},"FileNumber":1,"FileSize":548622} |
+-------+---------------------------+----------+---------------------+--------+---------------------------------------------------------+----------------------------------------------------+----------+---------------------+---------------------+---------------------+---------------------+---------------------+------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+


例5：跳过第一个加载作业并显示接下来的两个加载作业。此外，这两个加载作业按升序排序。

SHOW LOAD FROM example_db 
ORDER BY CreateTime ASC 
LIMIT 2 OFFSET 1;
```


或

```plaintext
SHOW LOAD FROM example_db 
ORDER BY CreateTime ASC 
LIMIT 1,2;
```

前述语句的输出如下。

```plaintext
+-------+---------------------------------------------+----------+---------------------+--------+---------------------------------------------------------+---------------------------------------------------------------------------------------------------------+----------+---------------------+---------------------+---------------------+---------------------+---------------------+--------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| JobId | Label                                       | State    | Progress            | Type   | EtlInfo                                                 | TaskInfo                                                                                                | ErrorMsg | CreateTime          | EtlStartTime        | EtlFinishTime       | LoadStartTime       | LoadFinishTime      | URL                                                                            | JobDetails                                                                                                                                                                                            |
+-------+---------------------------------------------+----------+---------------------+--------+---------------------------------------------------------+---------------------------------------------------------------------------------------------------------+----------+---------------------+---------------------+---------------------+---------------------+---------------------+--------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 10103 | unique_table_with_null                      | FINISHED | ETL:100%; LOAD:100% | SPARK  | unselected.rows=0; dpp.abnorm.ALL=0; dpp.norm.ALL=65546 | resource:test_spark_resource_07af473a_1230_11ed_b483_00163e0e550b; timeout(s):300; max_filter_ratio:0.0 | NULL     | 2022-08-02 14:56:06 | 2022-08-02 14:56:19 | 2022-08-02 14:56:41 | 2022-08-02 14:56:41 | 2022-08-02 14:56:44 | http://emr-header-1.cluster-49091:20888/proxy/application_1655710334658_26391/ | {"Unfinished backends":{"00000000-0000-0000-0000-000000000000":[]},"ScannedRows":65546,"TaskNumber":1,"All backends":{"00000000-0000-0000-0000-000000000000":[-1]},"FileNumber":1,"FileSize":8790855} |
| 10120 | insert_3a57b595-1230-11ed-b075-00163e14c85e | FINISHED | ETL:100%; LOAD:100% | INSERT | NULL                                                    | resource:N/A; timeout(s):3600; max_filter_ratio:0.0                                                     | NULL     | 2022-08-02 14:56:26 | 2022-08-02 14:56:26 | 2022-08-02 14:56:26 | 2022-08-02 14:56:26 | 2022-08-02 14:56:26 |                                                                                | {"Unfinished backends":{},"ScannedRows":0,"TaskNumber":0,"All backends":{},"FileNumber":0,"FileSize":0}                                                                                               |
+-------+---------------------------------------------+----------+---------------------+--------+---------------------------------------------------------+---------------------------------------------------------------------------------------------------------+----------+---------------------+---------------------+---------------------+---------------------+---------------------+--------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```