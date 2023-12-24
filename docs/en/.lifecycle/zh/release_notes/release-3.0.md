---
displayed_sidebar: English
---

# StarRocks 3.0 版本

## 3.0.8

发布日期：2023年11月17日

## 改进

- 系统数据库 `INFORMATION_SCHEMA` 中的 `COLUMNS` 视图可以显示 ARRAY、MAP 和 STRUCT 列。 [#33431](https://github.com/StarRocks/starrocks/pull/33431)

## Bug 修复

修复了以下问题：

- 在执行 `show proc '/current_queries';` 查询时，同时开始执行查询时，BE 可能会崩溃。 [#34316](https://github.com/StarRocks/starrocks/pull/34316)
- 当数据连续加载到主键表中，并指定了高频率的排序键时，可能会发生压缩失败。 [#26486](https://github.com/StarRocks/starrocks/pull/26486)
- 如果在 Broker Load 作业中指定了过滤条件，在某些情况下，BE 可能会在数据加载期间崩溃。 [#29832](https://github.com/StarRocks/starrocks/pull/29832)
- 执行 SHOW GRANTS 时报告未知错误。 [#30100](https://github.com/StarRocks/starrocks/pull/30100)
- 如果函数中指定的目标数据类型与原始数据类型相同，`cast()` 函数可能会导致特定数据类型的 BE 崩溃。[#31465](https://github.com/StarRocks/starrocks/pull/31465)
- `information_schema.columns` 视图中对于 BINARY 或 VARBINARY 数据类型的 `DATA_TYPE` 和 `COLUMN_TYPE` 显示为 `unknown`。 [#32678](https://github.com/StarRocks/starrocks/pull/32678)
- 长时间、频繁地将数据加载到启用了持久索引的主键表中可能会导致 BE 崩溃。 [#33220](https://github.com/StarRocks/starrocks/pull/33220)
- 启用查询缓存时，查询结果不正确。 [#32778](https://github.com/StarRocks/starrocks/pull/32778)
- 重启集群后，恢复后的表中的数据可能与备份前的数据不一致。 [#33567](https://github.com/StarRocks/starrocks/pull/33567)
- 如果执行 RESTORE 的同时进行 Compaction，可能会导致 BE 崩溃。 [#32902](https://github.com/StarRocks/starrocks/pull/32902)

## 3.0.7

发布日期：2023年10月18日

### 改进

- 窗口函数 COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD 和 STDDEV_SAMP 现在支持 ORDER BY 子句和 Window 子句。 [#30786](https://github.com/StarRocks/starrocks/pull/30786)
- 将数据写入主键表的加载作业的发布阶段从异步模式更改为同步模式。因此，加载作业完成后，可以立即查询加载的数据。 [#27055](https://github.com/StarRocks/starrocks/pull/27055)
- 如果在对 DECIMAL 类型数据进行查询期间发生十进制溢出，则返回错误而不是 NULL。 [#30419](https://github.com/StarRocks/starrocks/pull/30419)
- 现在，执行带有无效注释的 SQL 命令将返回与 MySQL 一致的结果。 [#30210](https://github.com/StarRocks/starrocks/pull/30210)
- 对于只使用一个分区列的 RANGE 分区或表达式分区的 StarRocks 表，也可以使用包含分区列表达式的 SQL 谓词进行分区修剪。 [#30421](https://github.com/StarRocks/starrocks/pull/30421)

### Bug 修复

修复了以下问题：

- 在某些情况下，同时创建和删除数据库和表可能会导致找不到该表，并进一步导致无法将数据加载到该表中。 [#28985](https://github.com/StarRocks/starrocks/pull/28985)
- 在某些情况下，使用 UDF 可能会导致内存泄漏。 [#29467](https://github.com/StarRocks/starrocks/pull/29467) [#29465](https://github.com/StarRocks/starrocks/pull/29465)
- 如果 ORDER BY 子句包含聚合函数，则返回错误“java.lang.IllegalStateException：null”。 [#30108](https://github.com/StarRocks/starrocks/pull/30108)
- 如果用户使用多层级的 Hive 目录对存储在腾讯 COS 中的数据进行查询，则查询结果会不正确。 [#30363](https://github.com/StarRocks/starrocks/pull/30363)
- 如果 ARRAY< 中 STRUCT 的某些子字段结构>类型数据缺失，查询时缺失的子字段填充默认值时数据长度不正确，导致 BE 崩溃。
- 对Berkeley DB Java版进行升级，避免安全漏洞。[#30029](https://github.com/StarRocks/starrocks/pull/30029)
- 如果用户将数据加载到同时执行截断操作和查询的主键表中，则在某些情况下会抛出错误“java.lang.NullPointerException”。 [#30573](https://github.com/StarRocks/starrocks/pull/30573)
- 如果架构更改执行时间过长，则可能会失败，因为指定版本的平板电脑是垃圾回收的。 [#31376](https://github.com/StarRocks/starrocks/pull/31376)
- 如果用户使用 CloudCanal 将数据加载到已设置但`NOT NULL`未指定默认值的表列中，则会抛出错误“不支持的 dataFormat 值为：\N”。 [#30799](https://github.com/StarRocks/starrocks/pull/30799)
- 在 StarRocks 共享数据集群中，表键的信息不会记录在 中 `information_schema.COLUMNS`。因此，使用Flink Connector加载数据时，无法执行DELETE操作。 [#31458](https://github.com/StarRocks/starrocks/pull/31458)
- 在升级过程中，如果某些列的类型也升级（例如，从 Decimal 类型升级为 Decimal v3 类型），则对某些具有特定特征的表进行压缩可能会导致 BE 崩溃。 [#31626](https://github.com/StarRocks/starrocks/pull/31626)
- 使用Flink Connector加载数据时，如果存在高并发加载作业，且HTTP线程数和扫描线程数均达到上限，则加载作业会意外挂起。 [#32251](https://github.com/StarRocks/starrocks/pull/32251)
- 调用 libcurl 时 BE 崩溃。 [#31667](https://github.com/StarRocks/starrocks/pull/31667)
- 将 BITMAP 类型的列添加到主键表中时发生错误。 [#31763](https://github.com/StarRocks/starrocks/pull/31763)

## 3.0.6

发布日期：2023年9月12日

### 行为改变

- 使用 [group_concat](../sql-reference/sql-functions/string-functions/group_concat.md) 函数时，必须使用 SEPARATOR 关键字声明分隔符。

### 新功能

- 聚合函数 [group_concat](../sql-reference/sql-functions/string-functions/group_concat.md) 支持 DISTINCT 关键字和 ORDER BY 子句。 [#28778](https://github.com/StarRocks/starrocks/pull/28778)
- 分区中的数据可以随时间推移自动冷却。（此功能不支持 [列表分区](../table_design/list_partitioning.md)。 #29335 [ ](https://github.com/StarRocks/starrocks/pull/29335)#29393[ ](https://github.com/StarRocks/starrocks/pull/29393)

### 改进

- 支持对 WHERE 子句中的所有复合谓词和所有表达式进行隐式转换。您可以使用 [session 变量](../reference/System_variable.md)`enable_strict_type` 启用或禁用隐式转换。此会话变量的缺省值为 `false`。 [#21870](https://github.com/StarRocks/starrocks/pull/21870)
- 统一 FE 和 BE 之间的逻辑，将字符串转换为整数。 [#29969](https://github.com/StarRocks/starrocks/pull/29969)

### Bug 修复

- 如果 `enable_orc_late_materialization` 设置为 `true`，则使用 Hive 目录查询 ORC 文件中的 STRUCT 类型数据时，会返回意外结果。 [#27971](https://github.com/StarRocks/starrocks/pull/27971)
- 通过 Hive Catalog 进行数据查询时，如果在 WHERE 子句中指定了分区列和 OR 运算符，则查询结果不正确。 [#28876](https://github.com/StarRocks/starrocks/pull/28876)
- 云原生表 `show_data` 的 RESTful API 操作返回的值不正确。[#29473](https://github.com/StarRocks/starrocks/pull/29473)
- 如果 [共享数据群集](../deployment/shared_data/azure.md) 将数据存储在 Azure Blob 存储中，并且创建了表，则在群集回滚到 3.0 版本后，FE 无法启动。 [#29433](https://github.com/StarRocks/starrocks/pull/29433)
- 用户在查询 Iceberg 目录中的表时没有权限，即使该用户被授予了该表的权限。 [#29173](https://github.com/StarRocks/starrocks/pull/29173)
- `SHOW FULL COLUMNS` 语句返回的 BITMAP 或 HLL 数据类型的列的 `Default` 字段值不正确。 [#29510](https://github.com/StarRocks/starrocks/pull/29510)
- 修改 FE 动态参数 `max_broker_load_job_concurrency` 使用 `ADMIN SET FRONTEND CONFIG` 命令不生效。
- 当实例化视图被刷新时，FE 可能无法启动，而其刷新策略正在被修改。 #[29964](https://github.com/StarRocks/starrocks/pull/29964) [#29720](https://github.com/StarRocks/starrocks/pull/29720)
- 执行时返回 `unknown error` 错误 `select count(distinct(int+double)) from table_name` 。 [#29691](https://github.com/StarRocks/starrocks/pull/29691)
- 恢复主键表后，如果重新启动 BE，则会出现元数据错误并导致元数据不一致。 [#30135](https://github.com/StarRocks/starrocks/pull/30135)

## 3.0.5

发布日期：2023年8月16日

### 新功能

- 支持聚合函数 [COVAR_SAMP](../sql-reference/sql-functions/aggregate-functions/covar_samp.md)、[COVAR_POP](../sql-reference/sql-functions/aggregate-functions/covar_pop.md) 和 [CORR](../sql-reference/sql-functions/aggregate-functions/corr.md)。
- 支持以下 [窗口函数](../sql-reference/sql-functions/Window_function.md)：COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD 和 STDDEV_SAMP。

### 改进
- 在错误消息中添加了更多提示 `xxx too many versions xxx`。 [#28397](https://github.com/StarRocks/starrocks/pull/28397)
- 进一步支持动态分区，使分区单元为 year。 [#28386](https://github.com/StarRocks/starrocks/pull/28386)
- 在创建表时，使用表达式分区和 [INSERT OVERWRITE 覆盖特定分区中的数据时，分区字段不区分大小写](../table_design/expression_partitioning.md#load-data-into-partitions)。 [#28309](https://github.com/StarRocks/starrocks/pull/28309)

### Bug 修复

修复了以下问题：

- FE 中的表级扫描统计信息不正确，导致表查询和加载指标不准确。 [#27779](https://github.com/StarRocks/starrocks/pull/27779)
- 如果修改了分区表的排序键，则查询结果不稳定。 [#27850](https://github.com/StarRocks/starrocks/pull/27850)
- 恢复数据后，平板电脑的 BE 和 FE 版本号不一致。 [#26518](https://github.com/StarRocks/starrocks/pull/26518/files)
- 如果用户创建 Colocation 表时未指定 Bucket 编号，则该编号将被推断为 0，这会导致添加新分区失败。 [#27086](https://github.com/StarRocks/starrocks/pull/27086)
- 当 INSERT INTO SELECT 的 SELECT 结果集为空时，SHOW LOAD 返回的加载作业状态为 `CANCELED`。 [#26913](https://github.com/StarRocks/starrocks/pull/26913)
- 当 sub_bitmap 函数的输入值不是 BITMAP 类型时，BE 可能会崩溃。 [#27982](https://github.com/StarRocks/starrocks/pull/27982)
- 更新 AUTO_INCREMENT 列时，BE 可能会崩溃。 [#27199](https://github.com/StarRocks/starrocks/pull/27199)
- 外部联接和反联接的重写错误。 [#28028](https://github.com/StarRocks/starrocks/pull/28028)
- 对平均行大小的不准确估计导致主键部分更新占用过大的内存。 [#27485](https://github.com/StarRocks/starrocks/pull/27485)
- 激活非活动物化视图可能导致 FE 崩溃。 [#27959](https://github.com/StarRocks/starrocks/pull/27959)
- 查询不能重写为基于 Hudi 目录中的外部表创建的物化视图。 [#28023](https://github.com/StarRocks/starrocks/pull/28023)
- 即使删除了 Hive 表，手动更新了元数据缓存，仍然可以查询到 Hive 表的数据。 [#28223](https://github.com/StarRocks/starrocks/pull/28223)
- 通过同步调用手动刷新异步物化视图会导致表中出现多个 INSERT OVERWRITE 记录 `information_schema.task_runs` 。 [#28060](https://github.com/StarRocks/starrocks/pull/28060)
- LabelCleaner 线程阻塞导致的 FE 内存泄漏。 [#28311](https://github.com/StarRocks/starrocks/pull/28311)

## 3.0.4

发布日期：2023年7月18日

### 新功能

即使查询包含与物化视图不同的联接类型，也可以重写查询。 [#25099](https://github.com/StarRocks/starrocks/pull/25099)

### 改进

- 优化了异步物化视图的手动刷新。支持使用 REFRESH MATERIALIZED VIEW WITH SYNC MODE 语法同步调用物化视图刷新任务。 [#25910](https://github.com/StarRocks/starrocks/pull/25910)
- 如果查询的字段未包含在物化视图的输出列中，而是包含在物化视图的谓词中，则仍可重写查询以从物化视图中受益。 [#23028](https://github.com/StarRocks/starrocks/issues/23028)
- 当 SQL 方言 (`sql_dialect`) 设置为 `trino` 时，表别名不区分大小写。 [#26094](https://github.com/StarRocks/starrocks/pull/26094) [#25282](https://github.com/StarRocks/starrocks/pull/25282)
- 在 `Information_schema.tables_config` 表中添加了一个新字段 `table_id`。您可以将该表 `tables_config` 与数据库 `be_tablets` 中的 `table_id` 列上的 `Information_schema` 表进行联接，以查询平板电脑所属的数据库和表的名称。 [#24061](https://github.com/StarRocks/starrocks/pull/24061)

### Bug 修复

修复了以下问题：

- 如果将包含 sum 聚合函数的查询重写为直接从单表物化视图获取查询结果，则 sum() 字段中的值可能由于类型推断问题而不正确。 [#25512](https://github.com/StarRocks/starrocks/pull/25512)
- 使用 SHOW PROC 查看 StarRocks 共享数据集群中的 Tablet 信息时，会出现错误。
- 当要插入的 STRUCT 中 CHAR 数据的长度超过最大长度时，INSERT 操作将挂起。 [#25942](https://github.com/StarRocks/starrocks/pull/25942)
- 对于带有 FULL JOIN 的 INSERT INTO SELECT，无法返回查询的某些数据行。 [#26603](https://github.com/StarRocks/starrocks/pull/26603)
- 当使用 ALTER TABLE 语句修改表的属性时，会显示错误 `ERROR xxx: Unknown table property xxx` `default.storage_medium`。 [#25870](https://github.com/StarRocks/starrocks/issues/25870)
- 使用 Broker Load 装入空文件时出现错误。 [#26212](https://github.com/StarRocks/starrocks/pull/26212)
- 有时停用 BE 会挂起。 [#26509](https://github.com/StarRocks/starrocks/pull/26509)

## 3.0.3

发布日期：2023年6月28日

### 改进

- StarRocks 外部表的元数据同步已更改为在数据加载期间进行。 [#24739](https://github.com/StarRocks/starrocks/pull/24739)
- 用户在对自动创建分区的表运行 INSERT OVERWRITE 时可以指定分区。有关详细信息，请参阅 [自动分区](https://docs.starrocks.io/en-us/3.0/table_design/dynamic_partitioning)。 [#25005](https://github.com/StarRocks/starrocks/pull/25005)
- 优化未分区表添加分区时报错信息。 [#25266](https://github.com/StarRocks/starrocks/pull/25266)

### Bug 修复

修复了以下问题：

- 当 Parquet 文件包含复杂数据类型时，最小/最大筛选器会获取错误的 Parquet 字段。 [#23976](https://github.com/StarRocks/starrocks/pull/23976)
- 即使删除了相关的数据库或表，加载任务仍在排队。 [#24801](https://github.com/StarRocks/starrocks/pull/24801)
- FE 重启导致 BE 崩溃的可能性很低。 [#25037](https://github.com/StarRocks/starrocks/pull/25037)
- 当变量 `enable_profile` 设置为 `true` 时，加载和查询作业偶尔会冻结。 [#25060](https://github.com/StarRocks/starrocks/pull/25060)
- 在活动 BE 少于 3 个的集群上执行 INSERT OVERWRITE 时，会显示不准确的错误消息。 [#25314](https://github.com/StarRocks/starrocks/pull/25314)

## 3.0.2

发布日期：2023年6月13日

### 改进

- 在异步物化视图重写查询后，可以向下推送 UNION 查询中的谓词。 [#23312](https://github.com/StarRocks/starrocks/pull/23312)
- 优化了[表的自动平板分发策略](../table_design/Data_distribution.md#determine-the-number-of-buckets)。 [#24543](https://github.com/StarRocks/starrocks/pull/24543)
- 删除了 NetworkTime 对系统时钟的依赖性，修复了由于服务器之间的系统时钟不一致而导致的不正确的 NetworkTime。 [#24858](https://github.com/StarRocks/starrocks/pull/24858)

### Bug 修复

修复了以下问题：

- 如果数据加载与架构更改同时发生，则架构更改有时可能会挂起。 [#23456](https://github.com/StarRocks/starrocks/pull/23456)
- 当会话变量 `pipeline_profile_level` 设置为 `0` 时，查询会遇到错误。 [#23873](https://github.com/StarRocks/starrocks/pull/23873)
- CREATE TABLE 设置为 `S3` 时遇到错误 `cloud_native_storage_type`。
- 即使未使用密码，LDAP 身份验证也会成功。 [#24862](https://github.com/StarRocks/starrocks/pull/24862)
- 如果加载作业中涉及的表不存在，则 CANCEL LOAD 将失败。 [#24922](https://github.com/StarRocks/starrocks/pull/24922)

### 升级说明

如果您的系统有一个名为 `starrocks` 的数据库，升级前请使用 ALTER DATABASE RENAME 将其更改为另一个名称。这是因为 `starrocks` 是存储权限信息的默认系统数据库的名称。

## 3.0.1

发布日期：2023年6月1日

### 新功能

- [预览] 支持将大型算子的中间计算结果溢出到磁盘上，减少大型算子的内存消耗。有关详细信息，请参阅 [溢出到磁盘](../administration/spill_to_disk.md)。
- [例程](../loading/RoutineLoad.md#load-avro-format-data)加载支持加载 Avro 数据。
- 支持 [Microsoft Azure 存储](../integrations/authenticate_to_azure_storage.md)（包括 Azure Blob 存储和 Azure Data Lake 存储）。

### 改进

- 共享数据集群支持使用 StarRocks 外部表将数据同步到另一个 StarRocks 集群。
- 添加到 `load_tracking_logs` [信息架构](../reference/information_schema/load_tracking_logs.md) 以记录最近的加载错误。
- 忽略 CREATE TABLE 语句中的特殊字符。 [#23885](https://github.com/StarRocks/starrocks/pull/23885)

### Bug 修复

修复了以下问题：

- 对于主键表，SHOW CREATE TABLE 返回的信息不正确。 [#24237](https://github.com/StarRocks/starrocks/issues/24237)
- BE 可能会在例行加载作业期间崩溃。 [#20677](https://github.com/StarRocks/starrocks/issues/20677)
- 如果在创建分区表时指定了不支持的属性，则会发生空指针异常（NPE）。 [#21374](https://github.com/StarRocks/starrocks/issues/21374)
- SHOW TABLE STATUS 返回的信息不完整。 [#24279](https://github.com/StarRocks/starrocks/issues/24279)

### 升级说明

如果您的系统有一个名为 `starrocks` 的数据库，升级前请使用 ALTER DATABASE RENAME 将其更改为其他名称。这是因为 `starrocks` 是一个默认系统数据库的名称，用于存储权限信息。

## 3.0.0

发布日期：2023年4月28日

### 新功能

#### 系统架构

- **解耦存储和计算。** StarRocks 现在支持将数据持久化到兼容 S3 的对象存储中，增强资源隔离，降低存储成本，并提高计算资源的可扩展性。本地磁盘用作热数据缓存，以提升查询性能。当命中本地磁盘缓存时，新的共享数据架构的查询性能与经典架构（无共享）相当。有关更多信息，请参见 [部署和使用共享数据 StarRocks](../deployment/shared_data/s3.md)。

#### 存储引擎和数据引入

- 支持 [AUTO_INCREMENT](../sql-reference/sql-statements/auto_increment.md) 属性，提供全局唯一的 ID，简化数据管理。
- 支持[自动分区和分区表达式](../table_design/dynamic_partitioning.md)，使分区创建更易于使用和更灵活。
- 主键表支持更完整的 [UPDATE](../sql-reference/sql-statements/data-manipulation/UPDATE.md) 和 [DELETE](../sql-reference/sql-statements/data-manipulation/DELETE.md#primary-key-tables) 语法，包括使用 CTE 和引用多个表。
- 为 Broker Load 和 INSERT INTO 作业添加了 Load Profile。您可以通过查询负载配置文件来查看负载作业的详细信息。用法与[分析查询配置文件](../administration/query_profile.md)相同。

#### 数据湖分析

- [预览] 支持 Presto/Trino 兼容方言。Presto/Trino 的 SQL 可以自动改写为 StarRocks 的 SQL 模式。有关更多信息，请参见[系统变量](../reference/System_variable.md)`sql_dialect`。
- [预览] 支持[JDBC 目录](../data_source/catalog/jdbc_catalog.md)。
- 支持使用 [SET CATALOG](../sql-reference/sql-statements/data-definition/SET_CATALOG.md) 在当前会话中手动切换目录。

#### 权限和安全

- 提供具有完整 RBAC 功能的新权限系统，支持角色继承和默认角色。有关详细信息，请参见 [权限概述](../administration/privilege_overview.md)。
- 提供更多权限管理对象和更细粒度的权限。有关详细信息，请参见 [StarRocks 支持的权限](../administration/privilege_item.md)。

#### 查询引擎

<!-- - [预览] 支持运算符**spilling**，用于大型查询，可使用磁盘空间以确保查询在内存不足的情况下稳定运行。 -->
- 允许对联接表的更多查询从[查询缓存](../using_starrocks/query_cache.md)中受益。例如，查询缓存现在支持 Broadcast Join 和 Bucket Shuffle Join。
- 支持[全局 UDF](../sql-reference/sql-functions/JAVA_UDF.md)。
- 动态自适应并行：StarRocks 可以自动调整 `pipeline_dop` 查询并发参数。

#### SQL 参考

- 添加了以下与权限相关的 SQL 语句：  SET DEFAULT ROLE [ ](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md)、[SET ROLE、](../sql-reference/sql-statements/account-management/SET_ROLE.md)[SHOW ROLES 和 ](../sql-reference/sql-statements/account-management/SHOW_ROLES.md)SHOW USERS[。](../sql-reference/sql-statements/account-management/SHOW_USERS.md)
- 新增map_apply[、](../sql-reference/sql-functions/map-functions/map_apply.md)map_from_arrays[等半结构化数据分析功能](../sql-reference/sql-functions/map-functions/map_from_arrays.md)。
- [array_agg](../sql-reference/sql-functions/array-functions/array_agg.md) 支持 ORDER BY。
- 窗口函数 [lead](../sql-reference/sql-functions/Window_function.md#lead) 和 [lag](../sql-reference/sql-functions/Window_function.md#lag) 支持 IGNORE NULLS。
- 添加了字符串函数 [replace](../sql-reference/sql-functions/string-functions/replace.md)、[hex_decode_binary](../sql-reference/sql-functions/string-functions/hex_decode_binary.md) 和 [hex_decode_string（）。](../sql-reference/sql-functions/string-functions/hex_decode_string.md)
- 增加了加密功能 [base64_decode_binary](../sql-reference/sql-functions/crytographic-functions/base64_decode_binary.md) 和 [base64_decode_string](../sql-reference/sql-functions/crytographic-functions/base64_decode_string.md)。
- 添加了数学函数 [sinh](../sql-reference/sql-functions/math-functions/sinh.md)、[cosh](../sql-reference/sql-functions/math-functions/cosh.md) 和 [tanh](../sql-reference/sql-functions/math-functions/tanh.md)。
- 增加了实用功能 [current_role](../sql-reference/sql-functions/utility-functions/current_role.md)。

### 改进

#### 部署

- 更新了 Docker 映像和相关的[Docker 部署文档](../quick_start/deploy_with_docker.md)。[#20623](https://github.com/StarRocks/starrocks/pull/20623) [#21021](https://github.com/StarRocks/starrocks/pull/21021)

#### 存储引擎和数据引入

- 支持更多 CSV 参数进行数据引入，包括 SKIP_HEADER、TRIM_SPACE、ENCLOSE 和 ESCAPE。请参阅 [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[BROKER LOAD ](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)和 [ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。
- 主键和排序键在主键表中是分离[](../table_design/table_types/primary_key_table.md)的。排序键可以在 `ORDER BY` 创建表时单独指定。
- 优化大容量引入、部分更新、持久化主索引等场景下数据引入主键表的内存使用。
- 支持创建异步 INSERT 任务。有关详细信息，请参阅 [INSERT](../loading/InsertInto.md#load-data-asynchronously-using-insert) 和 [SUBMIT TASK。](../sql-reference/sql-statements/data-manipulation/SUBMIT_TASK.md) [#20609](https://github.com/StarRocks/starrocks/issues/20609)

#### 物化视图

- 优化物化视图[的重写能力](../using_starrocks/Materialized_view.md)，包括：
  - 支持重写 View Delta Join、Outer Join 和 Cross Join。
  - 优化了 Union with partition 的 SQL 重写。
- 改进了物化视图构建功能：支持 CTE、select * 和 Union。
- 优化 SHOW MATERIALIZED VIEWS[ 返回的信息](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)。
- 支持批量添加 MV 分区，提高了物化视图构建过程中的分区添加效率。 [#21167](https://github.com/StarRocks/starrocks/pull/21167)

#### 查询引擎

- 管道引擎支持所有运算符。非管道代码将在以后的版本中删除。
- 改进了[大查询定位，并添加了大查询日志。 ](../administration/monitor_manage_big_queries.md)[SHOW PROCESSLIST](../sql-reference/sql-statements/Administration/SHOW_PROCESSLIST.md) 支持查看 CPU 和内存信息。
- 优化了外部连接重新排序。
- 优化SQL解析阶段的错误信息，提供更准确的错误定位和更清晰的错误信息。

#### 数据湖分析

- 优化元数据统计收集。
- 支持使用 [SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md) 查看外部目录管理并存储在 Apache Hive™、Apache Iceberg、Apache Hudi 或 Delta Lake 中的表的创建语句。

### Bug 修复

- StarRocks 源文件许可证头中的部分 URL 无法访问。 [#2224](https://github.com/StarRocks/starrocks/issues/2224)
- 在 SELECT 查询期间返回未知错误。 [#19731](https://github.com/StarRocks/starrocks/issues/19731)
- 支持 SHOW/SET CHARACTER。 [#17480](https://github.com/StarRocks/starrocks/issues/17480)
- 当加载的数据超出 StarRocks 支持的字段长度时，返回的错误信息不正确。 [#14](https://github.com/StarRocks/DataX/issues/14)
- 支持 `show full fields from 'table'`. [#17233](https://github.com/StarRocks/starrocks/issues/17233)
- 分区修剪会导致 MV 重写失败。 [#14641](https://github.com/StarRocks/starrocks/issues/14641)
- 当 CREATE MATERIALIZED VIEW 语句包含 `count(distinct)` 并 `count(distinct)` 应用于 DISTRIBUTED BY 列时，MV 重写失败。[#16558](https://github.com/StarRocks/starrocks/issues/16558)
- 当 VARCHAR 列用作物化视图的分区列时，FE 无法启动。 [#19366](https://github.com/StarRocks/starrocks/issues/19366)
- 窗口函数 [LEAD](../sql-reference/sql-functions/Window_function.md#lead) 和 [LAG](../sql-reference/sql-functions/Window_function.md#lag) 不正确地处理 IGNORE NULL。 [#21001](https://github.com/StarRocks/starrocks/pull/21001)
- 添加临时分区与自动创建分区冲突。 [#21222](https://github.com/StarRocks/starrocks/issues/21222)

### 行为变更

- 新的基于角色的访问控制（RBAC）系统支持以前的权限和角色。但是，GRANT [](../sql-reference/sql-statements/account-management/GRANT.md) 和 [REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md) 等相关语句的语法发生了变化。
- 将 SHOW MATERIALIZED VIEW 重命名为 SHOW MATERIALIZED [VIEWS。](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)
- 添加了以下[保留关键字](../sql-reference/sql-statements/keywords.md)：AUTO_INCREMENT、CURRENT_ROLE、DEFERRED、ENCLOSE、ESCAPE、IMMEDIATE、PRIVILEGES、SKIP_HEADER、TRIM_SPACE、VARBINARY。

### 升级说明

您可以从 v2.5 升级到 v3.0，也可以从 v3.0 降级到 v2.5。

> 理论上，还支持从 v2.5 之前的版本升级。为保证系统可用性，建议先将集群升级到v2.5，再升级到v3.0。

从 v3.0 降级到 v2.5 时，请注意以下几点。

#### BDBJE

StarRocks 在 v3.0 版本中升级了 BDB 库。但是，BDBJE 无法回滚。降级后必须使用 v3.0 的 BDB 库。执行以下步骤：

1. 将 FE 包替换为 v2.5 包后，将  v3.0 `fe/lib/starrocks-bdb-je-18.3.13.jar` 复制到 v2.5 `fe/lib` 目录下。

2. 删除 `fe/lib/je-7.*.jar`.

#### 权限系统


升级到 v3.0 后，默认使用新的 RBAC 权限系统。您只能降级到 v2.5。

降级后，请执行 [ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md) 命令以创建新的镜像，并等待新镜像同步到所有的跟随 FE。如果您不运行此命令，某些降级操作可能会失败。此命令从 2.5.3 版本开始支持。

有关 v2.5 和 v3.0 权限系统之间的区别详情，请参阅 [StarRocks 支持的权限](../administration/privilege_item.md) 中的“升级说明”部分。