# CBO 统计信息

This article introduces the basic concept of the StarRocks CBO optimizer (Cost-based Optimizer) and how to collect statistics for the CBO optimizer to optimize query plans. StarRocks 2.4 introduces histograms as statistical information, providing more accurate data distribution statistics.

Prior to version 3.2, StarRocks only supported collecting statistics for internal tables. Starting from version 3.2, it also supports collecting statistics for Hive, Iceberg, and Hudi tables without relying on other systems.

## What is the CBO Optimizer

The CBO optimizer is crucial for query optimization. When a SQL query arrives at StarRocks, it is parsed into a logical execution plan, which is then rewritten and transformed by the CBO optimizer to generate multiple physical execution plans. By estimating the execution cost of each operator in the plan (such as CPU, memory, network, I/O, etc.), the CBO optimizer selects the query path with the lowest cost as the final physical query plan.

StarRocks CBO optimizer uses the Cascades framework to estimate costs based on various statistical information, allowing it to select the lowest-cost execution plan from tens of thousands of plans, thereby improving the efficiency and performance of complex queries. The self-developed CBO optimizer was introduced in version 1.16.0. Starting from version 1.19, this feature is enabled by default.

Statistical information is an important component of the CBO optimizer. The accuracy of statistical information determines the accuracy of cost estimation, which is crucial for CBO to select the optimal plan, and thereby determines the performance of the CBO optimizer. The following sections will provide detailed information on the types of statistical information, collection strategies, as well as how to create collection tasks and view statistical information.

## Types of Statistical Information

StarRocks collects various types of statistical information to provide reference for query optimization.

### Basic Statistical Information

StarRocks defaults to periodically collecting the following basic information for tables and columns:

- row_count: total number of rows in the table

- data_size: data size of the column

- ndv: cardinality of the column, i.e., the number of distinct values

- null_count: number of NULL values in the column

- min: minimum value of the column

- max: maximum value of the column

Complete statistical information is stored in the `column_statistics` table under the `_statistics_` database of the StarRocks cluster, which can be viewed under the `_statistics_` database. The query will return information similar to the following:

```sql

SELECT * FROM _statistics_.column_statistics\G

*************************** 1. row ***************************
      table_id: 10174
  partition_id: 10170
   column_name: is_minor
         db_id: 10169
    table_name: zj_test.source_wiki_edit
partition_name: p06
     row_count: 2
     data_size: 2
           ndv: NULL
    null_count: 0
           max: 1
           min: 0
```

### Histogram

Starting from version 2.4, StarRocks introduced histograms. Histograms are commonly used to estimate data distribution. When there is data skew, histograms can compensate for the bias in estimating basic statistical information and provide more accurate data distribution information.

StarRocks uses equi-height histograms, which select a certain number of buckets, with almost equal amounts of data in each bucket. For values with high frequency and significant impact on selectivity, StarRocks stores them in separate buckets. The more buckets there are, the higher the estimation accuracy of the histogram, but this also increases the memory usage of statistical information. You can adjust the number of buckets and the number of Most Common Values (MCV) collected for a single collection task according to your business requirements.

**Histograms are suitable for columns with significant data skew and frequent query requests. If the data distribution of your table is relatively uniform, histograms may not be necessary. Histograms support numerical types, DATE, DATETIME, or string types.**

Currently, histograms only support **manual sampling** for collection. Histogram statistical information is stored in the `histogram_statistics` table of the `_statistics_` database in the StarRocks cluster. The query will return information similar to the following:

```sql

SELECT * FROM _statistics_.histogram_statistics\G
*************************** 1. row ***************************
   table_id: 10174
column_name: added
      db_id: 10169
 table_name: zj_test.source_wiki_edit
    buckets: NULL
        mcv: [["23","1"],["5","1"]]
update_time: 2023-12-01 15:17:10.274000
```


## Collection Types

As the size and data distribution of tables are frequently updated due to import or deletion operations, it is necessary to periodically update statistical information to ensure that the information accurately reflects the actual data in the table. Before creating collection tasks, you need to decide which collection types and methods to use based on your business scenarios.

StarRocks supports full collection and sampled collection, both of which can be performed automatically or manually.

