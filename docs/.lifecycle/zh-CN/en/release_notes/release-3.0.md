---
displayed_sidebar: "Chinese"
---

# StarRocks版本3.0

## 3.0.8

发布日期：2023年11月17日

## 改进

- 系统数据库`INFORMATION_SCHEMA`中的`COLUMNS`视图可以显示ARRAY、MAP和STRUCT列。[#33431](https://github.com/StarRocks/starrocks/pull/33431)

## Bug修复

修复了以下问题：

- 当执行`show proc '/current_queries';`命令时，同时开始执行查询时，BE可能会崩溃。[#34316](https://github.com/StarRocks/starrocks/pull/34316)
- 在高频率指定排序键的情况下，连续加载数据到具有指定排序键的主键表时，可能会发生压缩失败。[#26486](https://github.com/StarRocks/starrocks/pull/26486)
- 如果在Broker Load作业中指定了过滤条件，在某些情况下数据加载过程中BE可能会崩溃。[#29832](https://github.com/StarRocks/starrocks/pull/29832)
- 执行SHOW GRANTS时会报告未知错误。[#30100](https://github.com/StarRocks/starrocks/pull/30100)
- 如果`cast()`函数中指定的目标数据类型与原始数据类型相同，则对特定数据类型的BE可能会崩溃。[#31465](https://github.com/StarRocks/starrocks/pull/31465)
- 在`information_schema.columns`视图中，BINARY或VARBINARY数据类型的`DATA_TYPE`和`COLUMN_TYPE`显示为`unknown`。[#32678](https://github.com/StarRocks/starrocks/pull/32678)
- 长时间频繁加载数据到启用持久索引的主键表中可能会导致BE崩溃。[#33220](https://github.com/StarRocks/starrocks/pull/33220)
- 启用查询缓存后，查询结果不正确。[#32778](https://github.com/StarRocks/starrocks/pull/32778)
- 集群重新启动后，恢复表中的数据可能与备份前该表中的数据不一致。[#33567](https://github.com/StarRocks/starrocks/pull/33567)
- 如果执行RESTORE并且同时进行Compaction，则可能导致BE崩溃。[#32902](https://github.com/StarRocks/starrocks/pull/32902)

## 3.0.7

发布日期：2023年10月18日

### 改进

- 窗口函数COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD和STDDEV_SAMP现在支持ORDER BY子句和Window子句。[#30786](https://github.com/StarRocks/starrocks/pull/30786)
- 写入数据到主键表的加载作业的发布阶段从异步模式更改为同步模式。因此，加载作业完成后可以立即查询加载的数据。[#27055](https://github.com/StarRocks/starrocks/pull/27055)
- 在DECIMAL类型数据查询中发生十进制溢出时返回错误而非NULL。[#30419](https://github.com/StarRocks/starrocks/pull/30419)
- 执行带有无效注释的SQL命令现在返回与MySQL一致的结果。[#30210](https://github.com/StarRocks/starrocks/pull/30210)
- 对于使用只有一个分区列的RANGE分区的StarRocks表，也可以对包含分区列表达式的SQL谓词用于分区修剪。[#30421](https://github.com/StarRocks/starrocks/pull/30421)

### Bug修复

修复了以下问题：

- 并发创建和删除数据库和表可能在某些情况下导致找不到表，并进一步导致数据加载失败。[#28985](https://github.com/StarRocks/starrocks/pull/28985)
- 在某些情况下使用UDFs可能导致内存泄漏。[#29467](https://github.com/StarRocks/starrocks/pull/29467) [#29465](https://github.com/StarRocks/starrocks/pull/29465)
- 如果ORDER BY子句包含聚合函数，则返回错误"java.lang.IllegalStateException: null"。[#30108](https://github.com/StarRocks/starrocks/pull/30108)
- 如果用户通过其Hive目录对存储在Tencent COS中的数据运行查询，查询结果将不正确。[#30363](https://github.com/StarRocks/starrocks/pull/30363)
- 如果ARRAY<STRUCT>类型数据中的STRUCT的某些子字段丢失，在查询过程中填充丢失的子字段的默认值导致数据长度不正确，这会导致BE崩溃.
- 升级Berkeley DB Java Edition的版本以避免安全漏洞。[#30029](https://github.com/StarRocks/starrocks/pull/30029)
- 如果用户将数据加载到具有指定`NOT NULL`且未指定默认值的主键表中，并发执行截断操作和查询时，在某些情况下会抛出错误"java.lang.NullPointerException"。[#30573](https://github.com/StarRocks/starrocks/pull/30573)
- 如果模式更改执行时间过长，可能会因指定版本的tablet被垃圾收集而失败。[#31376](https://github.com/StarRocks/starrocks/pull/31376)
- 如果用户使用CloudCanal将数据加载到设置为`NOT NULL`但未指定默认值的表列中，会抛出错误"Unsupported dataFormat value is : \N"。[#30799](https://github.com/StarRocks/starrocks/pull/30799)
- 在StarRocks共享数据集群中，`information_schema.COLUMNS`中未记录表键的信息。结果，使用Flink Connector加载数据时无法执行DELETE操作。[#31458](https://github.com/StarRocks/starrocks/pull/31458)
- 在升级期间，如果某些列的类型也升级了（例如，从Decimal类型升级到Decimal v3类型），则具有特定特性的某些表的压缩可能会导致BE崩溃。[#31626](https://github.com/StarRocks/starrocks/pull/31626)
- 如果使用Flink Connector加载数据，如果有高并发加载作业，同时HTTP线程数和扫描线程数都达到上限，则加载作业会意外中断。[#32251](https://github.com/StarRocks/starrocks/pull/32251)
- libcurl调用时会导致BE崩溃。[#31667](https://github.com/StarRocks/starrocks/pull/31667)
- 向主键表添加BITMAP类型列时会发生错误。[#31763](https://github.com/StarRocks/starrocks/pull/31763)

## 3.0.6

发布日期：2023年9月12日

### 行为变更

- 在使用[group_concat](../sql-reference/sql-functions/string-functions/group_concat.md)函数时，必须使用SEPARATOR关键字声明分隔符。

### 新功能

- 聚合函数[group_concat](../sql-reference/sql-functions/string-functions/group_concat.md)支持DISTINCT关键字和ORDER BY子句。[#28778](https://github.com/StarRocks/starrocks/pull/28778)
- 分区中的数据可以随着时间自动冷却。（此功能不支持[list分区](../table_design/list_partitioning.md)。）[#29335](https://github.com/StarRocks/starrocks/pull/29335) [#29393](https://github.com/StarRocks/starrocks/pull/29393)

### 改进

- 支持所有复合谓词和WHERE子句中所有表达式的隐式转换。可以通过使用[会话变量](../reference/System_variable.md)`enable_strict_type`来启用或禁用隐式转换。此会话变量的默认值为`false`。[#21870](https://github.com/StarRocks/starrocks/pull/21870)
- 统一FEs和BEs中将字符串转换为整数的逻辑。[#29969](https://github.com/StarRocks/starrocks/pull/29969)

### Bug修复

- 如果设置`enable_orc_late_materialization`为`true`，则在使用Hive目录查询ORC文件中的STRUCT类型数据时会返回意外结果。[#27971](https://github.com/StarRocks/starrocks/pull/27971)
- 通过Hive Catalog进行数据查询时，如果在WHERE子句中指定了分割列和OR运算符，则查询结果不正确。[#28876](https://github.com/StarRocks/starrocks/pull/28876)
- RESTful API动作`show_data`返回的云原生表值不正确。[#29473](https://github.com/StarRocks/starrocks/pull/29473)
- 如果 [shared-data cluster](../deployment/shared_data/azure.md) 在 Azure Blob 存储中存储数据且创建了表，则在将集群回滚到 3.0 版本后，FE 失败无法启动。[#29433](https://github.com/StarRocks/starrocks/pull/29433)
- 用户在 Iceberg 目录中查询表时即使已授予用户对该表的权限，用户仍无权限。[#29173](https://github.com/StarRocks/starrocks/pull/29173)
- [SHOW FULL COLUMNS](../sql-reference/sql-statements/Administration/SHOW_FULL_COLUMNS.md) 语句返回 [BITMAP](../sql-reference/sql-statements/data-types/BITMAP.md) 或 [HLL](../sql-reference/sql-statements/data-types/HLL.md) 数据类型的列的 `Default` 字段值不正确。[#29510](https://github.com/StarRocks/starrocks/pull/29510)
- 使用 `ADMIN SET FRONTEND CONFIG` 命令修改 FE 动态参数 `max_broker_load_job_concurrency` 无效。
- 在刷新物化视图时同时修改其刷新策略可能导致 FE 启动失败。[#29964](https://github.com/StarRocks/starrocks/pull/29964) [#29720](https://github.com/StarRocks/starrocks/pull/29720)
- 执行 `select count(distinct(int+double)) from table_name` 时返回错误 `unknown error`。[#29691](https://github.com/StarRocks/starrocks/pull/29691)
- 将 Primary Key 表还原后，如果重启 BE，则会出现元数据错误，导致元数据不一致。[#30135](https://github.com/StarRocks/starrocks/pull/30135)

## 3.0.5

发布日期：2023 年 8 月 16 日

### 新特性

- 支持聚合函数 [COVAR_SAMP](../sql-reference/sql-functions/aggregate-functions/covar_samp.md)、[COVAR_POP](../sql-reference/sql-functions/aggregate-functions/covar_pop.md) 和 [CORR](../sql-reference/sql-functions/aggregate-functions/corr.md)。
- 支持以下 [窗口函数](../sql-reference/sql-functions/Window_function.md)：COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD 和 STDDEV_SAMP。

### 改进

- 在错误消息 `xxx too many versions xxx` 中添加了更多提示。[#28397](https://github.com/StarRocks/starrocks/pull/28397)
- 动态分区进一步支持将分区单元设置为 year。[#28386](https://github.com/StarRocks/starrocks/pull/28386)
- 在使用表创建和 [INSERT OVERWRITE 用于覆盖特定分区中的数据](../table_design/expression_partitioning.md#load-data-into-partitions) 时，表达式分区的分区字段是不区分大小写的。[#28309](https://github.com/StarRocks/starrocks/pull/28309)

### Bug 修复

解决了以下问题：

- FE 中不正确的表级扫描统计信息导致表查询和加载的指标不准确。[#27779](https://github.com/StarRocks/starrocks/pull/27779)
- 如果对分区表修改了排序键，则查询结果不稳定。[#27850](https://github.com/StarRocks/starrocks/pull/27850)
- 在数据恢复后 BE 和 FE 之间的 tablet 版本号不一致。[#26518](https://github.com/StarRocks/starrocks/pull/26518/files)
- 如果用户创建 Colocation 表时未指定桶数，则会将该数推断为 0，导致添加新分区时失败。[#27086](https://github.com/StarRocks/starrocks/pull/27086)
- 当 INSERT INTO SELECT 的 SELECT 结果集为空时，SHOW LOAD 返回的加载作业状态为 `CANCELED`。[#26913](https://github.com/StarRocks/starrocks/pull/26913)
- 如果 sub_bitmap 函数的输入值不是 BITMAP 类型，则 BE 可能会崩溃。[#27982](https://github.com/StarRocks/starrocks/pull/27982)
- 更新 AUTO_INCREMENT 列时可能导致 BE 崩溃。[#27199](https://github.com/StarRocks/starrocks/pull/27199)
- 物化视图的外部连接和反连接重写错误。[#28028](https://github.com/StarRocks/starrocks/pull/28028)
- 平均行大小估算不准确导致主键部分更新占用过多内存。[#27485](https://github.com/StarRocks/starrocks/pull/27485)
- 激活非活动物化视图可能导致 FE 崩溃。[#27959](https://github.com/StarRocks/starrocks/pull/27959)
- 无法将查询重写为基于 Hudi 目录的外部表创建的物化视图。[#28023](https://github.com/StarRocks/starrocks/pull/28023)
- 即使删除了 Hive 表并手动更新了元数据缓存，仍可查询 Hive 表的数据。[#28223](https://github.com/StarRocks/starrocks/pull/28223)
- 通过同步调用手动刷新异步物化视图，会导致 `information_schema.task_runs` 表中出现多个 INSERT OVERWRITE 记录。[#28060](https://github.com/StarRocks/starrocks/pull/28060)
- 受阻的 LabelCleaner 线程导致 FE 内存泄漏。[#28311](https://github.com/StarRocks/starrocks/pull/28311)

## 3.0.4

发布日期：2023 年 7 月 18 日

### 新特性

即使查询包含与物化视图不同类型的连接，也可以重写查询。[#25099](https://github.com/StarRocks/starrocks/pull/25099)

### 改进

- 优化了手动刷新异步物化视图。支持使用 REFRESH MATERIALIZED VIEW WITH SYNC MODE 语法同步调用物化视图刷新任务。[#25910](https://github.com/StarRocks/starrocks/pull/25910)
- 如果查询字段不包括在物化视图的输出列中，但包括在物化视图的条件中，则依然可重写查询以从物化视图中受益。[#23028](https://github.com/StarRocks/starrocks/issues/23028)
- 当 SQL 方言（`sql_dialect`）设置为 `trino` 时，表别名不区分大小写。[#26094](https://github.com/StarRocks/starrocks/pull/26094) [#25282](https://github.com/StarRocks/starrocks/pull/25282)
- 在表 `Information_schema.tables_config` 中增加了新字段 `table_id`。可以将 `tables_config` 表与数据库 `Information_schema` 中的 `be_tablets` 表关联，在列 `table_id` 上连接，以查询该 tablet 所属的数据库和表的名称。[#24061](https://github.com/StarRocks/starrocks/pull/24061)

### Bug 修复

解决了以下问题：

- 如果包含 sum 聚合函数的查询被重写为直接从单表物化视图中获取查询结果，则由于类型推断问题，sum() 字段中的值可能不正确。[#25512](https://github.com/StarRocks/starrocks/pull/25512)
- 使用 SHOW PROC 查看 StarRocks 共享数据集群中的 tablet 信息时会出现错误。
- 当要插入的 STRUCT 中 CHAR 数据的长度超过最大长度时，INSERT 操作会挂起。[#25942](https://github.com/StarRocks/starrocks/pull/25942)
- 使用 INSERT INTO SELECT 与 FULL JOIN 时，查询到的某些数据行无法返回。[#26603](https://github.com/StarRocks/starrocks/pull/26603)
- 当使用 ALTER TABLE 语句修改表的属性 `default.storage_medium` 时，会出现错误 `ERROR xxx: Unknown table property xxx`。[#25870](https://github.com/StarRocks/starrocks/issues/25870)
- 利用 Broker Load 加载空文件时会出现错误。[#26212](https://github.com/StarRocks/starrocks/pull/26212)
- 有时候 BE 的下线会挂起。[#26509](https://github.com/StarRocks/starrocks/pull/26509)

## 3.0.3

发布日期：2023 年 6 月 28 日

### 改进

- StarRocks 的外部表元数据同步已更改为在数据加载期间发生。[#24739](https://github.com/StarRocks/starrocks/pull/24739)
- 当用户运行INSERT OVERWRITE时，他们可以指定分区，而这些分区是自动创建的表的，了解更多信息，请查看[自动分区](https://docs.starrocks.io/zh-cn/3.0/table_design/dynamic_partitioning)。[#25005](https://github.com/StarRocks/starrocks/pull/25005)
- 当分区被添加到非分区表时，优化了报告的错误消息。[#25266](https://github.com/StarRocks/starrocks/pull/25266)

### Bug 修复

修复了以下问题:

- 当Parquet文件包含复杂数据类型时，最小/最大筛选器会获取错误的Parquet字段。[#23976](https://github.com/StarRocks/starrocks/pull/23976)
- 即使相关数据库或表已被删除，加载任务仍在排队。[#24801](https://github.com/StarRocks/starrocks/pull/24801)
- FE重新启动时，BE可能会崩溃的可能性较低。[#25037](https://github.com/StarRocks/starrocks/pull/25037)
- 当变量`enable_profile`设置为`true`时，加载和查询作业偶尔会冻结。[#25060](https://github.com/StarRocks/starrocks/pull/25060)
- 在具有少于三个活动BE的集群上执行INSERT OVERWRITE时，会显示不准确的错误消息。[#25314](https://github.com/StarRocks/starrocks/pull/25314)

## 3.0.2

发布日期：2023年6月13日

### 改进

- 在异步物化视图重写查询后，UNION查询中的谓词可以被推动下去。[#23312](https://github.com/StarRocks/starrocks/pull/23312)
- 优化了表的[自动Tablet分布策略](../table_design/Data_distribution.md#determine-the-number-of-buckets)。[#24543](https://github.com/StarRocks/starrocks/pull/24543)
- 移除了NetworkTime对系统时钟的依赖，修复由服务器之间的系统时钟不一致引起的不正确NetworkTime。[#24858](https://github.com/StarRocks/starrocks/pull/24858)

### Bug 修复

修复了以下问题:

- 如果数据加载与模式更改同时发生，有时模式更改可能会挂起。[#23456](https://github.com/StarRocks/starrocks/pull/23456)
- 当会话变量`pipeline_profile_level`设置为`0`时，查询会遇到错误。[#23873](https://github.com/StarRocks/starrocks/pull/23873)
- 当`cloud_native_storage_type`设置为`S3`时，在创建表时会遇到错误。
- 即使没有使用密码，LDAP验证也会成功。[#24862](https://github.com/StarRocks/starrocks/pull/24862)
- 如果涉及的表在加载作业中不存在，CANCEL LOAD将失败。[#24922](https://github.com/StarRocks/starrocks/pull/24922)

### 升级注意事项

如果您的系统有名为`starrocks`的数据库，在升级之前请使用ALTER DATABASE RENAME更改为另一个名称。这是因为`starrocks`是存储权限信息的默认系统数据库的名称。

## 3.0.1

发布日期：2023年6月1日

### 新功能

- [预览] 支持将大型运算符的中间计算结果溢出到磁盘以减少大型运算符的内存消耗。有关更多信息，请参阅[溢出到磁盘](../administration/spill_to_disk.md)。
- [Routine Load](../loading/RoutineLoad.md#load-avro-format-data)支持加载Avro数据。
- 支持使用[Microsoft Azure存储](../integrations/authenticate_to_azure_storage.md)（包括Azure Blob存储和Azure数据湖存储）。

### 改进

- 共享数据集群支持使用StarRocks外部表与另一个StarRocks集群同步数据。
- 向[信息模式](../reference/information_schema/load_tracking_logs.md)中添加了`load_tracking_logs`以记录最近的加载错误。
- 在CREATE TABLE语句中忽略特殊字符。[#23885](https://github.com/StarRocks/starrocks/pull/23885)

### Bug 修复

修复了以下问题:

- 对于主键表，SHOW CREATE TABLE返回的信息不正确。[#24237](https://github.com/StarRocks/starrocks/issues/24237)
- BE在执行Routine Load作业时可能会崩溃。[#20677](https://github.com/StarRocks/starrocks/issues/20677)
- 如果在创建分区表时指定了不受支持的属性，将发生空指针异常（NPE）。[#21374](https://github.com/StarRocks/starrocks/issues/21374)
- SHOW TABLE STATUS返回的信息不完整。[#24279](https://github.com/StarRocks/starrocks/issues/24279)

### 升级注意事项

如果您的系统有名为`starrocks`的数据库，在升级之前请使用ALTER DATABASE RENAME更改为另一个名称。这是因为`starrocks`是存储权限信息的默认系统数据库的名称。

## 3.0.0

发布日期：2023年4月28日

### 新功能

#### 系统架构

- **解耦存储和计算。** StarRocks现在支持将数据持久化到兼容S3的对象存储中，增强资源隔离，降低存储成本，并使计算资源更具可扩展性。本地磁盘用作热数据缓存，以提升查询性能。新的共享数据架构的查询性能在本地磁盘缓存命中时，与经典架构（共享无）可比。有关更多信息，请参阅[部署和使用共享数据StarRocks](../deployment/shared_data/s3.md)。

#### 存储引擎和数据摄入

- 支持[AUTO_INCREMENT](../sql-reference/sql-statements/auto_increment.md)属性以提供全局唯一ID，简化数据管理。
- 支持[自动分区和分区表达式](../table_design/dynamic_partitioning.md)，使分区创建更易于使用和更灵活。
- 主键表支持更完整的[UPDATE](../sql-reference/sql-statements/data-manipulation/UPDATE.md)和[DELETE](../sql-reference/sql-statements/data-manipulation/DELETE.md#primary-key-tables)语法，包括使用CTE和引用多个表。
- 为Broker Load和INSERT INTO作业添加了Load Profile。您可以通过查询加载配置文件来查看加载作业的详细信息。使用方法与[分析查询配置文件](../administration/query_profile.md)相同。

#### 数据湖分析

- [预览] 支持Presto/Trino兼容方言。Presto/Trino的SQL可以自动重写为StarRocks的SQL模式。有关更多信息，请参阅[系统变量](../reference/System_variable.md) `sql_dialect`。
- [预览] 支持[JDBC目录](../data_source/catalog/jdbc_catalog.md)。
- 支持使用[SET CATALOG](../sql-reference/sql-statements/data-definition/SET_CATALOG.md)在当前会话中手动切换目录。

#### 权限和安全

- 提供了一个具有完整RBAC功能的新权限系统，支持角色继承和默认角色。有关更多信息，请参阅[权限概述](../administration/privilege_overview.md)。
- 提供了更多的权限管理对象和更精细的权限。有关更多信息，请参阅[StarRocks支持的权限](../administration/privilege_item.md)。

#### 查询引擎

<!-- - [预览] 支持大型查询的**溢出**操作符，可使用磁盘空间以确保查询运行时内存不足时的稳定运行。-->
- 让更多的连接表的查询受益于[查询缓存](../using_starrocks/query_cache.md)。例如，查询缓存现在支持Broadcast Join和Bucket Shuffle Join。
- 支持全局UDF。[Global UDFs](../sql-reference/sql-functions/JAVA_UDF.md)。
- 动态自适应并行性：StarRocks可以自动调整查询并发的`pipeline_dop`参数。

#### SQL参考

- 添加了以下与权限相关的SQL语句：[SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md)、[SET ROLE](../sql-reference/sql-statements/account-management/SET_ROLE.md)、[SHOW ROLES](../sql-reference/sql-statements/account-management/SHOW_ROLES.md)和[SHOW USERS](../sql-reference/sql-statements/account-management/SHOW_USERS.md)。
- 添加了以下半结构化数据分析函数：[map_apply](../sql-reference/sql-functions/map-functions/map_apply.md)、[map_from_arrays](../sql-reference/sql-functions/map-functions/map_from_arrays.md)。
- [array_agg](../sql-reference/sql-functions/array-functions/array_agg.md)支持ORDER BY。
- 窗口函数[lead](../sql-reference/sql-functions/Window_function.md#lead)和[lag](../sql-reference/sql-functions/Window_function.md#lag)支持IGNORE NULLS。
- 添加字符串函数[replace](../sql-reference/sql-functions/string-functions/replace.md)、[hex_decode_binary](../sql-reference/sql-functions/string-functions/hex_decode_binary.md)和[hex_decode_string()](../sql-reference/sql-functions/string-functions/hex_decode_string.md)。
- 添加了加密函数 [base64_decode_binary](../sql-reference/sql-functions/crytographic-functions/base64_decode_binary.md) 和 [base64_decode_string](../sql-reference/sql-functions/crytographic-functions/base64_decode_string.md)。
- 添加了数学函数 [sinh](../sql-reference/sql-functions/math-functions/sinh.md), [cosh](../sql-reference/sql-functions/math-functions/cosh.md) 和 [tanh](../sql-reference/sql-functions/math-functions/tanh.md)。
- 添加了实用函数 [current_role](../sql-reference/sql-functions/utility-functions/current_role.md)。

### 改进

#### 部署

- 更新了Docker镜像和相关的[使用Docker部署文档](../quick_start/deploy_with_docker.md)，用于版本3.0。 [#20623](https://github.com/StarRocks/starrocks/pull/20623) [#21021](https://github.com/StarRocks/starrocks/pull/21021)

#### 存储引擎与数据摄入

- 支持更多用于数据摄入的CSV参数，包括 SKIP_HEADER，TRIM_SPACE，ENCLOSE 和 ESCAPE。参见 [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md), [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 和 [ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。
- 在[Primary Key tables](../table_design/table_types/primary_key_table.md)中，主键和排序键已解耦。在创建表时，可以单独指定 `ORDER BY` 中的排序键。
- 针对数据摄入进入主键表时的内存使用情况进行了优化，包括大量数据摄入、部分更新和持久主键索引等场景。
- 支持创建异步的INSERT任务。更多信息请参阅 [INSERT](../loading/InsertInto.md#load-data-asynchronously-using-insert) 和 [SUBMIT TASK](../sql-reference/sql-statements/data-manipulation/SUBMIT_TASK.md)。 [#20609](https://github.com/StarRocks/starrocks/issues/20609)

#### Materialized view

- 优化了[materialized views](../using_starrocks/Materialized_view.md)的重写能力，包括：
  - 支持对视图Delta Join、Outer Join和Cross Join的重写。
  - 优化了联合分区的SQL重写。
- 改进了物化视图的构建能力：支持CTE，select * 和 Union。
- 优化了[SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)返回的信息。
- 支持批量添加MV分区，提高了物化视图构建时的分区添加效率。 [#21167](https://github.com/StarRocks/starrocks/pull/21167)

#### 查询引擎

- 管道引擎支持所有操作符。非管道代码将在后续版本中删除。
- 改进了[Big Query Positioning](../administration/monitor_manage_big_queries.md)，并添加了大查询日志。[SHOW PROCESSLIST](../sql-reference/sql-statements/Administration/SHOW_PROCESSLIST.md) 支持查看CPU和内存信息。
- 优化了Outer Join Reorder。
- 在SQL解析阶段优化了错误消息，提供更准确的错误定位和更清晰的错误消息。

#### 数据湖分析

- 优化了元数据统计收集。
- 支持使用 [SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md) 查看由外部目录管理并存储在Apache Hive™、Apache Iceberg、Apache Hudi或Delta Lake中的表的创建语句。

### Bug修复

- StarRocks源文件的许可标头中的一些URL无法访问。 [#2224](https://github.com/StarRocks/starrocks/issues/2224)
- 在SELECT查询期间返回了未知错误。[#19731](https://github.com/StarRocks/starrocks/issues/19731)
- 支持SHOW/SET CHARACTER。[#17480](https://github.com/StarRocks/starrocks/issues/17480)
- 当加载的数据超过StarRocks支持的字段长度时，返回的错误消息不正确。[#14](https://github.com/StarRocks/DataX/issues/14)
- 支持 `show full fields from 'table'`。[#17233](https://github.com/StarRocks/starrocks/issues/17233)
- 分区修剪导致MV重写失败。[#14641](https://github.com/StarRocks/starrocks/issues/14641)
- 当CREATE MATERIALIZED VIEW语句包含`count(distinct)`并且`count(distinct)`应用于DISTRIBUTED BY列时，MV重写失败。[#16558](https://github.com/StarRocks/starrocks/issues/16558)
- 当VARCHAR列被用作物化视图的分区列时，FE无法启动。[#19366](https://github.com/StarRocks/starrocks/issues/19366)
- 窗口函数[LEAD](../sql-reference/sql-functions/Window_function.md#lead)和[LAG](../sql-reference/sql-functions/Window_function.md#lag)错误处理IGNORE NULLS。[#21001](https://github.com/StarRocks/starrocks/pull/21001)
- 添加临时分区与自动分区创建存在冲突。[#21222](https://github.com/StarRocks/starrocks/issues/21222)

### 行为变更

- 新的基于角色的访问控制（RBAC）系统支持先前的特权和角色。但是，相关语句的语法，如 [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) 和 [REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md) 发生了变化。
- 将SHOW MATERIALIZED VIEW重命名为 [SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)。
- 添加了以下[保留关键字](../sql-reference/sql-statements/keywords.md): AUTO_INCREMENT, CURRENT_ROLE, DEFERRED, ENCLOSE, ESCAPE, IMMEDIATE, PRIVILEGES, SKIP_HEADER, TRIM_SPACE, VARBINARY。

### 升级说明

您可以从v2.5升级到v3.0，或者从v3.0降级到v2.5。

> 理论上，也支持从早于v2.5的版本升级。为了确保系统的可用性，我们建议您首先将您的集群升级到v2.5，然后再升级到v3.0。

在从v3.0降级到v2.5时，请注意以下几点。

#### BDBJE

StarRocks在v3.0中升级了BDB库。但是，无法回滚BDBJE。降级后，您必须使用v3.0的BDB库。执行以下步骤：

1. 替换FE软件包为v2.5软件包后，将v3.0的`fe/lib/starrocks-bdb-je-18.3.13.jar`复制到v2.5的`fe/lib`目录中。

2. 删除`fe/lib/je-7.*.jar`。

#### 特权系统

升级到v3.0后，默认使用新的RBAC特权系统。您只能降级到v2.5。

降级后，运行 [ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md) 来创建新映像，并等待新映像同步到所有关注FE。如果您不运行此命令，则某些降级操作可能会失败。此命令从2.5.3及更高版本开始支持。

有关v2.5和v3.0特权系统之间差异的详细信息，请参阅[StarRocks支持的特权](../administration/privilege_item.md)中的“升级说明”。