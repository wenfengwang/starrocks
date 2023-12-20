---
displayed_sidebar: English
---

# 星石版本 3.0

## 3.0.8

发布日期：2023年11月17日

## 改进内容

- 系统数据库`INFORMATION_SCHEMA`中的`COLUMNS`视图现在可以展示ARRAY、MAP和STRUCT列。[#33431](https://github.com/StarRocks/starrocks/pull/33431)

## 问题修复

修复了以下问题：

- 当执行 `show proc '/current_queries';` 命令时，如果同时有查询开始执行，BE可能会崩溃。[#34316](https://github.com/StarRocks/starrocks/pull/34316)
- 当数据以高频率连续加载到设定了排序键的主键表中时，可能会发生压缩失败。[#26486](https://github.com/StarRocks/starrocks/pull/26486)
- 如果在Broker Load任务中指定了过滤条件，BEs可能会在数据加载过程中在某些情况下崩溃。[#29832](https://github.com/StarRocks/starrocks/pull/29832)
- 执行SHOW GRANTS时报告未知错误。[#30100](https://github.com/StarRocks/starrocks/pull/30100)
- 如果在`cast()`函数中指定的目标数据类型与原始数据类型相同，BE可能会因特定数据类型而崩溃。[#31465](https://github.com/StarRocks/starrocks/pull/31465)
- 在`information_schema.columns`视图中，BINARY或VARBINARY数据类型的`DATA_TYPE`和`COLUMN_TYPE`显示为`未知`。[#32678](https://github.com/StarRocks/starrocks/pull/32678)
- 长时间、频繁地向启用了持久性索引的主键表加载数据，可能会导致BE崩溃。[#33220](https://github.com/StarRocks/starrocks/pull/33220)
- 当查询缓存启用时，查询结果可能不正确。[#32778](https://github.com/StarRocks/starrocks/pull/32778)
- 在集群重启后，恢复的表中的数据可能与备份前的数据不一致。[#33567](https://github.com/StarRocks/starrocks/pull/33567)
- 如果在执行**RESTORE**操作的同时进行**压缩**操作，可能会导致BE崩溃。[#32902](https://github.com/StarRocks/starrocks/pull/32902)

## 3.0.7

发布日期：2023年10月18日

### 改进内容

- 窗口函数**COVAR_SAMP**、**COVAR_POP**、**CORR**、**VARIANCE**、**VAR_SAMP**、**STD**和**STDDEV_SAMP**现在支持**ORDER BY**子句和**窗口**子句。[#30786](https://github.com/StarRocks/starrocks/pull/30786)
- 将数据写入主键表的加载作业的发布阶段由异步模式改为同步模式。因此，加载作业完成后可以立即查询到加载的数据。[#27055](https://github.com/StarRocks/starrocks/pull/27055)
- 如果在查询**DECIMAL**类型数据时发生小数溢出，则返回错误而非**NULL**。[#30419](https://github.com/StarRocks/starrocks/pull/30419)
- 执行带有无效注释的SQL命令现在返回与MySQL一致的结果。[#30210](https://github.com/StarRocks/starrocks/pull/30210)
- 对于仅使用一个分区列或表达式分区的**RANGE**分区**StarRocks**表，SQL谓词中包含的分区列表达式也可以用于分区修剪。[#30421](https://github.com/StarRocks/starrocks/pull/30421)

### 问题修复

修复了以下问题：

- 在某些情况下，同时创建和删除数据库和表可能导致找不到表，从而导致无法将数据加载到该表。[#28985](https://github.com/StarRocks/starrocks/pull/28985)
- 在某些情况下，使用UDF可能会导致内存泄露。[#29467](https://github.com/StarRocks/starrocks/pull/29467) [#29465](https://github.com/StarRocks/starrocks/pull/29465)
- 如果ORDER BY子句包含聚合函数，则会返回错误 ["java.lang.IllegalStateException: null"](https://github.com/StarRocks/starrocks/pull/30108)。#30108
- 如果用户通过他们的Hive目录对存储在[Tencent COS](https://github.com/StarRocks/starrocks/pull/30363)中的数据运行查询，查询结果将是不正确的。#30363
- 如果ARRAY<STRUCT>类型数据中STRUCT的一些子字段缺失，在查询时填充缺失子字段的默认值时，数据长度不正确，可能导致BE崩溃。
- Berkeley DB Java Edition的版本已升级，以避免安全漏洞。[#30029](https://github.com/StarRocks/starrocks/pull/30029)
- 如果用户在对主键表执行截断操作和查询操作的同时加载数据，某些情况下可能会抛出错误"java.lang.NullPointerException"。[#30573](https://github.com/StarRocks/starrocks/pull/30573)
- 如果Schema Change执行时间过长，可能因为指定版本的tablet已被垃圾回收而失败。[#31376](https://github.com/StarRocks/starrocks/pull/31376)
- 如果用户使用CloudCanal将数据加载到设置为`NOT NULL`但未指定默认值的表列中，则可能会抛出错误"Unsupported dataFormat value is : \N"。[#30799](https://github.com/StarRocks/starrocks/pull/30799)
- 在StarRocks共享数据集群中，`information_schema.COLUMNS`中未记录表键信息，导致使用Flink Connector加载数据时无法执行DELETE操作。[#31458](https://github.com/StarRocks/starrocks/pull/31458)
- 在升级过程中，如果某些列的类型也升级（例如，从Decimal类型升级到Decimal v3类型），在对某些具有特定特征的表进行压缩时，可能会导致BEs崩溃。[#31626](https://github.com/StarRocks/starrocks/pull/31626)
- 当使用Flink Connector加载数据时，如果存在大量并发加载作业，并且HTTP线程数和Scan线程数都达到上限，加载作业可能会意外暂停。[#32251](https://github.com/StarRocks/starrocks/pull/32251)
- 当调用**libcurl**时，BE可能会崩溃。[#31667](https://github.com/StarRocks/starrocks/pull/31667)
- 在主键表中添加**BITMAP**类型列时可能会出错。[#31763](https://github.com/StarRocks/starrocks/pull/31763)

## 3.0.6

发布日期：2023年9月12日

### 行为变更

- 当使用[group_concat](../sql-reference/sql-functions/string-functions/group_concat.md)函数时，你必须使用SEPARATOR关键字来声明分隔符。

### 新特性

- 聚合函数 [group_concat](../sql-reference/sql-functions/string-functions/group_concat.md) 支持**DISTINCT**关键字和**ORDER BY**子句。[#28778](https://github.com/StarRocks/starrocks/pull/28778)
- 数据可以随着时间自动冷却在分区中。（此功能不支持[list partitioning](../table_design/list_partitioning.md)。）[#29335](https://github.com/StarRocks/starrocks/pull/29335) [#29393](https://github.com/StarRocks/starrocks/pull/29393)

### 改进内容

- 支持所有复合谓词和WHERE子句中所有表达式的隐式转换。您可以使用[会话变量](../reference/System_variable.md) `enable_strict_type`来启用或禁用隐式转换。该会话变量的默认值为`false`。[#21870](https://github.com/StarRocks/starrocks/pull/21870)
- 统一了FE和BE之间将字符串转换为整数的逻辑。[#29969](https://github.com/StarRocks/starrocks/pull/29969)

### 问题修复

- 如果将`enable_orc_late_materialization`设置为`true`，在使用Hive目录查询ORC文件中的STRUCT类型数据时，可能会返回意外结果。[#27971](https://github.com/StarRocks/starrocks/pull/27971)
- 通过Hive Catalog进行数据查询时，如果WHERE子句中同时指定了分区列和OR运算符，则查询结果可能不正确。[#28876](https://github.com/StarRocks/starrocks/pull/28876)
- 云原生表的RESTful API动作`show_data`返回的值不正确。[#29473](https://github.com/StarRocks/starrocks/pull/29473)
- 如果[共享数据集群](../deployment/shared_data/azure.md)将数据存储在Azure Blob Storage中并且已创建表，则在集群回滚到3.0版本后，FE可能无法启动。[#29433](https://github.com/StarRocks/starrocks/pull/29433)
- 即使用户已被授权访问该表，但在查询Iceberg目录中的表时，用户可能没有权限。[#29173](https://github.com/StarRocks/starrocks/pull/29173)
- `SHOW FULL COLUMNS`语句为[BITMAP](../sql-reference/sql-statements/data-types/BITMAP.md)或[HLL](../sql-reference/sql-statements/data-types/HLL.md)数据类型的列返回的`Default`字段值不正确。[#29510](https://github.com/StarRocks/starrocks/pull/29510)
- 使用ADMIN SET FRONTEND CONFIG命令修改FE动态参数max_broker_load_job_concurrency无效。
- 当物化视图正在刷新时，如果同时修改其刷新策略，FE可能无法启动。[#29964](https://github.com/StarRocks/starrocks/pull/29964) [#29720](https://github.com/StarRocks/starrocks/pull/29720)
- 执行`select count(distinct(int+double)) from table_name`时返回`未知错误`。[#29691](https://github.com/StarRocks/starrocks/pull/29691)
- 主键表恢复后，如果BE重启，则可能出现元数据错误，导致元数据不一致。[#30135](https://github.com/StarRocks/starrocks/pull/30135)

## 3.0.5

发布日期：2023年8月16日

### 新特性

- 支持聚合函数[COVAR_SAMP](../sql-reference/sql-functions/aggregate-functions/covar_samp.md)、[COVAR_POP](../sql-reference/sql-functions/aggregate-functions/covar_pop.md)和[CORR](../sql-reference/sql-functions/aggregate-functions/corr.md)。
- 支持以下窗口函数：COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD和STDDEV_SAMP。[window functions](../sql-reference/sql-functions/Window_function.md)

### 改进内容

- 在错误消息 `xxx版本过多xxx` 中添加了更多提示。[#28397](https://github.com/StarRocks/starrocks/pull/28397)
- 动态分区进一步支持年为分区单位。[#28386](https://github.com/StarRocks/starrocks/pull/28386)
- 在使用表达式分区创建表并使用[INSERT OVERWRITE覆盖特定分区中的数据](../table_design/expression_partitioning.md#load-data-into-partitions)时，分区字段不区分大小写。[#28309](https://github.com/StarRocks/starrocks/pull/28309)

### 问题修复

修复了以下问题：

- FE中不正确的表级扫描统计可能导致表查询和加载的指标不准确。[#27779](https://github.com/StarRocks/starrocks/pull/27779)
- 如果修改了分区表的排序键，查询结果可能不稳定。[#27850](https://github.com/StarRocks/starrocks/pull/27850)
- 数据恢复后，平板电脑版本号在BE和FE之间不一致。[#26518](https://github.com/StarRocks/starrocks/pull/26518/files)
- 如果用户在创建Colocation表时没有指定Bucket数量，则该数量会被推断为0，导致无法添加新分区。[#27086](https://github.com/StarRocks/starrocks/pull/27086)
- 当SELECT INTO INSERT的结果集为空时，通过SHOW LOAD返回的加载作业状态为 `CANCELED`。[#26913](https://github.com/StarRocks/starrocks/pull/26913)
- BEs可能在sub_bitmap函数的输入值不是BITMAP类型时崩溃。[#27982](https://github.com/StarRocks/starrocks/pull/27982)
- BEs可能在更新AUTO_INCREMENT列时崩溃。[#27199](https://github.com/StarRocks/starrocks/pull/27199)
- 物化视图的Outer join和Anti join重写错误。[#28028](https://github.com/StarRocks/starrocks/pull/28028)
- 对平均行大小的不准确估计可能导致主键部分更新占用过多内存。[#27485](https://github.com/StarRocks/starrocks/pull/27485)
- 激活不活跃的物化视图可能会导致FE崩溃。[#27959](https://github.com/StarRocks/starrocks/pull/27959)
- 查询无法重写为基于[Hudi目录](https://github.com/StarRocks/starrocks/pull/28023)中外部表创建的物化视图。#28023
- 即使在删除Hive表并手动更新元数据缓存后，仍可以查询到该表的数据。[#28223](https://github.com/StarRocks/starrocks/pull/28223)
- 通过同步调用手动刷新异步物化视图会导致多条`INSERT OVERWRITE`记录在[information_schema.task_runs](https://github.com/StarRocks/starrocks/pull/28060)表中。#28060
- FE内存泄漏是由被阻塞的LabelCleaner线程引起的。[#28311](https://github.com/StarRocks/starrocks/pull/28311)

## 3.0.4

发布日期：2023年7月18日

### 新特性

即使查询包含与物化视图不同类型的连接，也可以重写查询。[#25099](https://github.com/StarRocks/starrocks/pull/25099)

### 改进内容

- 优化了异步物化视图的手动刷新。支持使用[REFRESH MATERIALIZED VIEW WITH SYNC MODE](https://github.com/StarRocks/starrocks/pull/25910)语法同步调用物化视图刷新任务。#25910
- 如果查询的字段不在物化视图的输出列中，但包含在物化视图的谓词中，仍可以重写查询以从物化视图中受益。[#23028](https://github.com/StarRocks/starrocks/issues/23028)
- [当SQL方言(`sql_dialect`)设置为`trino`](../reference/System_variable.md)，表别名不区分大小写。[#26094](https://github.com/StarRocks/starrocks/pull/26094) [#25282](https://github.com/StarRocks/starrocks/pull/25282)
- 在`Information_schema.tables_config`表中添加了新字段`table_id`。您可以将`tables_config`表与`be_tablets`表在`Information_schema`数据库中的`table_id`列上进行连接，查询平板电脑所属的数据库和表的名称。[#24061](https://github.com/StarRocks/starrocks/pull/24061)

### 问题修复

修复了以下问题：

- 如果将包含`sum`聚合函数的查询重写为直接从单表物化视图获取查询结果，由于类型推断问题，`sum()`字段的值可能不正确。[#25512](https://github.com/StarRocks/starrocks/pull/25512)
- 使用SHOW PROC查看StarRocks共享数据集群中的平板电脑信息时出现错误。
- 当要插入的STRUCT中的CHAR数据长度超出最大长度时，INSERT操作会挂起。[#25942](https://github.com/StarRocks/starrocks/pull/25942)
- 对于INSERT INTO SELECT with FULL JOIN，查询到的某些数据行无法返回。[#26603](https://github.com/StarRocks/starrocks/pull/26603)
- An error `ERROR xxx: Unknown table property xxx` occurs when the ALTER TABLE statement is used to modify the table's property `default.storage_medium`. [#25870](https://github.com/StarRocks/starrocks/issues/25870)
- 使用Broker Load加载空文件时出现错误。[#26212](https://github.com/StarRocks/starrocks/pull/26212)
- 停用BE有时会挂起。[#26509](https://github.com/StarRocks/starrocks/pull/26509)

## 3.0.3

发布日期：2023年6月28日

### 改进内容

- StarRocks外部表的元数据同步已更改为在数据加载期间进行。[#24739](https://github.com/StarRocks/starrocks/pull/24739)
- 用户在自动创建分区的表上执行**INSERT OVERWRITE**时，可以指定分区。有关详细信息，请参见[自动分区功能](https://docs.starrocks.io/en-us/3.0/table_design/dynamic_partitioning)。[#25005](https://github.com/StarRocks/starrocks/pull/25005)
- 优化了向非分区表添加分区时的错误报告问题。 [#25266](https://github.com/StarRocks/starrocks/pull/25266)

### Bug 修复

修复了以下问题：

- 当 Parquet 文件包含复杂数据类型时，最小/最大过滤器可能会获取错误的 Parquet 字段。 [#23976](https://github.com/StarRocks/starrocks/pull/23976)
- 即使相关数据库或表已被删除，加载任务仍然会排队。 [#24801](https://github.com/StarRocks/starrocks/pull/24801)
- FE 重启可能会导致 BEs 崩溃的概率很低。[#25037](https://github.com/StarRocks/starrocks/pull/25037)
- 当变量 `enable_profile` 设置为 `true` 时，加载和查询作业偶尔会冻结。 [#25060](https://github.com/StarRocks/starrocks/pull/25060)
- 在活动 BE 少于三个的集群上执行 INSERT OVERWRITE时，会出现不准确的错误消息。 [#25314](https://github.com/StarRocks/starrocks/pull/25314)

## 3.0.2

发布日期：2023年6月13日

### 改进

- 在异步物化视图重写查询后，UNION查询中的谓词可以下推。[#23312](https://github.com/StarRocks/starrocks/pull/23312)
- 优化了表的[自动分配策略](../table_design/Data_distribution.md#determine-the-number-of-buckets)。[#24543](https://github.com/StarRocks/starrocks/pull/24543)
- 移除了 NetworkTime 对系统时钟的依赖，修复了由服务器系统时钟不一致导致的 NetworkTime 错误问题。 [#24858](https://github.com/StarRocks/starrocks/pull/24858)

### Bug 修复

修复了以下问题：

- 如果数据加载与架构更改同时进行，架构更改有时可能会挂起。 [#23456](https://github.com/StarRocks/starrocks/pull/23456)
- 当会话变量 `pipeline_profile_level` 设置为 `0` 时，查询可能会遇到错误。 [#23873](https://github.com/StarRocks/starrocks/pull/23873)
- 当 cloud_native_storage_type 设置为 S3 时，CREATE TABLE 可能会出错。
- 即使没有使用密码，LDAP 身份验证也可能成功。 [#24862](https://github.com/StarRocks/starrocks/pull/24862)
- 如果涉及的表不存在，CANCEL LOAD 可能会失败。 [#24922](https://github.com/StarRocks/starrocks/pull/24922)

### 升级注意事项

如果您的系统中有一个名为 starrocks 的数据库，请在升级前使用 ALTER DATABASE RENAME 命令将其重命名。这是因为 starrocks 是存储权限信息的默认系统数据库的名称。

## 3.0.1

发布日期：2023年6月1日

### 新功能

- [预览] 支持将大型运算符的中间计算结果**溢写到磁盘**，以降低大型运算符的内存消耗。有关详细信息，请参阅[溢写到磁盘](../administration/spill_to_disk.md)。
- [Routine Load](../loading/RoutineLoad.md#load-avro-format-data)现支持加载Avro数据。
- 支持[Microsoft Azure Storage](../integrations/authenticate_to_azure_storage.md)（包括 Azure Blob Storage 和 Azure Data Lake Storage）。

### 改进

- 共享数据集群支持使用 StarRocks 外部表与另一个 StarRocks 集群同步数据。
- 在 [Information Schema](../reference/information_schema/load_tracking_logs.md) 中添加了 `load_tracking_logs`，以记录最近的加载错误。
- 在 CREATE TABLE 语句中忽略了特殊字符。 [#23885](https://github.com/StarRocks/starrocks/pull/23885)

### Bug 修复

修复了以下问题：

- Information returned by SHOW CREATE TABLE is incorrect for Primary Key tables. [#24237](https://github.com/StarRocks/starrocks/issues/24237)
- BE 在 Routine Load 作业期间可能会崩溃。 [#20677](https://github.com/StarRocks/starrocks/issues/20677)
- 如果在创建分区表时指定了不支持的属性，可能会出现空指针异常 (NPE)。 [#21374](https://github.com/StarRocks/starrocks/issues/21374)
- Information returned by SHOW TABLE STATUS is incomplete. [#24279](https://github.com/StarRocks/starrocks/issues/24279)

### 升级注意事项

如果您的系统中有一个名为 starrocks 的数据库，请在升级前使用 ALTER DATABASE RENAME 命令将其重命名。这是因为 starrocks 是存储权限信息的默认系统数据库的名称。

## 3.0.0

发布日期：2023年4月28日

### 新功能

#### 系统架构

- **解耦存储和计算**。StarRocks现在支持将数据持久化到兼容S3的对象存储中，增强了资源隔离、降低了存储成本，并使计算资源更加可扩展。使用本地磁盘作为热数据缓存，以提高查询性能。新的共享数据架构的查询性能，在本地磁盘缓存命中时，与经典架构（无共享）相当。有关更多信息，请参阅[部署和使用共享数据StarRocks](../deployment/shared_data/s3.md)。

#### 存储引擎和数据摄入

- 支持 [AUTO_INCREMENT](../sql-reference/sql-statements/auto_increment.md) 属性，以提供全局唯一 ID，简化了数据管理。
- [自动分区和分区表达式](../table_design/dynamic_partitioning.md)被支持，使得分区创建更易于使用和更灵活。
- 主键表支持更完整的 [UPDATE](../sql-reference/sql-statements/data-manipulation/UPDATE.md) 和 [DELETE](../sql-reference/sql-statements/data-manipulation/DELETE.md#primary-key-tables) 语法，包括使用 CTE 和引用多个表。
- 为 Broker Load 和 INSERT INTO 作业添加了 Load Profile。您可以通过查询 Load Profile 来查看加载作业的详细信息。其使用方式与[分析查询 Profile](../administration/query_profile.md)相同。

#### 数据湖分析

- [预览] 支持 Presto/Trino 兼容的 SQL 方言。Presto/Trino 的 SQL 可以自动重写为 StarRocks 的 SQL 模式。有关详细信息，请参阅[系统变量](../reference/System_variable.md) `sql_dialect`。
- [预览] 支持[JDBC catalogs](../data_source/catalog/jdbc_catalog.md)。
- 支持使用[SET CATALOG](../sql-reference/sql-statements/data-definition/SET_CATALOG.md)在当前会话中手动切换目录。

#### 权限和安全

- 提供了具有完整 **RBAC** 功能的新权限系统，支持角色继承和默认角色。有关详细信息，请参阅[权限概述](../administration/privilege_overview.md)。
- 提供了更多权限管理对象和更细粒度的权限。有关更多信息，请参阅[StarRocks 支持的权限](../administration/privilege_item.md)。

#### 查询引擎

<!-- - [Preview] Supports operator **spilling** for large queries, which can use disk space to ensure stable running of queries in case of insufficient memory. -->

- 允许更多的连接表查询从[查询缓存](../using_starrocks/query_cache.md)中受益。例如，查询缓存现在支持 Broadcast Join 和 Bucket Shuffle Join。
- 支持[全局 UDFs](../sql-reference/sql-functions/JAVA_UDF.md)。
- 动态自适应并行性：StarRocks 可以自动调整 pipeline_dop 参数，以实现查询的并行度。

#### SQL 参考

- 添加了以下与权限相关的 SQL 语句：[SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md)、[SET ROLE](../sql-reference/sql-statements/account-management/SET_ROLE.md)、[SHOW ROLES](../sql-reference/sql-statements/account-management/SHOW_ROLES.md) 和 [SHOW USERS](../sql-reference/sql-statements/account-management/SHOW_USERS.md)。
- 新增了以下半结构化数据分析函数：[map_apply](../sql-reference/sql-functions/map-functions/map_apply.md)、[map_from_arrays](../sql-reference/sql-functions/map-functions/map_from_arrays.md)。
- [array_agg](../sql-reference/sql-functions/array-functions/array_agg.md) 支持 ORDER BY。
- 窗口函数 [lead](../sql-reference/sql-functions/Window_function.md#lead) 和 [lag](../sql-reference/sql-functions/Window_function.md#lag) 支持 IGNORE NULLS。
- 新增了字符串函数 [replace](../sql-reference/sql-functions/string-functions/replace.md)、[hex_decode_binary](../sql-reference/sql-functions/string-functions/hex_decode_binary.md) 和 [hex_decode_string()](../sql-reference/sql-functions/string-functions/hex_decode_string.md)。
- 新增了加密函数 [base64_decode_binary](../sql-reference/sql-functions/crytographic-functions/base64_decode_binary.md) 和 [base64_decode_string](../sql-reference/sql-functions/crytographic-functions/base64_decode_string.md)。
- 新增了数学函数 [sinh](../sql-reference/sql-functions/math-functions/sinh.md)、[cosh](../sql-reference/sql-functions/math-functions/cosh.md) 和 [tanh](../sql-reference/sql-functions/math-functions/tanh.md)。
- 新增了实用函数 [current_role](../sql-reference/sql-functions/utility-functions/current_role.md)。

### 改进

#### 部署

- 更新了 3.0 版本的 Docker 镜像和相关的 [Docker 部署文档](../quick_start/deploy_with_docker.md)。 [#20623](https://github.com/StarRocks/starrocks/pull/20623) [#21021](https://github.com/StarRocks/starrocks/pull/21021)

#### 存储引擎和数据摄入

- 支持更多 CSV 参数用于数据摄入，包括 SKIP_HEADER、TRIM_SPACE、ENCLOSE 和 ESCAPE。详见 [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 和 [ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。
- 在[主键表](../table_design/table_types/primary_key_table.md)中解耦了主键和排序键。在创建表时，可以单独在 `ORDER BY` 中指定排序键。
- 优化了大数据量摄入、部分更新、持久化主索引等场景下主键表数据摄入的内存使用。
- 支持创建异步 INSERT 任务。有关详细信息，请参阅 [INSERT](../loading/InsertInto.md#load-data-asynchronously-using-insert) 和 [SUBMIT TASK](../sql-reference/sql-statements/data-manipulation/SUBMIT_TASK.md)。[#20609](https://github.com/StarRocks/starrocks/issues/20609)

#### 物化视图

- 优化了[物化视图](../using_starrocks/Materialized_view.md)的重写能力，包括：
  - 支持重写 View Delta Join、Outer Join 和 Cross Join。
  - 优化了 Union 和分区的 SQL 重写。
- 改进了物化视图构建能力：支持 CTE、select * 和 Union。
- 优化了信息返回的[SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)。
- 支持批量添加 MV 分区，提高了物化视图构建过程中分区添加的效率。 [#21167](https://github.com/StarRocks/starrocks/pull/21167)

#### 查询引擎

- 管道引擎支持所有操作符。后续版本将移除非管道代码。
- 改进了[Big Query Positioning](../administration/monitor_manage_big_queries.md)并添加了大查询日志。[SHOW PROCESSLIST](../sql-reference/sql-statements/Administration/SHOW_PROCESSLIST.md)支持查看 CPU 和内存信息。
- 优化了 Outer Join 重排序。
- 优化了 SQL 解析阶段的错误消息，提供了更准确的错误定位和更清晰的错误提示。

#### 数据湖分析

- 优化了元数据统计收集。
- 支持使用[SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md)查看由外部目录管理并存储在 Apache Hive™、Apache Iceberg、Apache Hudi 或 Delta Lake 中的表的创建语句。

### Bug 修复

- StarRocks'源文件许可证头中的某些URL无法访问。[#2224](https://github.com/StarRocks/starrocks/issues/2224)
- 在SELECT查询期间返回未知错误。[#19731](https://github.com/StarRocks/starrocks/issues/19731)
- 支持 [SHOW/SET CHARACTER](https://github.com/StarRocks/starrocks/issues/17480)。 #17480
- 当加载的数据超过 StarRocks 支持的字段长度时，返回的错误信息不正确。 [#14](https://github.com/StarRocks/DataX/issues/14)
- 支持 `show full fields from 'table'`。[#17233](https://github.com/StarRocks/starrocks/issues/17233)
- 分区修剪导致 MV 重写失败。 [#14641](https://github.com/StarRocks/starrocks/issues/14641)
- 当 CREATE MATERIALIZED VIEW 语句包含 `count(distinct)` 且 `count(distinct)` 应用于 DISTRIBUTED BY 列时，MV 重写失败。 [#16558](https://github.com/StarRocks/starrocks/issues/16558)
- FEs无法在将VARCHAR列用作物化视图的分区列时启动。[#19366](https://github.com/StarRocks/starrocks/issues/19366)
- 窗口函数 [LEAD](../sql-reference/sql-functions/Window_function.md#lead) 和 [LAG](../sql-reference/sql-functions/Window_function.md#lag) 错误处理了 IGNORE NULLS。 [#21001](https://github.com/StarRocks/starrocks/pull/21001)
- 添加临时分区与自动创建分区冲突。 [#21222](https://github.com/StarRocks/starrocks/issues/21222)

### 行为变更

- 新的基于角色的访问控制（RBAC）系统支持之前的权限和角色。然而，相关语句的语法，比如[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)和[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md)，已经发生变化。
- 将 SHOW MATERIALIZED VIEW 重命名为[SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)。
- 新增了以下[保留关键字](../sql-reference/sql-statements/keywords.md)：AUTO_INCREMENT、CURRENT_ROLE、DEFERRED、ENCLOSE、ESCAPE、IMMEDIATE、PRIVILEGES、SKIP_HEADER、TRIM_SPACE、VARBINARY。

### 升级注意事项

您可以从 v2.5 升级到 v3.0 或从 v3.0 降级到 v2.5。

> 理论上，也支持从 v2.5 之前的版本升级。为了确保系统可用性，建议您先将集群升级到 v2.5，然后再升级到 v3.0。

从 v3.0 降级到 v2.5 时，请注意以下要点：

#### BDBJE

StarRocks 在 v3.0 中升级了 BDB 库。但是，BDBJE 不能回滚。降级后必须使用 v3.0 的 BDB 库。请执行以下步骤：

1. 在将 FE 包替换为 v2.5 包后，将 v3.0 的 fe/lib/starrocks-bdb-je-18.3.13.jar 复制到 v2.5 的 fe/lib 目录中。

2. 删除 fe/lib/je-7.*.jar。

#### 权限系统

升级到 v3.0 后，默认使用新的 RBAC 权限系统。您只能降级到 v2.5。

降级后，运行 [ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md) 来创建新镜像，并等待新镜像同步到所有 follower FE。如果您不执行此命令，某些降级操作可能会失败。此命令从 2.5.3 版本开始支持。

有关v2.5和v3.0权限系统差异的详细信息，请参阅[Privileges supported by StarRocks](../administration/privilege_item.md)中的“升级须知”。