| **Collection Type** | **Collection Method** | **Collection Method Description**                                                                                                            | **Advantages and Disadvantages**                                          |
|----------|----------|-----------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------|
| Full Collection     | Automatic or Manual    | Scan the entire table to collect actual statistical information values. Collection is performed at the partition level, and the corresponding columns of basic statistical information are collected for each partition. If there are no data updates for the corresponding partition, the statistical information for that partition will not be collected during the next automatic collection, reducing resource usage. Full statistical information is stored in the `column_statistics` table in the `_statistics_` database.   | Advantages: Accurate statistical information helps the optimizer more accurately estimate execution plans. Disadvantages: It consumes a large amount of system resources, resulting in slower speed. Starting from version 2.4.5, users can configure the automatic collection time period to reduce resource consumption. |
| Sampled Collection     | Automatic or Manual    | Equally extract N rows of data from each partition of the table to calculate statistical information. Sampled statistical information is collected at the table level, and one statistical information is stored for each column of basic statistical information. The cardinality information (ndv) of the column is estimated globally based on the sampled sample, and will have a certain deviation from the actual cardinality information. Sampled statistical information is stored in the `_statistics_.table_statistic_v1` table. | Advantages: It consumes fewer system resources and is faster. Disadvantages: There is a certain error in statistical information, which may affect the accuracy of the optimizer's evaluation of execution plans.                      |

## Collecting Statistical Information

StarRocks provides flexible information collection methods, allowing users to choose automatic collection, manual collection, or customize automatic collection tasks based on their business scenarios.

默认情况下，StarRocks will periodically automatically collect full statistical information of tables. The default check interval is once every 5 minutes. If the data update ratio meets the condition, automatic collection will be triggered. **Full collection may consume a large amount of system resources**. If you do not want to use automatic full collection, you can set the FE configuration item `enable_collect_full_statistic` to `false`, and the system will stop automatic full collection, allowing you to customize the collection based on the custom tasks you create.

### Automatic Full Collection

For basic statistical information, StarRocks automatically performs full table statistical information collection without the need for manual operation. For tables that have never collected statistical information, the collection will be automatically performed within one scheduling cycle. For tables that have already collected statistical information, StarRocks will automatically update the total number of rows and the number of modified rows in the table, and persist these information periodically as the conditions for triggering automatic collection.

Starting from version 2.4.5, users are supported to configure the time period for automatic full collection to prevent cluster performance fluctuations caused by centralized collection. The collection time period can be configured using the two FE configuration items `statistic_auto_analyze_start_time` and `statistic_auto_analyze_end_time`.

Conditions for triggering automatic collection:

- Whether the table has undergone data changes after the last statistical information collection.
- Whether partition data has been modified. No recollection will be conducted for partitions that have not been modified.
- Whether the collection falls within the configured automatic collection time period (default is full-day collection, which can be modified).
- The healthiness of the statistical information for the table (`statistic_auto_collect_ratio`) is lower than the configured threshold.

> Healthiness calculation formula:
> 
> 1. When the number of updated partitions is less than 10: 1 - MIN(updated rows after the last statistical information collection / total rows)
> 
> 2. When the number of updated partitions is greater than or equal to 10: 1 - MIN(updated rows after the last statistical information collection / total rows, updated partitions after the last statistical information collection / total partitions)

Additionally, StarRocks has configured detailed strategies for tables with different update frequencies and sizes.

For tables with a relatively small amount of data, **StarRocks does not impose any restrictions by default, even if the update frequency of the table is high, real-time collection will still be performed**. The size threshold for small tables can be configured using `statistic_auto_collect_small_table_size`, or the collection interval for small tables can be configured using `statistic_auto_collect_small_table_interval`.

For tables with a relatively large amount of data, StarRocks imposes the following restrictions:

- The default collection interval is not less than 12 hours, which can be configured using `statistic_auto_collect_large_table_interval`.

- Under the condition of meeting the collection interval, when the healthiness is lower than the sampling threshold collection value, sampling collection will be triggered, which can be configured using `statistic_auto_collect_sample_threshold`.

- Under the condition of meeting the collection interval, when the healthiness is higher than the sampling threshold collection value but lower than the full collection threshold value, full collection will be triggered, which can be configured using `statistic_auto_collect_ratio`.

