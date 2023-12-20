---
displayed_sidebar: English
---

# StarRocks版本3.0

## 3.0.8

发布日期：2023年11月17日

## 改进

- 系统数据库`INFORMATION_SCHEMA`中的`COLUMNS`视图现在可以显示ARRAY、MAP和STRUCT列。[#33431](https://github.com/StarRocks/starrocks/pull/33431)

## Bug修复

修复了以下问题：

- 当执行`show proc '/current_queries';`时，同时有查询开始执行，BE可能会崩溃。[#34316](https://github.com/StarRocks/starrocks/pull/34316)
- 当数据持续高频率地加载到指定了排序键的主键表中时，可能会发生压缩失败。[#26486](https://github.com/StarRocks/starrocks/pull/26486)
- 如果Broker Load作业中指定了过滤条件，在某些情况下BE可能在数据加载过程中崩溃。[#29832](https://github.com/StarRocks/starrocks/pull/29832)
- 当执行SHOW GRANTS时报告未知错误。[#30100](https://github.com/StarRocks/starrocks/pull/30100)
- 如果在`cast()`函数中指定的目标数据类型与原始数据类型相同，对于特定数据类型BE可能会崩溃。[#31465](https://github.com/StarRocks/starrocks/pull/31465)
- 在`information_schema.columns`视图中，BINARY或VARBINARY数据类型的`DATA_TYPE`和`COLUMN_TYPE`显示为`unknown`。[#32678](https://github.com/StarRocks/starrocks/pull/32678)
- 如果长时间频繁地将数据加载到启用了持久索引的主键表中，可能会导致BE崩溃。[#33220](https://github.com/StarRocks/starrocks/pull/33220)
- 当启用查询缓存时，查询结果可能不正确。[#32778](https://github.com/StarRocks/starrocks/pull/32778)
- 集群重启后，恢复的表中的数据可能与备份前的数据不一致。[#33567](https://github.com/StarRocks/starrocks/pull/33567)
- 如果在执行RESTORE时同时进行Compaction，可能会导致BE崩溃。[#32902](https://github.com/StarRocks/starrocks/pull/32902)

## 3.0.7

发布日期：2023年10月18日

### 改进

- 窗口函数COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD和STDDEV_SAMP现在支持ORDER BY子句和Window子句。[#30786](https://github.com/StarRocks/starrocks/pull/30786)
- 将数据写入主键表的加载作业的发布阶段从异步模式更改为同步模式，这样加载作业完成后数据即可被查询。[#27055](https://github.com/StarRocks/starrocks/pull/27055)
- 如果在查询DECIMAL类型数据时发生小数溢出，则返回错误而不是NULL。[#30419](https://github.com/StarRocks/starrocks/pull/30419)
- 执行带有无效注释的SQL命令现在返回与MySQL一致的结果。[#30210](https://github.com/StarRocks/starrocks/pull/30210)
- 对于使用单一分区列或表达式分区的RANGE分区StarRocks表，SQL谓词中包含的分区列表达式也可用于分区修剪。[#30421](https://github.com/StarRocks/starrocks/pull/30421)

### Bug修复

修复了以下问题：

- 在某些情况下，同时创建和删除数据库和表可能导致找不到表，进而导致数据加载失败。[#28985](https://github.com/StarRocks/starrocks/pull/28985)
- 在某些情况下，使用UDF可能导致内存泄漏。[#29467](https://github.com/StarRocks/starrocks/pull/29467) [#29465](https://github.com/StarRocks/starrocks/pull/29465)
- 如果ORDER BY子句包含聚合函数，则返回错误"java.lang.IllegalStateException: null"。[#30108](https://github.com/StarRocks/starrocks/pull/30108)
- 如果用户使用Hive目录查询腾讯COS中存储的数据，并且目录包含多个级别，则查询结果将不正确。[#30363](https://github.com/StarRocks/starrocks/pull/30363)
- 如果ARRAY<STRUCT>类型数据中的STRUCT的某些子字段缺失，在查询时填充缺失子字段的默认值时数据长度不正确，可能导致BE崩溃。
- Berkeley DB Java Edition的版本已升级，以避免安全漏洞。[#30029](https://github.com/StarRocks/starrocks/pull/30029)
- 如果用户在对主键表进行数据加载的同时执行截断操作和查询，某些情况下可能会抛出"java.lang.NullPointerException"错误。[#30573](https://github.com/StarRocks/starrocks/pull/30573)
- 如果Schema Change执行时间过长，可能因为指定版本的tablet被垃圾回收而失败。[#31376](https://github.com/StarRocks/starrocks/pull/31376)
- 如果用户使用CloudCanal将数据加载到设置为`NOT NULL`但未指定默认值的表列中，则会抛出错误"Unsupported dataFormat value is : \N"。[#30799](https://github.com/StarRocks/starrocks/pull/30799)
- 在StarRocks共享数据集群中，`information_schema.COLUMNS`中未记录表键信息，导致使用Flink Connector加载数据时无法执行DELETE操作。[#31458](https://github.com/StarRocks/starrocks/pull/31458)
- 在升级过程中，如果某些列的类型也升级（例如，从Decimal类型升级为Decimal v3类型），对具有特定特征的表进行压缩可能会导致BE崩溃。[#31626](https://github.com/StarRocks/starrocks/pull/31626)
- 使用Flink Connector加载数据时，如果存在高并发加载作业，并且HTTP线程数和Scan线程数都达到上限，加载作业可能会意外暂停。[#32251](https://github.com/StarRocks/starrocks/pull/32251)
- 当调用libcurl时，BE可能会崩溃。[#31667](https://github.com/StarRocks/starrocks/pull/31667)
- 当向主键表中添加BITMAP类型的列时，可能会发生错误。[#31763](https://github.com/StarRocks/starrocks/pull/31763)

## 3.0.6

发布日期：2023年9月12日

### 行为变更

- 使用[group_concat](../sql-reference/sql-functions/string-functions/group_concat.md)函数时，必须使用SEPARATOR关键字声明分隔符。

### 新特性

- 聚合函数[group_concat](../sql-reference/sql-functions/string-functions/group_concat.md)支持DISTINCT关键字和ORDER BY子句。[#28778](https://github.com/StarRocks/starrocks/pull/28778)
- 分区中的数据可以随时间自动冷却。（此功能不支持[list partitioning](../table_design/list_partitioning.md)。）[#29335](https://github.com/StarRocks/starrocks/pull/29335) [#29393](https://github.com/StarRocks/starrocks/pull/29393)

### 改进

- 支持所有复合谓词和WHERE子句中的所有表达式的隐式转换。您可以使用[会话变量](../reference/System_variable.md)`enable_strict_type`启用或禁用隐式转换。此会话变量的默认值为`false`。[#21870](https://github.com/StarRocks/starrocks/pull/21870)
- 在FE和BE之间统一了将字符串转换为整数的逻辑。[#29969](https://github.com/StarRocks/starrocks/pull/29969)

### Bug修复

- 如果`enable_orc_late_materialization`设置为`true`，在使用Hive目录查询ORC文件中STRUCT类型数据时，可能返回意外结果。[#27971](https://github.com/StarRocks/starrocks/pull/27971)
- 通过Hive Catalog进行数据查询时，如果WHERE子句中指定了分区列和OR运算符，查询结果可能不正确。[#28876](https://github.com/StarRocks/starrocks/pull/28876)
- 云原生表的RESTful API操作`show_data`返回的值不正确。[#29473](https://github.com/StarRocks/starrocks/pull/29473)
- 如果共享数据集群将数据存储在Azure Blob Storage中并创建了表，当集群回滚到3.0版本后，FE可能无法启动。[#29433](https://github.com/StarRocks/starrocks/pull/29433)
- 即使用户被授予了对表的权限，当查询Iceberg目录中的表时，用户可能没有权限。[#29173](https://github.com/StarRocks/starrocks/pull/29173)
- 对于BITMAP或HLL数据类型的列，SHOW FULL COLUMNS语句返回的`Default`字段值不正确。[#29510](https://github.com/StarRocks/starrocks/pull/29510)
- 使用`ADMIN SET FRONTEND CONFIG`命令修改FE动态参数`max_broker_load_job_concurrency`不生效。
- 当刷新物化视图且同时修改其刷新策略时，FE可能无法启动。[#29964](https://github.com/StarRocks/starrocks/pull/29964) [#29720](https://github.com/StarRocks/starrocks/pull/29720)
- 执行`select count(distinct(int+double)) from table_name`时返回错误`unknown error`。[#29691](https://github.com/StarRocks/starrocks/pull/29691)
- 在主键表恢复后，如果BE重启，可能会出现元数据错误并导致元数据不一致。[#30135](https://github.com/StarRocks/starrocks/pull/30135)

## 3.0.5

发布日期：2023年8月16日

### 新特性

- 支持聚合函数[COVAR_SAMP](../sql-reference/sql-functions/aggregate-functions/covar_samp.md)、[COVAR_POP](../sql-reference/sql-functions/aggregate-functions/covar_pop.md)和[CORR](../sql-reference/sql-functions/aggregate-functions/corr.md)。
- 支持以下[窗口函数](../sql-reference/sql-functions/Window_function.md)：COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD和STDDEV_SAMP。

### 改进

- 在错误消息`xxx too many versions xxx`中添加了更多提示。[#28397](https://github.com/StarRocks/starrocks/pull/28397)
```
发布日期：2023年4月28日

### 新功能

#### 系统架构

- **存储与计算分离。** StarRocks现在支持将数据持久化到S3兼容的对象存储，增强了资源隔离，降低了存储成本，并使计算资源更具可扩展性。本地磁盘被用作热数据缓存，以提升查询性能。当本地磁盘缓存命中时，新的共享数据架构的查询性能与经典架构（共享无存储）相当。有关更多信息，请参阅[部署和使用共享数据StarRocks](../deployment/shared_data/s3.md)。

#### 存储引擎和数据摄取

- 支持[AUTO_INCREMENT](../sql-reference/sql-statements/auto_increment.md)属性，提供全局唯一ID，简化数据管理。
- 支持[自动分区和分区表达式](../table_design/dynamic_partitioning.md)，使分区创建更易于使用和更灵活。
- 主键表支持更完整的[UPDATE](../sql-reference/sql-statements/data-manipulation/UPDATE.md)和[DELETE](../sql-reference/sql-statements/data-manipulation/DELETE.md#primary-key-tables)语法，包括使用CTE和引用多个表。
- 为Broker Load和INSERT INTO作业添加了Load Profile。您可以通过查询load profile来查看加载作业的详细信息。使用方法与[分析查询profile](../administration/query_profile.md)相同。

#### 数据湖分析

- [预览] 支持Presto/Trino兼容的方言。Presto/Trino的SQL可以自动重写为StarRocks的SQL模式。有关更多信息，请参阅[系统变量](../reference/System_variable.md)`sql_dialect`。
- [预览] 支持[JDBC目录](../data_source/catalog/jdbc_catalog.md)。
- 支持使用[SET CATALOG](../sql-reference/sql-statements/data-definition/SET_CATALOG.md)在当前会话中手动切换目录。

#### 权限和安全

- 提供了一个全新的基于角色的访问控制（RBAC）权限系统，支持角色继承和默认角色。有关更多信息，请参阅[权限概述](../administration/privilege_overview.md)。
- 提供了更多的权限管理对象和更细粒度的权限。有关更多信息，请参阅[StarRocks支持的权限](../administration/privilege_item.md)。

#### 查询引擎

<!-- - [预览] 支持大型查询的运算符**溢出**，可以使用磁盘空间确保查询在内存不足的情况下稳定运行。 -->

- 允许更多的联接表查询受益于[查询缓存](../using_starrocks/query_cache.md)。例如，现在的查询缓存支持Broadcast Join和Bucket Shuffle Join。
- 支持[Global UDFs](../sql-reference/sql-functions/JAVA_UDF.md)。
- 动态自适应并行度：StarRocks可以自动调整查询并发的`pipeline_dop`参数。

#### SQL参考

- 添加了以下与权限相关的SQL语句：[SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md)，[SET ROLE](../sql-reference/sql-statements/account-management/SET_ROLE.md)，[SHOW ROLES](../sql-reference/sql-statements/account-management/SHOW_ROLES.md)和[SHOW USERS](../sql-reference/sql-statements/account-management/SHOW_USERS.md)。
- 添加了以下半结构化数据分析函数：[map_apply](../sql-reference/sql-functions/map-functions/map_apply.md)，[map_from_arrays](../sql-reference/sql-functions/map-functions/map_from_arrays.md)。
- [array_agg](../sql-reference/sql-functions/array-functions/array_agg.md)支持ORDER BY。
- 窗口函数[lead](../sql-reference/sql-functions/Window_function.md#lead)和[lag](../sql-reference/sql-functions/Window_function.md#lag)支持IGNORE NULLS。
- 添加了字符串函数[replace](../sql-reference/sql-functions/string-functions/replace.md)，[hex_decode_binary](../sql-reference/sql-functions/string-functions/hex_decode_binary.md)和[hex_decode_string()](../sql-reference/sql-functions/string-functions/hex_decode_string.md)。
- 添加了加密函数[base64_decode_binary](../sql-reference/sql-functions/crytographic-functions/base64_decode_binary.md)和[base64_decode_string](../sql-reference/sql-functions/crytographic-functions/base64_decode_string.md)。
- 添加了数学函数[sinh](../sql-reference/sql-functions/math-functions/sinh.md)，[cosh](../sql-reference/sql-functions/math-functions/cosh.md)和[tanh](../sql-reference/sql-functions/math-functions/tanh.md)。
- 添加了实用函数[current_role](../sql-reference/sql-functions/utility-functions/current_role.md)。

### 改进

#### 部署

- 更新了版本3.0的Docker镜像和相关的[Docker部署文档](../quick_start/deploy_with_docker.md)。[#20623](https://github.com/StarRocks/starrocks/pull/20623) [#21021](https://github.com/StarRocks/starrocks/pull/21021)

#### 存储引擎和数据摄取

- 支持更多CSV参数进行数据摄取，包括SKIP_HEADER, TRIM_SPACE, ENCLOSE和ESCAPE。参见[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)，[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)和[ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。
- 在[主键表](../table_design/table_types/primary_key_table.md)中，主键和排序键解耦。创建表时可以在`ORDER BY`中单独指定排序键。
- 优化了大量摄取、部分更新和持久化主索引等场景下主键表数据摄取的内存使用。
- 支持创建异步INSERT任务。有关更多信息，请参阅[INSERT](../loading/InsertInto.md#load-data-asynchronously-using-insert)和[SUBMIT TASK](../sql-reference/sql-statements/data-manipulation/SUBMIT_TASK.md)。[#20609](https://github.com/StarRocks/starrocks/issues/20609)

#### 物化视图

- 优化了[物化视图](../using_starrocks/Materialized_view.md)的重写能力，包括：
  - 支持View Delta Join、Outer Join和Cross Join的重写。
  - 优化了带分区Union的SQL重写。
- 改进了物化视图构建能力：支持CTE、select *和Union。
- 优化了[SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)返回的信息。
- 支持批量添加MV分区，提高了物化视图构建过程中分区添加的效率。[#21167](https://github.com/StarRocks/starrocks/pull/21167)

#### 查询引擎

- 所有操作符都支持在管道引擎中使用。非管道代码将在后续版本中移除。
- 改进了[大查询定位](../administration/monitor_manage_big_queries.md)和添加了大查询日志。[SHOW PROCESSLIST](../sql-reference/sql-statements/Administration/SHOW_PROCESSLIST.md)支持查看CPU和内存信息。
- 优化了Outer Join Reorder。
- 优化了SQL解析阶段的错误消息，提供了更准确的错误定位和更清晰的错误消息。

#### 数据湖分析

- 优化了元数据统计收集。
- 支持使用[SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md)查看由外部目录管理并存储在Apache Hive™、Apache Iceberg、Apache Hudi或Delta Lake中的表的创建语句。

### Bug修复

- StarRocks源文件的许可证头中的一些URL无法访问。[#2224](https://github.com/StarRocks/starrocks/issues/2224)
- 在SELECT查询期间返回未知错误。[#19731](https://github.com/StarRocks/starrocks/issues/19731)
- 支持SHOW/SET CHARACTER。[#17480](https://github.com/StarRocks/starrocks/issues/17480)
- 当加载的数据超过StarRocks支持的字段长度时，返回的错误消息不正确。[#14](https://github.com/StarRocks/DataX/issues/14)
- 支持`show full fields from 'table'`。[#17233](https://github.com/StarRocks/starrocks/issues/17233)
- 分区剪枝导致MV重写失败。[#14641](https://github.com/StarRocks/starrocks/issues/14641)
- 当CREATE MATERIALIZED VIEW语句包含`count(distinct)`并且`count(distinct)`应用于DISTRIBUTED BY列时，MV重写失败。[#16558](https://github.com/StarRocks/starrocks/issues/16558)
- 当VARCHAR列用作物化视图的分区列时，FE启动失败。[#19366](https://github.com/StarRocks/starrocks/issues/19366)
- 窗口函数[LEAD](../sql-reference/sql-functions/Window_function.md#lead)和[LAG](../sql-reference/sql-functions/Window_function.md#lag)处理IGNORE NULLS不正确。[#21001](https://github.com/StarRocks/starrocks/pull/21001)
- 添加临时分区与自动分区创建冲突。[#21222](https://github.com/StarRocks/starrocks/issues/21222)

### 行为变更

- 新的基于角色的访问控制（RBAC）权限系统默认支持之前的权限和角色。但是，相关语句如[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)和[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md)的语法已更改。
- 将SHOW MATERIALIZED VIEW重命名为[SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)。
- 添加了以下[保留关键字](../sql-reference/sql-statements/keywords.md)：AUTO_INCREMENT, CURRENT_ROLE, DEFERRED, ENCLOSE, ESCAPE, IMMEDIATE, PRIVILEGES, SKIP_HEADER, TRIM_SPACE, VARBINARY。

### 升级注意事项

您可以从v2.5升级到v3.0，也可以从v3.0降级到v2.5。

> 理论上，也支持从早于v2.5的版本升级。为了确保系统可用性，我们建议您首先将集群升级到v2.5，然后再升级到v3.0。

当您从v3.0降级到v2.5时，请注意以下几点。

#### BDBJE

StarRocks在v3.0中升级了BDB库。但是，BDBJE不能回滚。降级后，您必须使用v3.0的BDB库。执行以下步骤：

1. 在用v2.5包替换FE包后，将v3.0的`fe/lib/starrocks-bdb-je-18.3.13.jar`复制到v2.5的`fe/lib`目录中。

2. 删除`fe/lib/je-7.*.jar`。

#### 权限系统

升级到v3.0后，默认使用新的RBAC权限系统。您只能降级到v2.5。

降级后，运行[ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md)创建新镜像，并等待新镜像同步到所有follower FE。如果您不运行此命令，某些降级操作可能会失败。该命令从2.5.3及更高版本开始支持。

有关v2.5和v3.0权限系统之间差异的详细信息，请参阅[StarRocks支持的权限](../administration/privilege_item.md)中的"升级注意事项"。
发布日期：2023 年 4 月 28 日

### 新功能

#### 系统架构

- **解耦存储和计算。** StarRocks 现在支持将数据持久化到兼容 S3 的对象存储中，从而增强资源隔离、降低存储成本并使计算资源更具可扩展性。本地磁盘被用作热数据缓存，以提升查询性能。当本地磁盘缓存命中时，新的共享数据架构的查询性能与经典架构（无共享）相当。有关更多信息，请参阅[部署和使用共享数据 StarRocks](../deployment/shared_data/s3.md)。

#### 存储引擎和数据摄取

- 支持 `AUTO_INCREMENT` 属性以提供全局唯一的 ID，从而简化数据管理。
- 支持[自动分区和分区表达式](../table_design/dynamic_partitioning.md)，使分区创建更易用、更灵活。
- 主键表支持更完整的 `UPDATE` 和 `DELETE` 语法，包括 CTE 的使用和对多个表的引用。
- 添加了 `Broker Load` 和 `INSERT INTO` 作业的负载配置文件。您可以通过查询负载配置文件来查看负载作业的详细信息。用法与[分析查询配置文件](../administration/query_profile.md)相同。

#### 数据湖分析

- [预览] 支持 Presto/Trino 兼容方言。Presto/Trino 的 SQL 可以自动重写为 StarRocks 的 SQL 模式。有关详细信息，请参阅[系统变量](../reference/System_variable.md) `sql_dialect`。
- [预览] 支持 [JDBC 目录](../data_source/catalog/jdbc_catalog.md)。
- 支持使用 [SET CATALOG](../sql-reference/sql-statements/data-definition/SET_CATALOG.md) 在当前会话中手动切换目录。

#### 权限和安全

- 提供具有完整 RBAC 功能的新权限系统，支持角色继承和默认角色。有关详细信息，请参阅[权限概述](../administration/privilege_overview.md)。
- 提供更多的权限管理对象和更细粒度的权限。有关更多信息，请参阅 [StarRocks 支持的权限](../administration/privilege_item.md)。

#### 查询引擎

- 允许对连接表进行更多查询，以从[查询缓存](../using_starrocks/query_cache.md)中受益。例如，查询缓存现在支持 Broadcast Join 和 Bucket Shuffle Join。
- 支持 [Global UDFs](../sql-reference/sql-functions/JAVA_UDF.md)。
- 动态自适应并行：StarRocks 可以自动调整 `pipeline_dop` 参数以实现查询并发。

#### SQL 参考

- 添加了以下与权限相关的 SQL 语句：[SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md)、[SET ROLE](../sql-reference/sql-statements/account-management/SET_ROLE.md)、[SHOW ROLES](../sql-reference/sql-statements/account-management/SHOW_ROLES.md) 和 [SHOW USERS](../sql-reference/sql-statements/account-management/SHOW_USERS.md)。
- 新增以下半结构化数据分析函数：[map_apply](../sql-reference/sql-functions/map-functions/map_apply.md)、[map_from_arrays](../sql-reference/sql-functions/map-functions/map_from_arrays.md)。
- [array_agg](../sql-reference/sql-functions/array-functions/array_agg.md) 支持 ORDER BY。
- 窗口函数 [lead](../sql-reference/sql-functions/Window_function.md#lead) 和 [lag](../sql-reference/sql-functions/Window_function.md#lag) 支持 IGNORE NULLS。
- 添加了字符串函数 [replace](../sql-reference/sql-functions/string-functions/replace.md)、[hex_decode_binary](../sql-reference/sql-functions/string-functions/hex_decode_binary.md) 和 [hex_decode_string()](../sql-reference/sql-functions/string-functions/hex_decode_string.md)。
- 添加了加密函数 [base64_decode_binary](../sql-reference/sql-functions/cryptographic-functions/base64_decode_binary.md) 和 [base64_decode_string](../sql-reference/sql-functions/cryptographic-functions/base64_decode_string.md)。
- 添加了数学函数 [sinh](../sql-reference/sql-functions/math-functions/sinh.md)、[cosh](../sql-reference/sql-functions/math-functions/cosh.md) 和 [tanh](../sql-reference/sql-functions/math-functions/tanh.md)。
- 添加了实用程序函数 [current_role](../sql-reference/sql-functions/utility-functions/current_role.md)。

### 改进

#### 部署

- 更新了 3.0 版本的 Docker 镜像和相关 [Docker 部署文档](../quick_start/deploy_with_docker.md)。[#20623](https://github.com/StarRocks/starrocks/pull/20623) [#21021](https://github.com/StarRocks/starrocks/pull/21021)

#### 存储引擎和数据摄取

- 支持更多用于数据摄取的 CSV 参数，包括 SKIP_HEADER、TRIM_SPACE、ENCLOSE 和 ESCAPE。请参阅 [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 和 [ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。
- 在 [主键表](../table_design/table_types/primary_key_table.md) 中，主键和排序键是解耦的。在创建表时，可以在 `ORDER BY` 中单独指定排序键。
- 优化了大数据量摄取、部分更新、持久化主索引等场景下数据摄取到主键表的内存使用情况。
- 支持创建异步 `INSERT` 任务。有关详细信息，请参阅 [INSERT](../loading/InsertInto.md#load-data-asynchronously-using-insert) 和 [SUBMIT TASK](../sql-reference/sql-statements/data-manipulation/SUBMIT_TASK.md)。[#20609](https://github.com/StarRocks/starrocks/issues/20609)

#### 物化视图

- 优化了 [物化视图](../using_starrocks/Materialized_view.md) 的重写能力，包括：
  - 支持 View Delta Join、Outer Join、Cross Join 的重写。
  - 优化 Union 与分区的 SQL 重写。
- 改进了物化视图构建功能：支持 CTE、select *、Union。
- 优化了 [SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md) 返回的信息。
- 支持批量添加 MV 分区，提高了在构建物化视图时添加分区的效率。[#21167](https://github.com/StarRocks/starrocks/pull/21167)

#### 查询引擎

- 管道引擎支持所有运算符。非管道代码将在后续版本中删除。
- 改进了 [Big Query Positioning](../administration/monitor_manage_big_queries.md) 并添加了大查询日志。[SHOW PROCESSLIST](../sql-reference/sql-statements/Administration/SHOW_PROCESSLIST.md) 支持查看 CPU 和内存信息。
- 优化了 Outer Join Reorder。
- 优化了 SQL 解析阶段的错误提示，提供了更准确的错误定位和更清晰的错误信息。

#### 数据湖分析

- 优化了元数据统计收集。
- 支持使用 [SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md) 查看外部目录管理的存储在 Apache Hive™、Apache Iceberg、Apache Hudi 或 Delta Lake 中的表的创建语句。

### Bug 修复

- StarRocks 源文件的许可证头中的某些 URL 无法访问。[#2224](https://github.com/StarRocks/starrocks/issues/2224)
- 在 SELECT 查询期间返回了一个未知错误。[#19731](https://github.com/StarRocks/starrocks/issues/19731)
- 支持 `SHOW/SET CHARACTER`。[#17480](https://github.com/StarRocks/starrocks/issues/17480)
- 当加载的数据超过 StarRocks 支持的字段长度时，返回的错误信息不正确。[#14](https://github.com/StarRocks/DataX/issues/14)
- 支持 `show full fields from 'table'`。[#17233](https://github.com/StarRocks/starrocks/issues/17233)
- 分区修剪导致 MV 重写失败。[#14641](https://github.com/StarRocks/starrocks/issues/14641)
- MV 重写失败，当 CREATE MATERIALIZED VIEW 语句包含 `count(distinct)` 并且 `count(distinct)` 应用于 DISTRIBUTED BY 列时。[#16558](https://github.com/StarRocks/starrocks/issues/16558)
- 当使用 VARCHAR 列作为物化视图的分区列时，FEs 无法启动。[#19366](https://github.com/StarRocks/starrocks/issues/19366)
- 窗口函数 [LEAD](../sql-reference/sql-functions/Window_function.md#lead) 和 [LAG](../sql-reference/sql-functions/Window_function.md#lag) 错误地处理 IGNORE NULLS。[#21001](https://github.com/StarRocks/starrocks/pull/21001)
- 添加临时分区与自动创建分区冲突。[#21222](https://github.com/StarRocks/starrocks/issues/21222)

### 行为变更

- 新的基于角色的访问控制（RBAC）系统支持之前的权限和角色。然而，相关语句的语法，比如 [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) 和 [REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md)，已经发生了变化。
- 将 `SHOW MATERIALIZED VIEW` 重命名为 [SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)。
- 添加了以下 [保留关键字](../sql-reference/sql-statements/keywords.md)：AUTO_INCREMENT、CURRENT_ROLE、DEFERRED、ENCLOSE、ESCAPE、IMMEDIATE、PRIVILEGES、SKIP_HEADER、TRIM_SPACE、VARBINARY。

### 升级注意事项

您可以从 v2.5 升级到 v3.0 或从 v3.0 降级到 v2.5。

> 理论上也支持从 v2.5 之前的版本升级。为保证系统可用性，建议您先将集群升级到 v2.5，然后再升级到 v3.0。

从 v3.0 降级到 v2.5 时，请注意以下几点。

#### BDBJE

StarRocks 在 v3.0 中升级了 BDB 库。但是，BDBJE 无法回滚。降级后必须使用 v3.0 的 BDB 库。执行以下步骤：

1. 将 FE 包替换为 v2.5 包后，将 v3.0 的 `fe/lib/starrocks-bdb-je-18.3.13.jar` 复制到 v2.5 的 `fe/lib` 目录中。

2. 删除 `fe/lib/je-7.*.jar`。

#### 权限系统

升级到 v3.0 后，默认使用新的 RBAC 权限系统。您只能降级到 v2.5。

降级后，运行 [ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md) 创建新镜像，并等待新镜像同步到所有 follower FEs。如果您不运行此命令，某些降级操作可能会失败。此命令从 2.5.3 及更高版本支持。

有关 v2.5 和 v3.0 权限系统之间的差异的详细信息，请参阅 [StarRocks 支持的权限](../administration/privilege_item.md) 中的“升级注意事项”。
降级后，运行 [`ALTER SYSTEM CREATE IMAGE`](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md) 来创建新的镜像，并等待新镜像同步到所有 follower FE。如果不执行此命令，某些降级操作可能会失败。该命令从 2.5.3 及更高版本开始支持。
