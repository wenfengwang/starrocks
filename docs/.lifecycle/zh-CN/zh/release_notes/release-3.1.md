---
displayed_sidebar: "Chinese"
---

# StarRocks版本3.1

## 3.1.5

发布日期：2023年11月28日

### 新增特性

- 存算分离模式下CN节点支持数据导出。[#34018](https://github.com/StarRocks/starrocks/pull/34018)

### 功能优化

- [`INFORMATION_SCHEMA.COLUMNS`](../reference/information_schema/columns.md)表支持显示ARRAY、MAP、STRUCT类型的字段。[#33431](https://github.com/StarRocks/starrocks/pull/33431)
- 支持查询[Hive](../data_source/catalog/hive_catalog.md)中使用LZO算法压缩的Parquet、ORC、和CSV格式的文件。[#30923](https://github.com/StarRocks/starrocks/pull/30923)  [#30721](https://github.com/StarRocks/starrocks/pull/30721)
- 如果是自动分区表，也支持指定分区名进行更新，如果分区不存在则报错。[#34777](https://github.com/StarRocks/starrocks/pull/34777)
- 当创建物化视图涉及的表、视图及视图内涉及的表、物化视图发生Swap、Drop或者Schema Change操作后，物化视图可以进行自动刷新。[#32829](https://github.com/StarRocks/starrocks/pull/32829)
- 优化Bitmap相关的某些操作的性能，主要包括：
  - 优化Nested Loop Join性能。[#340804](https://github.com/StarRocks/starrocks/pull/34804)  [#35003](https://github.com/StarRocks/starrocks/pull/35003)
  - 优化`bitmap_xor`函数性能。[#34069](https://github.com/StarRocks/starrocks/pull/34069)
  - 支持Copy on Write（简称COW），优化性能，并减少内存使用。[#34047](https://github.com/StarRocks/starrocks/pull/34047)

### 问题修复

修复了如下问题：

- 如果提交的Broker Load作业包含过滤条件，在数据导入过程中，某些情况下会出现BE Crash。[#29832](https://github.com/StarRocks/starrocks/pull/29832)
- SHOW GRANTS时报`unknown error`。[#30100](https://github.com/StarRocks/starrocks/pull/30100)
- 如果使用表达式作为自动分区列，导入数据时可能会报错 "Error: The row create partition failed since Runtime error: failed to analyse partition value"。[#33513](https://github.com/StarRocks/starrocks/pull/33513)
- 查询时报错"get_applied_rowsets failed, tablet updates is in error state: tablet:18849 actual row size changed after compaction"。[#33246](https://github.com/StarRocks/starrocks/pull/33246)
- 存算一体模式下，单独查询Iceberg或者Hive外表容易出现BE crash。[#34682](https://github.com/StarRocks/starrocks/pull/34682)
- 存算一体模式下，导入数据时同时自动创建多个分区，偶尔会出现数据写错分区的情况。[#34731](https://github.com/StarRocks/starrocks/pull/34731)
- 长时间向持久化索引打开的主键模型表高频导入，可能会引起BE crash。[#33220](https://github.com/StarRocks/starrocks/pull/33220)
- 查询时报错"Exception: java.lang.IllegalStateException: null"。[#33535](https://github.com/StarRocks/starrocks/pull/33535)
- 执行`show proc '/current_queries';`时，如果某个查询刚开始执行， 可能会引起BE Crash。[#34316](https://github.com/StarRocks/starrocks/pull/34316)
- 向打开持久化索引的主键模型表中导入大量数据，有时会报错。[#34352](https://github.com/StarRocks/starrocks/pull/34352)
- 2.4及以下的版本升级到高版本，可能会出现Compaction Score很高的问题。[#34618](https://github.com/StarRocks/starrocks/pull/34618)
- 使用MariaDB ODBC Driver查询`INFORMATION_SCHEMA`中的信息时，`schemata`视图中`CATALOG_NAME`列中取值都显示的是`null`。[#34627](https://github.com/StarRocks/starrocks/pull/34627)
- 导入数据异常导致FE Crash后无法重启。[#34590](https://github.com/StarRocks/starrocks/pull/34590)
- Stream Load导入作业在**PREPARD**状态下、同时有Schema Change在执行，会导致数据丢失。[#34381](https://github.com/StarRocks/starrocks/pull/34381)
- 如果HDFS路径以两个或以上斜杠（`/`）结尾，HDFS备份恢复会失败。[#34601](https://github.com/StarRocks/starrocks/pull/34601)
- 打开`enable_load_profile`后，Stream Load会很容易失败。[#34544](https://github.com/StarRocks/starrocks/pull/34544)
- 使用列模式进行主键模型表部分列更新后，会有Tablet出现副本之间数据不一致。[#34555](https://github.com/StarRocks/starrocks/pull/34555)
- 使用ALTER TABLE增加`partition_live_number`属性没有生效。[#34842](https://github.com/StarRocks/starrocks/pull/34842)
- FE启动失败，报错"failed to load journal type 118"。[#34590](https://github.com/StarRocks/starrocks/pull/34590)
- 当`recover_with_empty_tablet`设置为`true`时可能会引起FE Crash。[#33071](https://github.com/StarRocks/starrocks/pull/33071)
- 副本操作重放失败可能会引起FE Crash。[#32295](https://github.com/StarRocks/starrocks/pull/32295)

### 兼容性变更

#### 配置参数

- 新增FE配置项[`enable_statistics_collect_profile`](../administration/Configuration.md#enable_statistics_collect_profile) 用于控制统计信息查询时是否生成Profile，默认值是`false`。[#33815](https://github.com/StarRocks/starrocks/pull/33815)
- FE配置项[`mysql_server_version`](../administration/Configuration.md#mysql_server_version)从静态变为动态（`mutable`），修改配置项设置后，无需重启FE即可在当前会话动态生效。[#34033](https://github.com/StarRocks/starrocks/pull/34033)
- 新增BE/CN配置项[`update_compaction_ratio_threshold`](../administration/Configuration.md#update_compaction_ratio_threshold)，用于手动设置存算分离模式下主键模型表单次Compaction合并的最大数据比例，默认值是`0.5`。如果单个Tablet过大，建议适当调小该配置项取值。存算一体模式下主键模型表单次Compaction合并的最大数据比例仍然保持原来自动调整模式。[#35129](https://github.com/StarRocks/starrocks/pull/35129)

#### 系统变量

- Add session variable `cbo_decimal_cast_string_strict` to optimize the behavior of the optimizer controlling the conversion of DECIMAL type to STRING type. When the value is `true`, the processing logic of v2.5.x and later versions is used, and strict conversion (i.e., truncation and padding with `0` according to Scale) is performed. When the value is `false`, the processing logic of v2.5.x and earlier versions is retained (i.e., processed according to significant figures). The default value is `true`. [#34208](https://github.com/StarRocks/starrocks/pull/34208)
- Add session variable `cbo_eq_base_type` to specify the forced type when comparing DECIMAL type and STRING type data, defaulting to `VARCHAR`, or optionally `DECIMAL`. [#34208](https://github.com/StarRocks/starrocks/pull/34208)
- Add session variable `big_query_profile_second_threshold` to generate a Profile when the session variable [`enable_profile`](../reference/System_variable.md#enable_profile) is set to `false` and the query time exceeds the threshold set by `big_query_profile_second_threshold`. [#33825](https://github.com/StarRocks/starrocks/pull/33825)

## 3.1.4

Release Date: November 2, 2023

### New Features

- The primary key model under storage-and-compute separation supports Sort Key for tables.
- Asynchronous materialized views support specifying partition expressions through the str2date function, which can be used for incremental refresh and query rewrite of materialized views with external table partition type as STRING type data. [#29923](https://github.com/StarRocks/starrocks/pull/29923) [#31964](https://github.com/StarRocks/starrocks/pull/31964)
- Add session variable [`enable_query_tablet_affinity`](../reference/System_variable.md#enable_query_tablet_affinity25-及以后) to control selecting a fixed replica when querying the same tablet multiple times, defaulting to off. [#33049](https://github.com/StarRocks/starrocks/pull/33049)
- Add utility function `is_role_in_session` to check if a specified role is active in the current session, and support viewing the activation of nested roles. [#32984](https://github.com/StarRocks/starrocks/pull/32984)
- Add resource group-level query queue, which needs to be enabled through the global variable `enable_group_level_query_queue` (default value is `false`). When the resource consumption of either global granularity or resource group granularity reaches the threshold, queries will be queued until all resource consumption does not exceed the threshold before execution.
  - Each resource group can set `concurrency_limit` to limit the maximum concurrent queries in a single BE node.
  - Each resource group can set `max_cpu_cores` to limit the maximum CPU usage in a single BE node.
- Resource group classifier adds two parameters: `plan_cpu_cost_range` and `plan_mem_cost_range`.
  - `plan_cpu_cost_range`: the estimated query CPU cost range by the system. Default is `NULL`, indicating no such limitation.
  - `plan_mem_cost_range`: the estimated query memory cost range by the system. Default is `NULL`, indicating no such limitation.

### Functionality Optimization

- Window functions COVAR_SAMP, COVAR_POP, CORR, VARIANCE, VAR_SAMP, STD, STDDEV_SAMP support ORDER BY clause and Window clause. [#30786](https://github.com/StarRocks/starrocks/pull/30786)
- When there is a query result overflow for DECIMAL type data, an error is returned instead of NULL. [#30419](https://github.com/StarRocks/starrocks/pull/30419)
- The concurrent query quantity of the query queue is managed by the Leader FE, and each Follower FE will notify the Leader FE when initiating and ending a query. If it exceeds the `concurrency_limit` of global or resource group granularity, the query will be rejected or enter the query queue.

### Issue Fixes

Fixed the following issues:

- Due to inaccurate memory statistics, there is a chance of errors when Spark or Flink reads data. [#30702](https://github.com/StarRocks/starrocks/pull/30702) [#30751](https://github.com/StarRocks/starrocks/pull/30751)
- Inaccurate memory usage statistics of Metadata Cache. [#31978](https://github.com/StarRocks/starrocks/pull/31978)
- Calling libcurl can cause a BE Crash. [#31667](https://github.com/StarRocks/starrocks/pull/31667)
- Refreshing StarRocks materialized views created based on Hive views will report an error "java.lang.ClassCastException: com.starrocks.catalog.HiveView cannot be cast to com.starrocks.catalog.HiveMetaStoreTable". [#31004](https://github.com/StarRocks/starrocks/pull/31004)
- Error when ORDER BY clause contains aggregate functions "java.lang.IllegalStateException: null". [#30108](https://github.com/StarRocks/starrocks/pull/30108)
- Under the storage-and-compute separation mode, the Key information of the table is not recorded in `information_schema.COLUMNS`, resulting in the inability to execute DELETE operations when importing data using Flink Connector. [#31458](https://github.com/StarRocks/starrocks/pull/31458)
- When importing data using Flink Connector, if the concurrency is high and HTTP and Scan thread counts are limited, it may get stuck. [#32251](https://github.com/StarRocks/starrocks/pull/32251)
- If a field of a smaller byte type is added, an error "error: invalid field name" will be reported after the change and executing SELECT COUNT(*). [#33243](https://github.com/StarRocks/starrocks/pull/33243)
- Query Cache enabled results in incorrect query results. [#32781](https://github.com/StarRocks/starrocks/pull/32781)
- Failure in Hash Join during query results in a BE Crash. [#32219](https://github.com/StarRocks/starrocks/pull/32219)
- BINARY or VARBINARY types in `column` view of `information_schema.` show `DATA_TYPE` and `COLUMN_TYPE` as `unknown`. [#32678](https://github.com/StarRocks/starrocks/pull/32678)

### Behavioral Changes

- Starting from version 3.1.4, the default persistence index for the primary key model in newly built clusters will be enabled when creating tables (unchanged when upgrading from a lower version to version 3.1.4). [#33374](https://github.com/StarRocks/starrocks/pull/33374)
- Add FE parameter `enable_sync_publish` and enable it by default. When set to `true`, the Publish process for importing primary key model tables will wait for Apply to complete before returning the results, making the imported data immediately visible after the import job returns successful, but it may cause a delay compared to the previous asynchronous Apply during the Publish process. (Previously, there was no such parameter, and Apply was asynchronous during the Publish process). [#27055](https://github.com/StarRocks/starrocks/pull/27055)

## 3.1.3

Release Date: September 25, 2023

### New Features

- The primary key model under storage-and-compute separation supports persistent indexes based on local disk, consistent with the usage of storage-and-compute integration.
- Aggregation function [group_concat](../sql-reference/sql-functions/string-functions/group_concat.md) supports using DISTINCT keyword and ORDER BY clause. [#28778](https://github.com/StarRocks/starrocks/pull/28778)
- [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Kafka Connector](../loading/Kafka-connector-starrocks.md)、[Flink Connector](../loading/Flink-connector-starrocks.md)和[Spark Connector](../loading/Spark-connector-starrocks.md)都支持在对主键模型表进行部分列更新时启用列模式。[#28288](https://github.com/StarRocks/starrocks/pull/28288)
- 分区中的数据可以随着时间推移自动进行降温操作（[List 分区方式](../table_design/list_partitioning.md)目前还不支持）。[#29335](https://github.com/StarRocks/starrocks/pull/29335) [#29393](https://github.com/StarRocks/starrocks/pull/29393)

### 功能优化

执行含有不合法注释的SQL命令将返回与MySQL一致的结果。[#30210](https://github.com/StarRocks/starrocks/pull/30210)

### 问题修复

解决了以下问题：

- 在执行 [DELETE](../sql-reference/sql-statements/data-manipulation/DELETE.md) 语句时，如果WHERE条件中的字段类型是[BITMAP](../sql-reference/sql-statements/data-types/BITMAP.md)或[HLL](../sql-reference/sql-statements/data-types/HLL.md)，将导致语句执行失败。[#28592](https://github.com/StarRocks/starrocks/pull/28592)
- 若某个跟随的FE重启后，由于CpuCores不同步，将导致查询性能受到影响。[#28472](https://github.com/StarRocks/starrocks/pull/28472)  [#30434](https://github.com/StarRocks/starrocks/pull/30434)
- [to_bitmap()](../sql-reference/sql-functions/bitmap-functions/to_bitmap.md)函数的成本统计计算不正确，导致物化视图改写后选择了错误的执行计划。[#29961](https://github.com/StarRocks/starrocks/pull/29961)
- 在存储计算分离架构的特定场景下，当跟随的FE重启后，发送到该FE的查询会返回错误信息“Backend node not found. Check if any backend node is down”。[#28615](https://github.com/StarRocks/starrocks/pull/28615)
- 在[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)过程中持续导入数据可能会报错“Tablet is in error state”。[#29364](https://github.com/StarRocks/starrocks/pull/29364)
- 在线修改FE动态参数`max_broker_load_job_concurrency`会导致无效。[#29964](https://github.com/StarRocks/starrocks/pull/29964) [#29720](https://github.com/StarRocks/starrocks/pull/29720)
- 调用[date_diff()](../sql-reference/sql-functions/date-time-functions/date_diff.md)函数时，如果函数中指定的时间单位是个常量而日期是非常量，会导致BE崩溃。[#29937](https://github.com/StarRocks/starrocks/issues/29937)
- 如果[Hive Catalog](../data_source/catalog/hive_catalog.md)是多级目录，且数据存储在腾讯云COS中，会导致查询结果不正确。[#30363](https://github.com/StarRocks/starrocks/pull/30363)
- 在存储计算分离架构下，开启数据异步写入后，自动分区不生效。[#29986](https://github.com/StarRocks/starrocks/issues/29986)
- 通过[CREATE TABLE LIKE](../sql-reference/sql-statements/data-definition/CREATE_TABLE_LIKE.md)语句创建主键模型表时，会报错`Unexpected exception: Unknown properties: {persistent_index_type=LOCAL}`。[#30255](https://github.com/StarRocks/starrocks/pull/30255)
- 主键模型表恢复之后，BE重启后元数据发生错误，导致元数据不一致。[#30135](https://github.com/StarRocks/starrocks/pull/30135)
- 在主键模型表导入时，若Truncate操作和查询并发，在某些情况下会报错“java.lang.NullPointerException”。[#30573](https://github.com/StarRocks/starrocks/pull/30573)
- 当物化视图创建语句中包含谓词表达式时，刷新结果不正确。[#29904](https://github.com/StarRocks/starrocks/pull/29904)
- 升级到3.1.2版本后，原来创建的表中Storage Volume属性被重置为“null”。[#30647](https://github.com/StarRocks/starrocks/pull/30647)
- Tablet元数据进行Checkpoint与Restore操作并行时，会导致某些副本丢失不可查。[#30603](https://github.com/StarRocks/starrocks/pull/30603)
- 如果表字段为“NOT NULL”但没有设置默认值，使用CloudCanal导入时会报错“Unsupported dataFormat value is: \N”。[#30799](https://github.com/StarRocks/starrocks/pull/30799)

### 行为变更

- 聚合函数[group_concat](../sql-reference/sql-functions/string-functions/group_concat.md)的分隔符必须使用`SEPARATOR`关键字声明。
- 会话变量[`group_concat_max_len`](../reference/System_variable.md#group_concat_max_len)（用于控制聚合函数[group_concat](../sql-reference/sql-functions/string-functions/group_concat.md)可以返回的字符串最大长度）的默认值从原来的无限制变更为默认`1024`。

## 3.1.2

发布日期：2023年8月25日

### 问题修复

解决了以下问题：

- 用户在连接时指定默认数据库，并且仅有该数据库下的表权限，但没有该数据库的权限时，会报无法访问该数据库的权限。[#29767](https://github.com/StarRocks/starrocks/pull/29767)
- RESTful API`show_data`对于云原生表的返回信息不正确。[#29473](https://github.com/StarRocks/starrocks/pull/29473)
- 在[array_agg()](../sql-reference/sql-functions/array-functions/array_agg.md)函数运行过程中，如果查询取消，则会发生BE崩溃。[#29400](https://github.com/StarRocks/starrocks/issues/29400)
- [BITMAP](../sql-reference/sql-statements/data-types/BITMAP.md)和[HLL](../sql-reference/sql-statements/data-types/HLL.md)类型的列在[SHOW FULL COLUMNS](../sql-reference/sql-statements/Administration/SHOW_FULL_COLUMNS.md)查询结果中返回的`Default`字段值不正确。[#29510](https://github.com/StarRocks/starrocks/pull/29510)
- [array_map()](../sql-reference/sql-functions/array-functions/array_map.md)同时涉及多个表时，下推策略问题导致查询失败。[#29504](https://github.com/StarRocks/starrocks/pull/29504)
- 由于未合入上游Apache ORC的BugFix ORC-1304（[apache/orc#1299](https://github.com/apache/orc/pull/1299)），导致ORC文件查询失败。[#29804](https://github.com/StarRocks/starrocks/pull/29804)

### 行为变更

从此版本开始，执行SET CATALOG操作必须要有目标Catalog的USAGE权限。您可以使用[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)命令进行授权操作。

如果是从低版本升级上来的集群，已经做好了已有用户的升级逻辑，不需要重新赋权。[#29389](https://github.com/StarRocks/starrocks/pull/29389)。如果是新增授权，则需要注意赋予目标Catalog的USAGE权限。

## 3.1.1

### 发布日期：2023 年 8 月 18 日

### 新增特性

- 在[存算分离架构](../deployment/shared_data/s3.md)下，增加了以下功能：
  - 数据存储在 Azure Blob Storage 上。
  - 列出分区。
- 新增支持 [COVAR_SAMP](../sql-reference/sql-functions/aggregate-functions/covar_samp.md)、[COVAR_POP](../sql-reference/sql-functions/aggregate-functions/covar_pop.md)、[CORR](../sql-reference/sql-functions/aggregate-functions/corr.md) 聚合函数。
- 支持[窗口函数](../sql-reference/sql-functions/Window_function.md) COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、STDDEV_SAMP。

### 功能优化

对所有复合谓词以及 WHERE 子句中的表达式支持隐式转换，可通过[会话变量](../reference/System_variable.md) `enable_strict_type` 控制是否打开隐式转换（默认取值为 `false`）。

### 问题修复

修复了以下问题：

- 向多副本的表中导入数据时，如果某些分区没有数据，则会写入很多无用日志。 [#28824](https://github.com/StarRocks/starrocks/issues/28824)
- 主键模型表部分列更新时平均 row size 预估不准导致内存占用过多。 [#27485](https://github.com/StarRocks/starrocks/pull/27485)
- 某个 Tablet 出现某种 ERROR 状态之后触发 Clone 操作，会导致磁盘使用上升。 [#28488](https://github.com/StarRocks/starrocks/pull/28488)
- Compaction 会触发冷数据写入 Local Cache。 [#28831](https://github.com/StarRocks/starrocks/pull/28831)

## 3.1.0

### 发布日期：2023 年 8 月 7 日

### 新增特性

#### 存算分离架构

- 新增支持主键模型（Primary Key）表，暂不支持持久化索引。
- 支持自增列属性 [AUTO_INCREMENT](../sql-reference/sql-statements/auto_increment.md)，提供表内全局唯一 ID，简化数据管理。
- 支持[导入时自动创建分区和使用分区表达式定义分区规则](../table_design/expression_partitioning.md)，提高了分区创建的易用性和灵活性。
- 支持[存储卷（Storage Volume）抽象](../deployment/shared_data/s3.md)，方便在存算分离架构中配置存储位置及鉴权等相关信息。后续创建库表时可以直接引用，提升易用性。

#### 数据湖分析

- 支持访问 [Hive Catalog](../data_source/catalog/hive_catalog.md) 内的视图。
- 支持访问 Parquet 格式的 Iceberg v2 数据表。
- 支持[写出数据到 Parquet 格式的 Iceberg 表](../data_source/catalog/iceberg_catalog.md#向-iceberg-表中插入数据)。
- 【公测中】支持通过外部 [Elasticsearch catalog](../data_source/catalog/elasticsearch_catalog.md) 访问 Elasticsearch，简化外表创建等过程。
- 【公测中】支持 [Paimon catalog](../data_source/catalog/paimon_catalog.md)，帮助用户使用 StarRocks 对流式数据进行湖分析。

#### 存储、导入与查询

- 自动创建分区功能升级为[表达式分区](../table_design/expression_partitioning.md)，建表时只需要使用简单的分区表达式（时间函数表达式或列表达式）即可配置需要的分区方式，并且数据导入时 StarRocks 会根据数据和分区表达式的定义规则自动创建分区。这种创建分区的方式，更加灵活易用，能满足大部分场景。
- 支持 [List 分区](../table_design/list_partitioning.md)。数据按照分区列枚举值列表进行分区，可以加速查询和高效管理分类明确的数据。
- `Information_schema` 库新增表 `loads`，支持查询 [Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 和 [Insert](../sql-reference/sql-statements/data-manipulation/INSERT.md) 作业的结果信息。
- [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) 支持打印因数据质量不合格而过滤掉的错误数据行，通过 `log_rejected_record_num` 参数设置允许打印的最大数据行数。
- 支持[随机分桶（Random Bucketing）](../table_design/Data_distribution.md#设置分桶)功能，建表时无需选择分桶键，StarRocks 将导入数据随机分发到各个分桶中，同时配合使用 2.5.7 版本起支持的自动设置分桶数量 (`BUCKETS`) 功能，用户可以不再关心分桶配置，大大简化建表语句。不过，在大数据、高性能要求场景中，建议继续使用 Hash 分桶方式，借助分桶裁剪来加速查询。
- 支持在 [INSERT INTO](../loading/InsertInto.md) 语句中使用表函数 FILES()，从 AWS S3 或 HDFS 直接导入 Parquet 或 ORC 格式文件的数据。FILES() 函数会自动进行表结构 (Table Schema) 推断，不再需要提前创建 External Catalog 或文件外部表，大大简化导入过程。
- 支持[生成列（Generated Column）](../sql-reference/sql-statements/generated_columns.md)功能，自动计算生成列表达式的值并存储，且在查询时可自动改写，以提升查询性能。
- 支持通过 [Spark connector](../loading/Spark-connector-starrocks.md) 导入 Spark 数据至 StarRocks。相较于 [Spark Load](../loading/SparkLoad.md)，Spark connector 能力更完善。您可以自定义 Spark 作业，对数据进行 ETL 操作，Spark connector 只作为 Spark 作业中的 sink。
- 支持导入数据到 [MAP](../sql-reference/sql-statements/data-types/Map.md)、[STRUCT](../sql-reference/sql-statements/data-types/STRUCT.md) 类型的字段，并且在 ARRAY、MAP、STRUCT 类型中支持了 Fast Decimal 类型。

#### SQL 语句和函数

- 增加 Storage Volume 相关 SQL 语句：[CREATE STORAGE VOLUME](../sql-reference/sql-statements/Administration/CREATE_STORAGE_VOLUME.md)、[ALTER STORAGE VOLUME](../sql-reference/sql-statements/Administration/ALTER_STORAGE_VOLUME.md)、[DROP STORAGE VOLUME](../sql-reference/sql-statements/Administration/DROP_STORAGE_VOLUME.md)、[SET DEFAULT STORAGE VOLUME](../sql-reference/sql-statements/Administration/SET_DEFAULT_STORAGE_VOLUME.md)、[DESC STORAGE VOLUME](../sql-reference/sql-statements/Administration/DESC_STORAGE_VOLUME.md)、[SHOW STORAGE VOLUMES](../sql-reference/sql-statements/Administration/SHOW_STORAGE_VOLUMES.md)。

- 支持通过 [ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md) 修改表的注释。[#21035](https://github.com/StarRocks/starrocks/pull/21035)

- 新增如下函数：

  - Struct 函数：[struct (row)](../sql-reference/sql-functions/struct-functions/row.md)、[named_struct](../sql-reference/sql-functions/struct-functions/named_struct.md)
- Map functions: [str_to_map](../sql-reference/sql-functions/string-functions/str_to_map.md), [map_concat](../sql-reference/sql-functions/map-functions/map_concat.md), [map_from_arrays](../sql-reference/sql-functions/map-functions/map_from_arrays.md), [element_at](../sql-reference/sql-functions/map-functions/element_at.md), [distinct_map_keys](../sql-reference/sql-functions/map-functions/distinct_map_keys.md), [cardinality](../sql-reference/sql-functions/map-functions/cardinality.md)
  - Map higher-order functions: [map_filter](../sql-reference/sql-functions/map-functions/map_filter.md), [map_apply](../sql-reference/sql-functions/map-functions/map_apply.md), [transform_keys](../sql-reference/sql-functions/map-functions/transform_keys.md), [transform_values](../sql-reference/sql-functions/map-functions/transform_values.md)
  - Array functions: [array_agg](../sql-reference/sql-functions/array-functions/array_agg.md) supports `ORDER BY`, [array_generate](../sql-reference/sql-functions/array-functions/array_generate.md), [element_at](../sql-reference/sql-functions/array-functions/element_at.md), [cardinality](../sql-reference/sql-functions/array-functions/cardinality.md)
  - Array higher-order functions: [all_match](../sql-reference/sql-functions/array-functions/all_match.md), [any_match](../sql-reference/sql-functions/array-functions/any_match.md)
  - Aggregate functions: [min_by](../sql-reference/sql-functions/aggregate-functions/min_by.md), [percentile_disc](../sql-reference/sql-functions/aggregate-functions/percentile_disc.md)
  - Table functions: [FILES](../sql-reference/sql-functions/table-functions/files.md), [generate_series](../sql-reference/sql-functions/table-functions/generate_series.md)
  - Date functions: [next_day](../sql-reference/sql-functions/date-time-functions/next_day.md), [previous_day](../sql-reference/sql-functions/date-time-functions/previous_day.md), [last_day](../sql-reference/sql-functions/date-time-functions/last_day.md), [makedate](../sql-reference/sql-functions/date-time-functions/makedate.md), [date_diff](../sql-reference/sql-functions/date-time-functions/date_diff.md)
  - Bitmap functions: [bitmap_subset_limit](../sql-reference/sql-functions/bitmap-functions/bitmap_subset_limit.md), [bitmap_subset_in_range](../sql-reference/sql-functions/bitmap-functions/bitmap_subset_in_range.md)
  - Math functions: [cosine_similarity](../sql-reference/sql-functions/math-functions/cos_similarity.md), [cosine_similarity_norm](../sql-reference/sql-functions/math-functions/cos_similarity_norm.md)

#### Permissions and Security

  Add related [permissions](../administration/privilege_item.md#存储卷权限-storage-volume) for Storage Volume and related [permissions](../administration/privilege_item.md#数据目录权限-catalog) for External Catalog, supporting the granting and revocation of related permissions through `GRANT` and `REVOKE` statements.

### Feature Optimization

#### Storage and Compute Separation Architecture

  Optimize the data caching function in the storage and compute separation architecture, which allows specifying the range of hot data to prevent cold data queries from occupying the Local Disk Cache and affecting the efficiency of hot data queries.

#### Materialized Views

- Optimize the creation of asynchronous materialized views:
  - Support random bucketing, which defaults to random bucketing when creating a materialized view without specifying the bucketing column.
  - Support specifying the sort key through `ORDER BY`.
  - Support using attributes such as `colocate_group`, `storage_medium`, and `storage_cooldown_time`.
  - Support using session variables to flexibly adjust the execution strategy for view refreshing by setting `properties("session.<variable_name>"="<value>")`.
  - By default, enable the Spill feature for refreshing all materialized views, with a query timeout of 1 hour.
  - Support creating materialized views based on views, optimizing the usability of data modeling scenarios by flexibly using views and materialized views for hierarchical modeling.
- Optimize the query rewrite for asynchronous materialized views:
  - Support Stale Rewrite, allowing the use of materialized views that have not been refreshed within a specified time for query rewriting, regardless of whether the corresponding base table data has been updated. Users can configure the tolerance for not refreshing through the `mv_rewrite_staleness_second` property when creating materialized views.
  - Support query rewriting for materialized views created based on Hive Catalog external tables in View Delta Join scenarios (requires defining primary and foreign key constraints).
  - Support Join derivation rewriting, Count Distinct, time_slice function, and optimize Union rewriting.
- Optimize the refreshing of asynchronous materialized views:
  - Optimize the refreshing mechanism for materialized views based on Hive Catalog external tables, where StarRocks can perceive data changes at the partition level and only refresh partitions with data changes during automatic refresh.
  - Support synchronous invocation of materialized view refreshing tasks through `REFRESH MATERIALIZED VIEW WITH SYNC MODE`.
- Enhance the usage of asynchronous materialized views:
  - Support enabling or disabling materialized views with the `ALTER MATERIALIZED VIEW {ACTIVE | INACTIVE}` statement. Materialized views that are disabled (in `INACTIVE` state) are not refreshed or used for query rewriting, but can still be queried directly.
  - Support replacing materialized views with the `ALTER MATERIALIZED VIEW SWAP WITH` statement, allowing the atomic replacement of materialized views by creating a new materialized view.
- Optimize synchronous materialized views:
  - Support directly querying synchronous materialized views through SQL Hint `[_SYNC_MV_]`, avoiding a few scenarios that cannot be automatically rewritten.
  - Synchronous materialized views support more expressions, now allowing the use of expressions such as `CASE-WHEN`, `CAST`, mathematical operations, expanding their usage scenarios.

#### Data Lake Analytics

- Optimize Iceberg metadata caching and access to improve query performance.
- Optimize data caching for Data Lake Analytics to further improve performance.

#### Storage, Import, and Query

- Officially support [Spill to Disk](../administration/spill_to_disk.md) functionality, allowing the caching of intermediate results of blocking operators. After enabling this feature, when the query includes aggregation, sorting, or join operators, StarRocks will cache the intermediate results of these operators to the disk to reduce memory usage and avoid query failures due to insufficient memory.
- Support the pruning of Cardinality-preserving Joins. In the modeling of star and snowflake schemas with multiple tables, and some scenarios where queries involve only a few tables, unnecessary tables can be pruned to improve the performance of JOINs.
- When updating primary key model tables with partial updates using the [UPDATE](../sql-reference/sql-statements/data-manipulation/UPDATE.md) statement, support enabling column mode, suitable for scenarios where a few columns are updated but there are a large number of rows, improving update performance by tenfold.
- Optimize statistics collection to reduce the impact on imports and improve collection performance.
- Optimize parallel Merge algorithm, with a maximum performance improvement of 2 times in full sorting scenarios.
- Optimize query logic to no longer rely on database locks.
- Dynamic partitioning now supports a partition granularity of year. [#28386](https://github.com/StarRocks/starrocks/pull/28386)

#### SQL Statements and Functions

- 条件函数 case、coalesce、if、ifnull、nullif 支持 ARRAY、MAP、STRUCT、JSON 类型。
- 以下 Array 函数支持嵌套结构类型 MAP、STRUCT、ARRAY：
  - array_agg
  - array_contains、array_contains_all、array_contains_any
  - array_slice、array_concat
  - array_length、array_append、array_remove、array_position
  - reverse、array_distinct、array_intersect、arrays_overlap
  - array_sortby
- 以下 Array 函数支持 Fast Decimal 类型：
  - array_agg
  - array_append、array_remove、array_position、array_contains
  - array_length
  - array_max、array_min、array_sum、array_avg
  - arrays_overlap、array_difference
  - array_slice、array_distinct、array_sort、reverse、array_intersect、array_concat
  - array_sortby、array_contains_all、array_contains_any

### 问题修复

修复了如下问题：

- 执行 Routine Load 时，无法正常处理重连 Kafka 的请求。[#23477](https://github.com/StarRocks/starrocks/issues/23477)
- SQL 查询中涉及多张表、并且含有 `WHERE` 子句时，如果这些 SQL 查询的语义相同但给定表顺序不同，则有些 SQL 查询不能改写成对相关物化视图的使用。[#22875](https://github.com/StarRocks/starrocks/issues/22875)
- 查询包含 `GROUP BY` 子句时，会返回重复的数据结果。[#19640](https://github.com/StarRocks/starrocks/issues/19640)
- 调用 lead() 或 lag() 函数可能会导致 BE 意外退出。[#22945](https://github.com/StarRocks/starrocks/issues/22945)
- 根据 External Catalog 外表物化视图重写部分分区查询失败。[#19011](https://github.com/StarRocks/starrocks/issues/19011)
- SQL 语句同时包含反斜线 (`\`) 和分号 (`;`) 时解析报错。[#16552](https://github.com/StarRocks/starrocks/issues/16552)
- 物化视图删除后，其基表数据无法清空 (Truncate)。[#19802](https://github.com/StarRocks/starrocks/issues/19802)

### 行为变更

- 存算分离架构下删除建表时的 `storage_cache_ttl` 参数，Cache 写满后按 LRU 算法进行淘汰。
- BE 配置项 `disable_storage_page_cache`、`alter_tablet_worker_count` 和 FE 配置项 `lake_compaction_max_tasks` 由静态参数改为动态参数。
- BE 配置项 `block_cache_checksum_enable` 默认值由 `true` 改为 `false`。
- BE 配置项 `enable_new_load_on_memory_limit_exceeded` 默认值由 `false` 改为 `true`。
- FE 配置项 `max_running_txn_num_per_db` 默认值由 `100` 改为 `1000`。
- FE 配置项 `http_max_header_size` 默认值由 `8192` 改为 `32768`。
- FE 配置项 `tablet_create_timeout_second` 默认值由 `1` 改为 `10`。
- FE 配置项 `max_routine_load_task_num_per_be` 默认值由 `5` 改为 `16`，并且当创建的 Routine Load 任务数量较多时，如果发生报错会给出提示。
- FE 配置项 `quorom_publish_wait_time_ms` 更名为 `quorum_publish_wait_time_ms`，`async_load_task_pool_size` 更名为 `max_broker_load_job_concurrency`。
- CN 配置项 `thrift_port` 更名为 `be_port`。
- 废弃 BE 配置项 `routine_load_thread_pool_size`，单 BE 节点上 Routine Load 线程池大小完全由 FE 配置项 `max_routine_load_task_num_per_be` 控制。
- 废弃 BE 配置项 `txn_commit_rpc_timeout_ms` 和系统变量 `tx_visible_wait_timeout`。
- 废弃 FE 配置项 `max_broker_concurrency`、`load_parallel_instance_num`。
- 废弃 FE 配置项 `max_routine_load_job_num`，通过 `max_routine_load_task_num_per_be` 来动态判断每个 BE 节点上支持的 Routine Load 任务最大数，并且在任务失败时给出建议。
- Routine Load 作业新增两个属性 `task_consume_second` 和 `task_timeout_second`，作用于单个 Routine Load 导入作业内的任务，更加灵活。如果作业中没有设置这两个属性，则采用 FE 配置项 `routine_load_task_consume_second` 和 `routine_load_task_timeout_second` 的配置。
- 默认启用[资源组](../administration/resource_group.md)功能，因此弃用会话变量 `enable_resource_group`。
- 增加如下保留关键字：COMPACTION、TEXT。