- When the maximum partition size collected exceeds 100G, sampling collection will be triggered, which can be configured using `statistic_max_full_collect_data_size`.

The automatic full collection task is automatically executed by the system with the default configuration as follows. You can modify it using the [ADMIN SET CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md) command.

Configuration items:

| **FE Configuration Item**                    | **Type** | **Default Value** | **Description**                                         |
|------------------------------------------|--------|----------------|--------------------------------------------------------|
| enable_statistic_collect                 | Boolean| TRUE           | Whether to collect statistical information. This parameter is turned on by default.                                    |
| enable_collect_full_statistic            | Boolean| TRUE           | Whether to enable automatic full statistical information collection. This parameter is turned on by default. |
| statistic_collect_interval_sec           | Long   | 300            | Interval for detecting data updates in automatic periodic tasks, default is 5 minutes. Unit: seconds.                     |
| statistic_auto_analyze_start_time        | String | 00:00:00       | Start time for configuring automatic full collection. Value range: `00:00:00` ~ `23:59:59`.             |
| statistic_auto_analyze_end_time          | String | 23:59:59       | End time for configuring automatic full collection. Value range: `00:00:00` ~ `23:59:59`.             |
| statistic_auto_collect_small_table_size  | Long   | 5368709120     | Small table threshold for automatic full collection tasks, default is 5 GB. Unit: Byte.                        |
| statistic_auto_collect_small_table_interval | Long   | 0         | Collection interval for small table automatic full collection tasks, in seconds.                        |
| statistic_auto_collect_large_table_interval | Long   | 43200          | Collection interval for large table automatic full collection tasks, default is 12 hours. Unit: seconds.                      |
| statistic_auto_collect_ratio             | Double | 0.8            | Healthiness threshold triggering automatic statistical information collection. Automatic collection is triggered if the healthiness of the statistical information is lower than this threshold.          |
| statistic_auto_collect_sample_threshold  | Double | 0.3            | Healthiness threshold triggering automatic sampling collection of statistical information. Automatic sampling collection is triggered if the healthiness of the statistical information is lower than this threshold.      |
| statistic_max_full_collect_data_size     | Long   | 107374182400   | Maximum partition size for automatic statistical information collection, default is 100 GB. Unit: Byte. If it exceeds this value, full collection will be abandoned and sampling collection will be performed instead. |
| statistic_full_collect_buffer             | Long   | 20971520       | Write buffer size for automatic full collection tasks, unit: Byte.                                 |
| statistic_collect_max_row_count_per_query | Long   | 5000000000     | Maximum number of data rows queried for statistical information collection at a time. The statistical information task is automatically split into multiple tasks according to this configuration. |
| statistic_collect_too_many_version_sleep  | Long   | 600000         | Sleep time for automatic collection tasks when there are too many write versions for the statistical information table (Too many tablet exceptions), unit: seconds. |

### Manual Collection

Manual collection tasks can be created using the ANALYZE TABLE statement. **By default, manual collection is synchronous. You can also set manual tasks to be asynchronous. After executing the command, the system will immediately return the status of the command, but the statistical information collection task will run asynchronously in the background. Asynchronous collection is suitable for scenarios with large table data volumes, while synchronous collection is suitable for scenarios with small table data volumes. Once created, manual tasks will only run once and do not need to be manually deleted. The running status can be viewed using SHOW ANALYZE STATUS**.

#### Manually Collecting Basic Statistical Information

```SQL
ANALYZE [FULL|SAMPLE] TABLE tbl_name (col_name [,col_name])
[WITH SYNC | ASYNC MODE]
[PROPERTIES (property [,property])]
```

Parameter description:

- Collection type
  - FULL: Full collection.
  - SAMPLE: Sampling collection.
  - If the collection type is not specified, it defaults to full collection.

- `WITH SYNC | ASYNC MODE`: If not specified, it defaults to synchronous collection.

- `col_name`: Columns for which to collect statistical information, use commas to separate multiple columns. If not specified, it means collecting information for the entire table.

- `PROPERTIES`: Custom parameters for the collection task. If not configured, the default configuration in `fe.conf` is used.

| **PROPERTIES**                | **Type** | **Default Value** | **Description**                           |
|-------------------------------|--------|---------|----------------------------------|
| statistic_sample_collect_rows | INT    | 200000  | Minimum sampling rows. If the parameter value exceeds the actual number of table rows, a full collection is performed by default. |

**Example**

- Manual full collection

```SQL
-- Manually collect statistical information for the specified table, using the default configuration.
ANALYZE TABLE tbl_name;

-- Manually collect statistical information for the specified table, using the default configuration.
ANALYZE FULL TABLE tbl_name;

-- Manually collect statistical information for the specified columns of the table, using the default configuration.
ANALYZE TABLE tbl_name(c1, c2, c3);
```

- Manual sampled collection

```SQL
-- Manually collect sampled statistical information for the specified table, using the default configuration.
ANALYZE SAMPLE TABLE tbl_name;

-- Manually collect sampled statistical information for the specified columns of the table, setting the sample rows.
ANALYZE SAMPLE TABLE tbl_name (v1, v2, v3) PROPERTIES(
    "statistic_sample_collect_rows" = "1000000"
);
```

#### Manually collecting histogram statistical information

```SQL
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON col_name [, col_name]
[WITH SYNC | ASYNC MODE]
[WITH N BUCKETS]
[PROPERTIES (property [,property])]
```

Parameter description:

- `col_name`: Columns for which to collect statistical information, multiple columns are separated by commas. This parameter is required.

- `WITH SYNC | ASYNC MODE`: If not specified, it defaults to synchronous collection.

- `WITH N BUCKETS`: `N` is the number of buckets in the histogram. If not specified, the default value in `fe.conf` is used.

- PROPERTIES: Custom parameters for the collection task. If not specified, the default configuration in `fe.conf` is used.

| **PROPERTIES**                 | **Type** | **Default Value**  | **Description**                           |
|--------------------------------|--------|----------|----------------------------------|
| statistic_sample_collect_rows  | INT    | 200000   | Minimum sampling rows. If the parameter value exceeds the actual number of table rows, a full collection is performed by default. |
| histogram_mcv_size             | INT    | 100      | The number of most common values (MCV) in the histogram. |
| histogram_sample_ratio         | FLOAT  | 0.1      | Histogram sampling ratio.                         |
| histogram_buckets_size                      | LONG    | 64           | Default number of histogram buckets.                                                                                             |
| histogram_max_sample_row_count | LONG   | 10000000 | Maximum sampling row count of the histogram.                       |

The sampling row count of the histogram is jointly controlled by multiple parameters, and the sampling row count takes the maximum value between `statistic_sample_collect_rows` and the total number of rows in the table `histogram_sample_ratio`. It does not exceed the number of rows specified by `histogram_max_sample_row_count`. If it exceeds, sampling is performed according to the upper limit number defined by this parameter.

The actual **PROPERTIES** used in the histogram task execution can be viewed through the **PROPERTIES** column in SHOW ANALYZE STATUS.

**Example**

```SQL
-- Manually collect histogram information for the v1 column using the default configuration.
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1;

-- Manually collect histogram information for the v1 and v2 columns, specify 32 buckets, 32 for MCV and a sampling ratio of 50%.
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1,v2 WITH 32 BUCKETS 
PROPERTIES(
   "histogram_mcv_size" = "32",
   "histogram_sample_ratio" = "0.5"
);
```

### Custom automatic collection (Custom Collection)

#### Create automatic collection task

Custom automatic collection tasks can be created using the CREATE ANALYZE statement.

Before creating a custom automatic collection task, it is necessary to disable automatic full collection `enable_collect_full_statistic=false`, otherwise the custom collection task will not take effect. After disabling `enable_collect_full_statistic=false`, **StarRocks will automatically create custom collection tasks, defaulting to collect statistics for all tables**.

```SQL
-- Periodically collect statistics for all databases.
CREATE ANALYZE [FULL|SAMPLE] ALL [PROPERTIES (property [,property])]

-- Periodically collect statistics for all tables in a specified database.
CREATE ANALYZE [FULL|SAMPLE] DATABASE db_name [PROPERTIES (property [,property])]

-- Periodically collect statistics for specified tables and columns.
CREATE ANALYZE [FULL|SAMPLE] TABLE tbl_name (col_name [,col_name]) [PROPERTIES (property [,property])]
```

Parameter description:

- Collection type
  - FULL: Full collection.
  - SAMPLE: Sampled collection.
  - If the collection type is not specified, it defaults to full collection.

- `col_name`: Columns for which to collect statistical information, use commas (,) to separate multiple columns. If not specified, it means collecting information for the entire table.

- `PROPERTIES`: Custom parameters for the collection task. If not configured, the default configuration in `fe.conf` is used.

| **PROPERTIES**                        | **Type** | **Default Value** | **Description**                                                                                                                           |
|---------------------------------------|--------|---------|----------------------------------------------------------------------------------------------------------------------------------|
| statistic_auto_collect_ratio          | FLOAT  | 0.8     | The health threshold for automatic statistical information collection. If the health of the statistics is less than this threshold, automatic collection is triggered.                                                                                            |
| statistics_max_full_collect_data_size | INT    | 100     | The maximum size of partitions for automatic statistical information collection. Unit: GB. If a partition exceeds this value, full statistical information collection is abandoned and a sampled statistical information collection is performed for the table.                                                                  |
| statistic_sample_collect_rows         | INT    | 200000  | The number of sample rows for sampled collection. If the parameter value exceeds the actual number of table rows, a full collection is performed.                                                                                              |
| statistic_exclude_pattern             | STRING | NULL    | Libraries and tables to be excluded during automatic collection, supporting regular expression matching in the format `database.table`.                                                                                  |
| statistic_auto_collect_interval       | LONG   |  0      | The interval for automatic collection, in seconds. StarRocks by default selects to use `statistic_auto_collect_small_table_interval` or `statistic_auto_collect_large_table_interval` based on the table size. If `statistic_auto_collect_interval` is specified in PROPERTIES when creating an Analyze task, the task will be executed according to this interval without using the parameters `statistic_auto_collect_small_table_interval` or `statistic_auto_collect_large_table_interval`.|

**Example**

**Automatic full collection**

```SQL
-- Periodically collect full statistics for all databases.
CREATE ANALYZE ALL;

-- Periodically collect statistics for all tables in a specified database.
CREATE ANALYZE DATABASE db_name;

-- Periodically collect statistics for all tables in a specified database with full collection.
CREATE ANALYZE FULL DATABASE db_name;

-- Periodically collect statistics for specified tables and columns.
CREATE ANALYZE TABLE tbl_name(c1, c2, c3); 
    
-- Periodically collect statistics for all databases, excluding the `db_name` database.
CREATE ANALYZE ALL PROPERTIES (
   "statistic_exclude_pattern" = "db_name\."
);    
```

**Automatic sampled collection**

```SQL
-- 定期抽样采集指定数据库下所有表的统计信息。
创建分析样本数据库 db_name；

-- 定期抽样采集指定表、列的统计信息，设置采样行数和健康度阈值。
创建分析样本表 tbl_name(c1, c2, c3) 参数 (
   "statistic_auto_collect_ratio" = "0.5",
   "statistic_sample_collect_rows" = "1000000"
);
    
-- 自动采集所有数据库的统计信息，不收集`db_name.tbl_name`表。
创建分析样本数据库 db_name 参数(
   "statistic_exclude_pattern" = "db_name\.tbl_name"
);    
```

#### 查看自动采集任务

```SQL
显示分析作业 [WHERE predicate]
```

您可以使用 WHERE 子句设定筛选条件，进行返回结果筛选。该语句会返回如下列。

| **列名**       | **说明**                                                         |
|--------------|----------------------------------------------------------------|
| Id           | 采集任务的ID。                                                       |
| Database     | 数据库名。                                                          |
| Table        | 表名。                                                            |
| Columns      | 列名列表。                                                          |
| Type         | 统计信息的类型。取值： FULL，SAMPLE。                                       |
| Schedule     | 调度的类型。自动采集任务固定为 `SCHEDULE`。                                    |
| Properties   | 自定义参数信息。                                                       |
| Status       | 任务状态，包括 PENDING（等待）、RUNNING（正在执行）、SUCCESS（执行成功）和 FAILED（执行失败）。 |
| LastWorkTime | 最近一次采集时间。                                                      |
| Reason       | 任务失败的原因。如果执行成功则为 `NULL`。                                       |

**示例**

```SQL
-- 查看集群全部自定义采集任务。
SHOW ANALYZE JOB

-- 查看数据库test下的自定义采集任务。
SHOW ANALYZE JOB where `database` = 'test';
```

#### 删除自动采集任务

```SQL
删除分析 <ID>
```

**示例**

```SQL
DROP ANALYZE 266030;
```

## 查看采集任务状态

您可以通过 SHOW ANALYZE STATUS 语句查看当前所有采集任务的状态。该语句不支持查看自定义采集任务的状态，如要查看，请使用 SHOW ANALYZE JOB。

```SQL
显示分析状态 [LIKE | WHERE predicate]
```

您可以使用 `Like` 或 `Where` 来筛选需要返回的信息。

目前 SHOW ANALYZE STATUS 会返回如下列。

| **列名**   | **说明**                                                     |
| ---------- | ------------------------------------------------------------ |
| Id         | 采集任务的ID。                                               |
| Database   | 数据库名。                                                   |
| Table      | 表名。                                                       |
| Columns    | 列名列表。                                                   |
| Type       | 统计信息的类型，包括 FULL，SAMPLE，HISTOGRAM。               |
| Schedule   | 调度的类型。`ONCE` 表示手动，`SCHEDULE` 表示自动。             |
| Status     | 任务状态，包括 RUNNING（正在执行）、SUCCESS（执行成功）和 FAILED（执行失败）。 |
| StartTime  | 任务开始执行的时间。                                         |
| EndTime    | 任务结束执行的时间。                                         |
| Properties | 自定义参数信息。                                             |
| Reason     | 任务失败的原因。如果执行成功则为 NULL。                      |

## 查看统计信息

### 基础统计信息元数据

```SQL
显示统计 META [WHERE predicate]
```

该语句返回如下列。

| **列名**   | **说明**                                            |
| ---------- | --------------------------------------------------- |
| Database   | 数据库名。                                          |
| Table      | 表名。                                              |
| Columns    | 列名。                                              |
| Type       | 统计信息的类型，`FULL` 表示全量，`SAMPLE` 表示抽样。 |
| UpdateTime | 当前表的最新统计信息更新时间。                      |
| Properties | 自定义参数信息。                                    |
| Healthy    | 统计信息健康度。                                    |

### 直方图统计信息元数据

```SQL
显示直方图 META [WHERE predicate]
```

该语句返回如下列。

| **列名**   | **说明**                                  |
| ---------- | ----------------------------------------- |
| Database   | 数据库名。                                |
| Table      | 表名。                                    |
| Column     | 列名。                                    |
| Type       | 统计信息的类型，直方图固定为 `HISTOGRAM`。 |
| UpdateTime | 当前表的最新统计信息更新时间。            |
| Properties | 自定义参数信息。                          |

## 删除统计信息

StarRocks 支持手动删除统计信息。手动删除统计信息时，会删除统计信息数据和统计信息元数据，并且会删除过期内存中的统计信息缓存。需要注意的是，如果当前存在自动采集任务，可能会重新采集之前已删除的统计信息。您可以使用 SHOW ANALYZE STATUS 查看统计信息采集历史记录。

### 删除基础统计信息

```SQL
删除统计 tbl_name
```

该命令将删除 `default_catalog._statistics_` 下表中存储的统计信息，FE cache 的对应表统计信息也将失效。

### 删除直方图统计信息

```SQL
ANALYZE TABLE tbl_name DROP HISTOGRAM ON col_name [, col_name]
```

## 取消采集任务

您可以通过 KILL ANALYZE 语句取消正在运行中（Running）的统计信息采集任务，包括手动采集任务和自定义自动采集任务。

手动采集任务的任务 ID 可以在 SHOW ANALYZE STATUS 中查看。自定义自动采集任务的任务 ID 可以在 SHOW ANALYZE JOB 中查看。

```SQL
KILL ANALYZE <ID>
```

## 其他 FE 配置项

| statistic_collect_concurrency               | INT     | 3            | 手动采集任务的最大并发数，默认为 3，即最多可以有 3 个手动采集任务同时运行。<br />超出的任务处于 PENDING 状态，等待调度。                                              |
| statistic_manager_sleep_time_sec            | LONG    | 60           | 统计信息相关元数据调度间隔周期。单位：秒。系统根据这个间隔周期，来执行如下操作：<ul><li>创建统计信息表；</li><li>删除已经被删除的表的统计信息；</li><li>删除过期的统计信息历史记录。</li></ul> |
| statistic_analyze_status_keep_second        | LONG    | 259200       | 采集任务记录保留时间，默认为 3 天。单位：秒。                                                                                          |

## 系统变量

`statistic_collect_parallel` 用于调整 BE 上能并发执行的统计信息收集任务的个数，默认值为 1，可以调大该数值来加快收集任务的执行速度。

## 收集 Hive/Iceberg/Hudi 表的统计信息

从 3.2 版本起，支持收集 Hive, Iceberg, Hudi 表的统计信息。**收集的语法和内表相同，但是只支持手动全量采集和自动全量采集两种方式，不支持抽样采集和直方图采集**。收集的统计信息会写入到 `_statistics_` 数据库的 `external_column_statistics` 表中，不会写入到 Hive Metastore 中，因此无法和其他查询引擎共用。您可以通过查询 `default_catalog._statistics_.external_column_statistics` 表中是否写入了表的统计信息。

查询时，会返回如下信息：

```sql
SELECT * FROM _statistics_.external_column_statistics\G
*************************** 1. row ***************************
    table_uuid: hive_catalog.tn_test.ex_hive_tbl.1673596430
partition_name: 
   column_name: k1
  catalog_name: hive_catalog
       db_name: tn_test
    table_name: ex_hive_tbl
     row_count: 3
     data_size: 12
           ndv: NULL
    null_count: 0
           max: 3
           min: 2
   update_time: 2023-12-01 14:57:27.137000
```

### Usage Limitations

When collecting statistics for Hive, Iceberg, and Hudi tables, the following limitations apply:

1. Currently, only full collection is supported, and sampling and histogram collection are not supported.
2. Currently, only statistics collection for Hive, Iceberg, and Hudi tables is supported.
3. For automatic collection tasks, only the statistics of specified tables are supported, and the statistics of all databases and tables under the databases are not supported.
4. For automatic collection tasks, only Hive and Iceberg tables can check whether the data has been updated every time, and the collection task will only be executed when the data has been updated, and only the partitions where the data has been updated will be collected. Hudi tables currently cannot determine whether the data has been updated, so the entire table will be collected periodically based on the collection interval.

The following examples assume the collection of statistical information for tables under the External Catalog specified. If statistical information for tables under External Catalog needs to be collected under the `default_catalog`, the table name can be referenced in the format `[catalog_name.][database_name.]<table_name>`.

### Manual Collection

#### Creating a Manual Collection Task

Syntax:

```sql
ANALYZE [FULL] TABLE tbl_name (col_name [,col_name])
[WITH SYNC | ASYNC MODE]
[PROPERTIES(property [,property])]
```

Example:

```sql
ANALYZE TABLE ex_hive_tbl(k1);
+----------------------------------+---------+----------+----------+
| Table                            | Op      | Msg_type | Msg_text |
+----------------------------------+---------+----------+----------+
| hive_catalog.tn_test.ex_hive_tbl | analyze | status   | OK       |
+----------------------------------+---------+----------+----------+
```

#### Viewing the Analyze Task Execution Status

Syntax:

```sql
SHOW ANALYZE STATUS [LIKE | WHERE predicate]
```

Example:

```sql
SHOW ANALYZE STATUS where `table` = 'ex_hive_tbl';
+-------+----------------------+-------------+---------+------+----------+---------+---------------------+---------------------+------------+--------+
| Id    | Database             | Table       | Columns | Type | Schedule | Status  | StartTime           | EndTime             | Properties | Reason |
+-------+----------------------+-------------+---------+------+----------+---------+---------------------+---------------------+------------+--------+
| 16400 | hive_catalog.tn_test | ex_hive_tbl | k1      | FULL | ONCE     | SUCCESS | 2023-12-04 16:31:42 | 2023-12-04 16:31:42 | {}         |        |
| 16465 | hive_catalog.tn_test | ex_hive_tbl | k1      | FULL | ONCE     | SUCCESS | 2023-12-04 16:37:35 | 2023-12-04 16:37:35 | {}         |        |
| 16467 | hive_catalog.tn_test | ex_hive_tbl | k1      | FULL | ONCE     | SUCCESS | 2023-12-04 16:37:46 | 2023-12-04 16:37:46 | {}         |        |
+-------+----------------------+-------------+---------+------+----------+---------+---------------------+---------------------+------------+--------+
```

#### Viewing Statistics Metadata

View the statistical information metadata for the table.

Syntax:

```sql
SHOW STATS META [WHERE predicate]
```

Example:

```sql
SHOW STATS META where `table` = 'ex_hive_tbl';
+----------------------+-------------+---------+------+---------------------+------------+---------+
| Database             | Table       | Columns | Type | UpdateTime          | Properties | Healthy |
+----------------------+-------------+---------+------+---------------------+------------+---------+
| hive_catalog.tn_test | ex_hive_tbl | k1      | FULL | 2023-12-04 16:37:46 | {}         |         |
+----------------------+-------------+---------+------+---------------------+------------+---------+
```

#### Cancelling Collection Tasks

Cancel the running (Running) statistics collection tasks.

Syntax:

```sql
KILL ANALYZE <ID>
```

The task ID can be viewed in SHOW ANALYZE STATUS.

### Automatic Collection

Create an automatic collection task, and StarRocks will periodically check whether the collection task needs to be executed, with a default check time of 5 minutes. Hive and Iceberg will only automatically execute a collection task when data updates are discovered. Hudi currently does not support data update detection, so it can only be collected periodically (the collection period is determined by the interval of the collection thread and the user-set collection interval, refer to the FE parameter adjustments below).

- statistic_collect_interval_sec

  The interval period for checking whether statistical information needs to be collected in automatic periodic tasks, default is 5 minutes.

- statistic_auto_collect_small_table_rows

  In automatic collection, the row count threshold for determining whether a table in an external data source (Hive, Iceberg, Hudi) is a small table, default value is 10,000,000 rows. This parameter is introduced in version 3.2.

- statistic_auto_collect_small_table_interval

  The collection interval for small tables in automatic collection, in seconds. Default value: 0.

- statistic_auto_collect_large_table_interval

  The collection interval for large tables in automatic collection, in seconds. Default value: 43,200 (12 hours).

The automatic collection thread checks whether there is a task to be executed every `statistic_collect_interval_sec` interval. If the number of rows in the table is less than `statistic_auto_collect_small_table_rows`, the collection interval is set to the collection interval for small tables; otherwise, it is set to the collection interval for large tables. If `last update time + collection interval > current time`, an update is required, otherwise it is not required. This can help avoid frequent execution of Analyze tasks for large tables.

#### Creating an Automatic Collection Task

Syntax:

```sql
CREATE ANALYZE TABLE tbl_name (col_name [,col_name])
[PROPERTIES (property [,property])]
```

Example:

```sql
CREATE ANALYZE TABLE ex_hive_tbl (k1)
PROPERTIES ("statistic_auto_collect_interval" = "5");

Query OK, 0 rows affected (0.01 sec)
```

Example:

#### Viewing Analyze Task Execution Status

Same as manual collection.

#### Viewing Statistics Metadata

Same as manual collection.

#### Viewing Automatic Collection Tasks

Syntax:

```sql
SHOW ANALYZE JOB [WHERE predicate]
```

Example:

```sql
SHOW ANALYZE JOB WHERE `id` = '17204';

Empty set (0.00 sec)
```

#### Cancelling Collection Tasks

Same as manual collection.

### Dropping Hive/Iceberg/Hudi Table Statistics

```sql
DROP STATS tbl_name
```

## More Information

- To query the values of FE configuration items, execute [ADMIN SHOW CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SHOW_CONFIG.md).

- To modify the values of FE configuration items, execute [ADMIN SET CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md).