---
displayed_sidebar: "Chinese"
---

# StarRocks 版本 3.1

## 3.1.5

发布日期：2023年11月28日

### 新功能

- StarRocks 共享数据集群的 CN 节点现在支持数据导出。[#34018](https://github.com/StarRocks/starrocks/pull/34018)

### 改进

- 系统数据库 `INFORMATION_SCHEMA` 中的 [`COLUMNS`](../reference/information_schema/columns.md) 视图可以显示 ARRAY、MAP 和 STRUCT 列。[#33431](https://github.com/StarRocks/starrocks/pull/33431)
- 支持针对使用 LZO 压缩并存储在 [Hive](../data_source/catalog/hive_catalog.md) 中的 Parquet、ORC 和 CSV 格式文件的查询。[#30923](https://github.com/StarRocks/starrocks/pull/30923)  [#30721](https://github.com/StarRocks/starrocks/pull/30721)
- 支持对自动分区表的指定分区进行更新。如果指定的分区不存在，则返回错误。[#34777](https://github.com/StarRocks/starrocks/pull/34777)
- 在对创建的物化视图进行 Swap、Drop 或 Schema Change 操作时，支持相关表和物化视图（包括其他与这些物化视图相关的表和物化视图）的自动刷新。[#32829](https://github.com/StarRocks/starrocks/pull/32829)
- 优化了一些与位图相关操作的性能，包括：
  - 优化了嵌套循环连接。[#340804](https://github.com/StarRocks/starrocks/pull/34804)  [#35003](https://github.com/StarRocks/starrocks/pull/35003)
  - 优化了 `bitmap_xor` 函数。[#34069](https://github.com/StarRocks/starrocks/pull/34069)
  - 支持写时复制以优化位图性能并减少内存消耗。[#34047](https://github.com/StarRocks/starrocks/pull/34047)

### Bug 修复

修复了以下问题：

- 在 Broker Load 作业中指定了过滤条件时，在特定情况下在数据加载期间 BEs 可能会崩溃。[#29832](https://github.com/StarRocks/starrocks/pull/29832)
- 执行 SHOW GRANTS 时会报告未知错误。[#30100](https://github.com/StarRocks/starrocks/pull/30100)
- 当加载数据到使用基于表达式的自动分区的表时，可能会抛出错误“Error: The row create partition failed since Runtime error: failed to analyse partition value”。[#33513](https://github.com/StarRocks/starrocks/pull/33513)
- 对查询返回错误“get_applied_rowsets failed, tablet updates is in error state: tablet:18849 actual row size changed after compaction”。[#33246](https://github.com/StarRocks/starrocks/pull/33246)
- 在 StarRocks 共享无集群中，针对 Iceberg 或 Hive 表的查询可能导致 BEs 崩溃。[#34682](https://github.com/StarRocks/starrocks/pull/34682)
- 在 StarRocks 共享无集群中，如果在数据加载期间自动创建了多个分区，则加载的数据可能偶尔写入不匹配的分区。[#34731](https://github.com/StarRocks/starrocks/pull/34731)
- 长时间、频繁地加载数据到启用持久性索引的主键表可能导致 BEs 崩溃。[#33220](https://github.com/StarRocks/starrocks/pull/33220)
- 对查询返回错误“Exception: java.lang.IllegalStateException: null”。[#33535](https://github.com/StarRocks/starrocks/pull/33535)
- 当执行 `show proc '/current_queries';` 时，同时开始执行查询，BEs 可能会崩溃。[#34316](https://github.com/StarRocks/starrocks/pull/34316)
- 如果向启用持久性索引的主键表加载大量数据，则可能引发错误。[#34352](https://github.com/StarRocks/starrocks/pull/34352)
- 在将 StarRocks 从 v2.4 或更早版本升级到后续版本后，整理分数可能会意外上升。[#34618](https://github.com/StarRocks/starrocks/pull/34618)
- 如果使用数据库驱动程序 MariaDB ODBC 查询 `INFORMATION_SCHEMA`，则 `schemata` 视图中返回的 `CATALOG_NAME` 列仅持有 `null` 值。[#34627](https://github.com/StarRocks/starrocks/pull/34627)
- FEs 因加载异常数据并无法重新启动而崩溃。[#34590](https://github.com/StarRocks/starrocks/pull/34590)
- 如果在 **PREPARD** 状态下执行流加载作业时执行模式更改，则作业要加载的部分源数据会丢失。[#34381](https://github.com/StarRocks/starrocks/pull/34381)
- 在 HDFS 存储路径末尾包含两个或更多斜杠（`/`）可能导致从 HDFS 备份和恢复数据失败。[#34601](https://github.com/StarRocks/starrocks/pull/34601)
- 将会话变量 `enable_load_profile` 设置为 `true` 会使流加载作业易于失败。[#34544](https://github.com/StarRocks/starrocks/pull/34544)
- 在主键表的列模式中执行部分更新会导致表的部分片副本显示数据不一致。[#34555](https://github.com/StarRocks/starrocks/pull/34555)
- 使用 ALTER TABLE 语句添加的 `partition_live_number` 属性不生效。[#34842](https://github.com/StarRocks/starrocks/pull/34842)
- FEs 无法启动并报告错误“failed to load journal type 118”。[#34590](https://github.com/StarRocks/starrocks/pull/34590)
- 将 FE 参数 `recover_with_empty_tablet` 设置为 `true` 可能导致 FEs 崩溃。[#33071](https://github.com/StarRocks/starrocks/pull/33071)
- 在重新播放副本操作失败时可能导致 FEs 崩溃。[#32295](https://github.com/StarRocks/starrocks/pull/32295)

### 兼容性更改

#### 参数

- 添加了 FE 配置项 [`enable_statistics_collect_profile`](../administration/Configuration.md#enable_statistics_collect_profile)，用于控制是否为统计查询生成配置文件。默认值为 `false`。[#33815](https://github.com/StarRocks/starrocks/pull/33815)
- FE 配置项 [`mysql_server_version`](../administration/Configuration.md#mysql_server_version) 现在是可变的。无需重新启动 FE 即可使新设置对当前会话生效。[#34033](https://github.com/StarRocks/starrocks/pull/34033)
- 添加了 BE/CN 配置项 [`update_compaction_ratio_threshold`](../administration/Configuration.md#update_compaction_ratio_threshold)，用于控制在 StarRocks 共享数据集群的主键表中紧缩操作可以合并的最大数据比例。默认值为 `0.5`。如果单个片变得过大，建议缩小此值。对于 StarRocks 共享无集群，主键表可合并的数据比例仍然会自动调整。[#35129](https://github.com/StarRocks/starrocks/pull/35129)

#### 系统变量

- 添加了会话变量 `cbo_decimal_cast_string_strict`，用于控制 CBO 如何将 DECIMAL 类型的数据转换为 STRING 类型。如果将此变量设置为 `true`，则 v2.5.x 版本及更高版本中内置的逻辑生效，系统执行严格转换（即系统截断生成的字符串并根据标度长度填充 0）。如果将此变量设置为 `false`，则将使用早于 v2.5.x 版本的逻辑，系统处理所有有效数字以生成字符串。默认值为 `true`。[#34208](https://github.com/StarRocks/starrocks/pull/34208)
- 添加了会话变量 `cbo_eq_base_type`，指定在 DECIMAL 类型数据与 STRING 类型数据之间进行数据比较时使用的数据类型。默认值为 `VARCHAR`，也可以使用 `DECIMAL`。[#34208](https://github.com/StarRocks/starrocks/pull/34208)
- 添加了会话变量 `big_query_profile_second_threshold`。当会话变量 [`enable_profile`](../reference/System_variable.md#enable_profile) 设置为 `false` 且查询花费的时间超过 `big_query_profile_second_threshold` 变量指定的阈值时，为该查询生成配置文件。[#33825](https://github.com/StarRocks/starrocks/pull/33825)

## 3.1.4

发布日期：2023年11月2日

### 新特性

- 支持在共享数据StarRocks集群中创建的主键表上使用排序键。 
- 支持使用str2date函数为异步物化视图指定分区表达式。这有助于增量更新和查询重写，这些异步物化视图创建在外部目录中的表上，并使用STRING类型数据作为其分区表达式。 [#29923](https://github.com/StarRocks/starrocks/pull/29923) [#31964](https://github.com/StarRocks/starrocks/pull/31964)
- 添加了新的会话变量`enable_query_tablet_affinity`，用于控制是否将对同一tablet的多个查询定向到一个固定的副本。此会话变量默认设置为`false`。 [#33049](https://github.com/StarRocks/starrocks/pull/33049)
- 添加了实用函数`is_role_in_session`，用于检查当前会话中是否激活了指定的角色。它支持检查赋予用户的嵌套角色。 [#32984](https://github.com/StarRocks/starrocks/pull/32984)
- 支持设置资源组级别的查询队列，由全局变量`enable_group_level_query_queue`（默认值：`false`）控制。当全局级别或资源组级别的资源消耗达到预定义的阈值时，新的查询将被放置在队列中，并且当全局级别资源消耗和资源组级别资源消耗都低于它们的阈值时才会运行。
  - 用户可以为每个资源组设置`concurrency_limit`，以限制每个BE允许的并发查询的最大数量。
  - 用户可以为每个资源组设置`max_cpu_cores`，以限制每个BE允许的最大CPU消耗。
- 为资源组分类器添加了两个参数，`plan_cpu_cost_range`和`plan_mem_cost_range`。
  - `plan_cpu_cost_range`：系统估算的CPU消耗范围。默认值`NULL`表示不设限制。
  - `plan_mem_cost_range`：系统估算的内存消耗范围。默认值`NULL`表示不设限制。

### 改进

- 窗口函数COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD和STDDEV_SAMP现在支持ORDER BY子句和窗口子句。 [#30786](https://github.com/StarRocks/starrocks/pull/30786)
- 如果在DECIMAL类型数据的查询期间发生小数溢出，则返回错误而不是NULL。 [#30419](https://github.com/StarRocks/starrocks/pull/30419)
- 现在，查询队列中允许的并发查询数量由leader FE进行管理。每个follower FE在查询开始和完成时通知leader FE。如果并发查询数量达到全局级别或资源组级别的`concurrency_limit`，则新的查询将被拒绝或放置在队列中。

### Bug修复

解决了以下问题：

- 由于内存使用统计不准确，Spark或Flink可能会报告数据读取错误。 [#30702](https://github.com/StarRocks/starrocks/pull/30702)  [#30751](https://github.com/StarRocks/starrocks/pull/30751)
- 元数据缓存的内存使用统计不准确。 [#31978](https://github.com/StarRocks/starrocks/pull/31978)
- 在调用libcurl时，BE可能会崩溃。 [#31667](https://github.com/StarRocks/starrocks/pull/31667)
- 当StarRocks物化视图创建在Hive视图上刷新时，会返回错误“java.lang.ClassCastException: com.starrocks.catalog.HiveView无法转换为com.starrocks.catalog.HiveMetaStoreTable”。 [#31004](https://github.com/StarRocks/starrocks/pull/31004)
- 如果ORDER BY子句包含聚合函数，则会返回错误“java.lang.IllegalStateException: null”。 [#30108](https://github.com/StarRocks/starrocks/pull/30108)
- 在共享数据StarRocks集群中，“information_schema.COLUMNS”中未记录表键的信息。因此，使用Flink Connector加载数据时，无法执行DELETE操作。 [#31458](https://github.com/StarRocks/starrocks/pull/31458)
- 使用Flink Connector加载数据时，如果存在高并发加载作业，并且HTTP线程的数量和扫描线程的数量都达到其上限，加载作业将意外暂停。 [#32251](https://github.com/StarRocks/starrocks/pull/32251)
- 当仅添加了几个字节的字段时，在数据更改完成前执行SELECT COUNT(*)将返回错误，错误内容为“error: invalid field name”。 [#33243](https://github.com/StarRocks/starrocks/pull/33243)
- 在启用查询缓存后，查询结果不正确。 [#32781](https://github.com/StarRocks/starrocks/pull/32781)
- 在哈希连接期间查询失败，导致BE崩溃。 [#32219](https://github.com/StarRocks/starrocks/pull/32219)
- `information_schema.columns`视图中对于BINARY或VARBINARY数据类型的`DATA_TYPE`和`COLUMN_TYPE`显示为`unknown`。 [#32678](https://github.com/StarRocks/starrocks/pull/32678)

### 行为更改

- 从v3.1.4版本开始，默认情况下已为新的StarRocks集群创建的主键表启用了持久索引（对于将版本从早期版本升级到v3.1.4的现有StarRocks集群不适用）。 [#33374](https://github.com/StarRocks/starrocks/pull/33374)
- 添加了一个新的FE参数`enable_sync_publish`，默认设置为`true`。当将此参数设置为`true`时，数据加载到主键表的发布阶段仅在应用任务完成后返回执行结果。因此，加载作业在返回成功消息后可以立即查询。但将此参数设置为`true`可能会导致加载到主键表的数据花费更长的时间。 （在添加此参数之前，应用任务与发布阶段是异步的。） [#27055](https://github.com/StarRocks/starrocks/pull/27055)

## 3.1.3

发布日期：2023年9月25日

### 新特性

- 在共享数据StarRocks集群中创建的主键表也支持索引持久性，与共享无关的StarRocks集群一样。
- 聚合函数[group_concat](../sql-reference/sql-functions/string-functions/group_concat.md)支持DISTINCT关键字和ORDER BY子句。 [#28778](https://github.com/StarRocks/starrocks/pull/28778)
- [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md) 、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 、[Kafka Connector](../loading/Kafka-connector-starrocks.md) 、[Flink Connector](../loading/Flink-connector-starrocks.md)  和[Spark Connector](../loading/Spark-connector-starrocks.md) 支持以列模式对主键表进行部分更新。 [#28288](https://github.com/StarRocks/starrocks/pull/28288)
- 分区中的数据可以随着时间自动降温。 （此功能不支持[list partitioning](../table_design/list_partitioning.md)。） [#29335](https://github.com/StarRocks/starrocks/pull/29335) [#29393](https://github.com/StarRocks/starrocks/pull/29393)

### 改进

现在，使用无效注释执行SQL命令将返回与MySQL一致的结果。 [#30210](https://github.com/StarRocks/starrocks/pull/30210)

### Bug修复

解决了以下问题：

- 如果在执行[DELETE](../sql-reference/sql-statements/data-manipulation/DELETE.md)语句的WHERE子句中指定了[BITMAP](../sql-reference/sql-statements/data-types/BITMAP.md)或[HLL](../sql-reference/sql-statements/data-types/HLL.md)数据类型，则无法正确执行该语句。 [#28592](https://github.com/StarRocks/starrocks/pull/28592)
- 在follower FE重新启动后，CpuCores统计数据不是最新的，导致查询性能下降。 [#28472](https://github.com/StarRocks/starrocks/pull/28472) [#30434](https://github.com/StarRocks/starrocks/pull/30434)
- [to_bitmap()](../sql-reference/sql-functions/bitmap-functions/to_bitmap.md)函数的执行成本计算不正确。结果，在重写物化视图后，该函数选择了不恰当的执行计划。 [#29961](https://github.com/StarRocks/starrocks/pull/29961)
- 在共享数据架构的某些用例中，当跟随者 FE 重新启动后，提交给跟随者 FE 的查询会返回一个错误，内容为“找不到后端节点。请检查是否有任何后端节点宕机”。[#28615](https://github.com/StarRocks/starrocks/pull/28615)
- 如果连续加载数据到正在使用 [ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md) 语句进行更改的表中，可能会抛出错误“Tablet is in error state”。[#29364](https://github.com/StarRocks/starrocks/pull/29364)
- 使用 `ADMIN SET FRONTEND CONFIG` 命令修改 FE 动态参数 `max_broker_load_job_concurrency` 无法生效。[#29964](https://github.com/StarRocks/starrocks/pull/29964) [#29720](https://github.com/StarRocks/starrocks/pull/29720)
- 如果在 [date_diff()](../sql-reference/sql-functions/date-time-functions/date_diff.md) 函数中，时间单位是常量但日期不是常量，BE 会崩溃。[#29937](https://github.com/StarRocks/starrocks/issues/29937)
- 在共享数据架构中，启用异步加载后，自动分区无法生效。[#29986](https://github.com/StarRocks/starrocks/issues/29986)
- 如果用户使用 [CREATE TABLE LIKE](../sql-reference/sql-statements/data-definition/CREATE_TABLE_LIKE.md) 语句创建主键表，会抛出错误“意外异常: 未知属性: {persistent_index_type=LOCAL}”。[#30255](https://github.com/StarRocks/starrocks/pull/30255)
- 在 BE 重新启动后，还原主键表会导致元数据不一致。[#30135](https://github.com/StarRocks/starrocks/pull/30135)
- 如果用户在对具有截断操作和查询的主键表进行数据加载，可能会在某些情况下抛出错误“java.lang.NullPointerException”。[#30573](https://github.com/StarRocks/starrocks/pull/30573)
- 如果在物化视图的创建语句中指定了谓词表达式，则这些物化视图的刷新结果是不正确的。[#29904](https://github.com/StarRocks/starrocks/pull/29904)
- 用户将 StarRocks 集群升级到 v3.1.2 后，之前创建的表的存储卷属性会重置为 `null`。[#30647](https://github.com/StarRocks/starrocks/pull/30647)
- 如果对平板元数据同时进行检查点和恢复操作，则会丢失一些平板副本且无法检索。[#30603](https://github.com/StarRocks/starrocks/pull/30603)
- 如果用户使用 CloudCanal 将数据加载到设置为 `NOT NULL` 但未指定默认值的表列中，会抛出错误“不支持的数据格式值: \N”。[#30799](https://github.com/StarRocks/starrocks/pull/30799)

### 行为更改

- 在使用 [group_concat](../sql-reference/sql-functions/string-functions/group_concat.md) 函数时，用户必须使用 SEPARATOR 关键字声明分隔符。
- 会话变量 [`group_concat_max_len`](../reference/System_variable.md#group_concat_max_len) 的默认值已从无限更改为 `1024`，该变量控制 [group_concat](../sql-reference/sql-functions/string-functions/group_concat.md) 函数返回的字符串的默认最大长度。

## 3.1.2

发布日期: 2023年8月25日

### Bug 修复

修复了以下问题：

- 如果用户指定默认连接哪个数据库，并且用户只对数据库中的表具有权限但对数据库没有权限，则会抛出一条错误，指示用户对数据库没有权限。[#29767](https://github.com/StarRocks/starrocks/pull/29767)
- 云原生表的 RESTful API 操作 `show_data` 返回的值不正确。[#29473](https://github.com/StarRocks/starrocks/pull/29473)
- 在运行 [array_agg()](../sql-reference/sql-functions/array-functions/array_agg.md) 函数时取消查询会导致 BE 崩溃。[#29400](https://github.com/StarRocks/starrocks/issues/29400)
- 对于 [BITMAP](../sql-reference/sql-statements/data-types/BITMAP.md) 或 [HLL](../sql-reference/sql-statements/data-types/HLL.md) 数据类型的列，[SHOW FULL COLUMNS](../sql-reference/sql-statements/Administration/SHOW_FULL_COLUMNS.md) 语句返回的“默认”字段值不正确。[#29510](https://github.com/StarRocks/starrocks/pull/29510)
- 如果查询中的 [array_map()](../sql-reference/sql-functions/array-functions/array_map.md) 函数涉及多个表，由于下推策略问题，查询会失败。[#29504](https://github.com/StarRocks/starrocks/pull/29504)
- 由于未合并来自 Apache ORC 的 bug 修复 ORC-1304 ([apache/orc#1299](https://github.com/apache/orc/pull/1299))，导致对 ORC 格式文件的查询失败。[#29804](https://github.com/StarRocks/starrocks/pull/29804)

### 行为更改

对于新部署的 StarRocks v3.1 集群，如果要运行 [SET CATALOG](../sql-reference/sql-statements/data-definition/SET_CATALOG.md) 切换到该目标外部目录，则必须具有目的地外部目录的 USAGE 权限。您可以使用 [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) 授予所需的权限。

对于从早期版本升级的 v3.1 集群，可以使用继承的权限运行 SET CATALOG。

## 3.1.1

发布日期: 2023年8月18日

### 新功能

- 支持 Azure Blob 存储用于[`shared-data clusters`](../deployment/shared_data/s3.md)。
- 为[`共享数据集群`](../deployment/shared_data/s3.md)添加了列表分区支持。
- 支持聚合函数 [COVAR_SAMP](../sql-reference/sql-functions/aggregate-functions/covar_samp.md)、[COVAR_POP](../sql-reference/sql-functions/aggregate-functions/covar_pop.md) 和 [CORR](../sql-reference/sql-functions/aggregate-functions/corr.md)。
- 支持以下[窗口函数](../sql-reference/sql-functions/Window_function.md)：COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD 和 STDDEV_SAMP。

### 改进

支持对 WHERE 子句中的所有复合谓词和所有表达式进行隐式转换。您可以通过使用[会话变量](../reference/System_variable.md) `enable_strict_type` 启用或禁用隐式转换。此会话变量的默认值为 `false`。

### Bug 修复

修复了以下问题：

- 如果在具有多个副本的表中加载数据，且表的一些分区为空，则会写入大量无效的日志记录。[#28824](https://github.com/StarRocks/starrocks/issues/28824)
- 平均行大小的估算不准确，导致主键表上的列模式部分更新占用过大的内存。[#27485](https://github.com/StarRocks/starrocks/pull/27485)
- 如果在 ERROR 状态的平板上触发克隆操作，则磁盘使用量会增加。[#28488](https://github.com/StarRocks/starrocks/pull/28488)
- 合并导致冷数据写入本地缓存。[#28831](https://github.com/StarRocks/starrocks/pull/28831)

## 3.1.0

发布日期: 2023年8月7日

### 新功能

#### 共享数据集群

- 支持无法启用持久索引的主键表。
- 支持 [AUTO_INCREMENT](../sql-reference/sql-statements/auto_increment.md) 列属性，为每个数据行启用全局唯一 ID，从而简化数据管理。
- 支持[在加载过程中自动创建分区并使用分区表达式定义分区规则](../table_design/expression_partitioning.md)，从而使分区创建更加方便和灵活。
- 支持[存储卷的抽象概念](../deployment/shared_data/s3.md#use-your-shared-data-starrocks-cluster)，用户可以配置存储位置和认证信息在共享数据 StarRocks 集群中。用户可以在创建数据库或表时直接引用现有的存储卷，使认证配置更加便捷。

#### 数据湖分析

- 支持访问在[Hive 目录](../data_source/catalog/hive_catalog.md)中创建的视图表。
- 支持访问 Parquet 格式的 Iceberg v2 表。
- 支持[将数据下沉到 Parquet 格式的 Iceberg 表](../data_source/catalog/iceberg_catalog.md#sink-data-to-an-iceberg-table)。
- [预览] 支持通过使用[Elasticsearch 目录](../data_source/catalog/elasticsearch_catalog.md)访问存储在 Elasticsearch 中的数据。这样可以简化创建 Elasticsearch 外部表。
- [预览] 通过使用[Paimon目录](../data_source/catalog/paimon_catalog.md)，支持对存储在Apache Paimon上的流数据执行分析。

#### 存储引擎、数据摄入和查询

- 升级了自动分区至[表达式分区](../table_design/expression_partitioning.md)。用户只需在创建表时使用一个简单的分区表达式（时间函数表达式或列表达式），StarRocks将会根据数据特性和分区表达式中定义的规则在数据载入过程中自动创建分区。该分区创建方法适用于大多数场景，更加灵活和用户友好。
- 支持[列表分区](../table_design/list_partitioning.md)。数据根据预定义列的值列表进行分区，可以加速查询和更高效地管理明确定义的数据。
- 向`Information_schema`数据库增加了名为`loads`的新表。用户可以从`loads`表中查询[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)和[Insert](../sql-reference/sql-statements/data-manipulation/INSERT.md)作业的结果。
- 支持记录被[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)和[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)作业过滤掉的不合格数据行。用户可以在载入作业中使用`log_rejected_record_num`参数来指定可以记录的数据行的最大数量。
- 支持[随机分桶](../table_design/Data_distribution.md#how-to-choose-the-bucketing-columns)。用户无需在表创建时配置分桶列，StarRocks将数据随机分配到分桶中。结合自v2.5.7版以来提供的自动设置分桶数（`BUCKETS`）功能，用户不再需要考虑分桶配置，表创建语句大大简化。然而，在大数据和高性能场景中，我们建议用户继续使用哈希分桶，因为这样可以使用分桶剪枝来加速查询。
- 支持在[INSERT INTO](../loading/InsertInto.md)中使用表函数 FILES()，直接载入存储在AWS S3上的Parquet或ORC格式数据文件的数据。FILES()函数可以自动推断表模式，无需在数据载入之前创建外部目录或文件外部表，大大简化了数据加载过程。
- 支持[生成列](../sql-reference/sql-statements/generated_columns.md)。通过生成列功能，StarRocks可以自动生成并存储列表达式的值，并自动重写查询以提高查询性能。
- 支持使用[Spark连接器](../loading/Spark-connector-starrocks.md)从Spark载入数据到StarRocks。与[Spark Load](../loading/SparkLoad.md)相比，Spark连接器提供了更全面的功能。用户可以定义一个Spark作业对数据进行ETL操作，而Spark连接器则作为Spark作业的接收端。
- 支持将数据载入到[MAP](../sql-reference/sql-statements/data-types/Map.md)和[STRUCT](../sql-reference/sql-statements/data-types/STRUCT.md)数据类型的列中，并支持在ARRAY、MAP和STRUCT中嵌套Fast Decimal值。

#### SQL参考

- 增加了以下与存储容量相关的语句：[CREATE STORAGE VOLUME](../sql-reference/sql-statements/Administration/CREATE_STORAGE_VOLUME.md)、[ALTER STORAGE VOLUME](../sql-reference/sql-statements/Administration/ALTER_STORAGE_VOLUME.md)、[DROP STORAGE VOLUME](../sql-reference/sql-statements/Administration/DROP_STORAGE_VOLUME.md)、[SET DEFAULT STORAGE VOLUME](../sql-reference/sql-statements/Administration/SET_DEFAULT_STORAGE_VOLUME.md)、[DESC STORAGE VOLUME](../sql-reference/sql-statements/Administration/DESC_STORAGE_VOLUME.md)、[SHOW STORAGE VOLUMES](../sql-reference/sql-statements/Administration/SHOW_STORAGE_VOLUMES.md)。

- 支持使用[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)修改表注释。[#21035](https://github.com/StarRocks/starrocks/pull/21035)

- 增加了以下函数：

  - 结构体函数：[struct (row)](../sql-reference/sql-functions/struct-functions/row.md)、[named_struct](../sql-reference/sql-functions/struct-functions/named_struct.md)
  - Map函数：[str_to_map](../sql-reference/sql-functions/string-functions/str_to_map.md)、[map_concat](../sql-reference/sql-functions/map-functions/map_concat.md)、[map_from_arrays](../sql-reference/sql-functions/map-functions/map_from_arrays.md)、[element_at](../sql-reference/sql-functions/map-functions/element_at.md)、[distinct_map_keys](../sql-reference/sql-functions/map-functions/distinct_map_keys.md)、[cardinality](../sql-reference/sql-functions/map-functions/cardinality.md)
  - 高阶Map函数：[map_filter](../sql-reference/sql-functions/map-functions/map_filter.md)、[map_apply](../sql-reference/sql-functions/map-functions/map_apply.md)、[transform_keys](../sql-reference/sql-functions/map-functions/transform_keys.md)、[transform_values](../sql-reference/sql-functions/map-functions/transform_values.md)
  - 数组函数：[array_agg](../sql-reference/sql-functions/array-functions/array_agg.md)支持 `ORDER BY`、[array_generate](../sql-reference/sql-functions/array-functions/array_generate.md)、[element_at](../sql-reference/sql-functions/array-functions/element_at.md)、[cardinality](../sql-reference/sql-functions/array-functions/cardinality.md)
  - 高阶数组函数：[all_match](../sql-reference/sql-functions/array-functions/all_match.md)、[any_match](../sql-reference/sql-functions/array-functions/any_match.md)
  - 聚合函数：[min_by](../sql-reference/sql-functions/aggregate-functions/min_by.md)、[percentile_disc](../sql-reference/sql-functions/aggregate-functions/percentile_disc.md)
  - 表函数：[FILES](../sql-reference/sql-functions/table-functions/files.md)、[generate_series](../sql-reference/sql-functions/table-functions/generate_series.md)
  - 日期函数：[next_day](../sql-reference/sql-functions/date-time-functions/next_day.md)、[previous_day](../sql-reference/sql-functions/date-time-functions/previous_day.md)、[last_day](../sql-reference/sql-functions/date-time-functions/last_day.md)、[makedate](../sql-reference/sql-functions/date-time-functions/makedate.md)、[date_diff](../sql-reference/sql-functions/date-time-functions/date_diff.md)
  - 位图函数：[bitmap_subset_limit](../sql-reference/sql-functions/bitmap-functions/bitmap_subset_limit.md)、[bitmap_subset_in_range](../sql-reference/sql-functions/bitmap-functions/bitmap_subset_in_range.md)
  - 数学函数：[cosine_similarity](../sql-reference/sql-functions/math-functions/cos_similarity.md)、[cosine_similarity_norm](../sql-reference/sql-functions/math-functions/cos_similarity_norm.md)

#### 权限和安全

增加与存储容量相关的[权限项](../administration/privilege_item.md#storage-volume)和外部目录相关的[权限项](../administration/privilege_item.md#catalog)，支持使用[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)和[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md)来授予和撤销这些权限。

### 改进

#### 共享数据集群

优化了共享数据StarRocks集群中的数据缓存。优化后的数据缓存允许指定热数据的范围。它还可以防止针对冷数据的查询占用本地磁盘缓存，从而确保对热数据的查询性能。

#### 材料化视图

- 优化了异步材料化视图的创建：
  - 支持随机分桶。如果用户未指定分桶列，StarRocks默认采用随机分桶。
  - 支持使用 `ORDER BY` 指定排序键。
  - 支持指定属性，如 `colocate_group`、`storage_medium` 和 `storage_cooldown_time`。
  - 支持使用会话变量，用户可以使用 `properties("session.<variable_name>" = "<value>")` 语法来灵活调整视图刷新策略。
  - 对所有异步材料化视图启用溢写功能，并默认实现查询超时时长为1小时。
  - 支持基于视图创建材料化视图。这使得材料化视图在数据建模场景中更容易使用，因为用户可以根据不同需求灵活使用视图和材料化视图来实现分层建模。
- 优化了异步材料化视图的查询重写：
  - 支持陈旧重写，允许在指定的时间间隔内未刷新的材料化视图用于查询重写，而无论材料化视图的基表是否被更新。用户可以在材料化视图创建时使用 `mv_rewrite_staleness_second` 属性来指定时间间隔。
  - 支持针对在Hive目录表（必须定义主键和外键）上创建的材料化视图重写视图增量连接查询。
  - 对包含联合操作的查询重写机制进行了优化，并支持对包含联接或函数（如COUNT DISTINCT和time_slice）的查询进行重写。
- 优化了异步材料化视图的刷新：
- 优化了在Hive目录表上创建的物化视图的刷新机制。StarRocks现在能感知分区级的数据更改，并在每次自动刷新时仅刷新具有数据更改的分区。
  - 支持使用`REFRESH MATERIALIZED VIEW WITH SYNC MODE`语法来同步调用物化视图刷新任务。
- 加强了异步物化视图的使用：
  - 支持使用`ALTER MATERIALIZED VIEW {ACTIVE | INACTIVE}`来启用或禁用物化视图。禁用的（处于`INACTIVE`状态的）物化视图无法进行刷新或用于查询重写，但可以直接查询。
  - 支持使用`ALTER MATERIALIZED VIEW SWAP WITH`来交换两个物化视图。用户可以创建一个新的物化视图，然后与现有的物化视图进行原子交换以实现对现有物化视图的架构更改。
- 优化了同步物化视图：
  - 支持使用SQL提示`[_SYNC_MV_]`直接针对同步物化视图进行查询，从而解决一些查询在少数情况下无法被正确重写的问题。
  - 支持更多的表达式，如`CASE-WHEN`、`CAST`和数学运算，使物化视图适用于更多的业务场景。

#### 数据湖分析

- 优化了对Iceberg的元数据缓存和访问，以提高Iceberg数据查询性能。
- 优化了数据缓存，进一步提升了数据湖分析的性能。

#### 存储引擎、数据摄入和查询

- 宣布了[spill](../administration/spill_to_disk.md)功能的一般可用性，支持将一些阻塞操作符的中间计算结果溢出到磁盘。启用溢出功能后，当查询包含聚合、排序或连接操作符时，StarRocks可以将这些操作符的中间计算结果缓存到磁盘，以减少内存消耗，从而最小化由内存限制引起的查询失败。
- 支持基于保持基数的连接进行修剪。如果用户维护了大量以星型模式（例如SSB）或雪花模式（例如TCP-H）组织的表，但只查询了其中的一小部分表，则该功能有助于修剪不必要的表，以提高连接的性能。
- 支持列模式的部分更新。用户可以在对主键表执行部分更新时使用[UPDATE](../sql-reference/sql-statements/data-manipulation/UPDATE.md)语句启用列模式。列模式适用于更新少量列但大量行，可将更新性能提高多达10倍。
- 优化了对CBO的统计信息收集，减少了统计信息收集对数据摄入的影响，提高了统计信息收集的性能。
- 优化了合并算法，使排列情况下的整体性能提高多达2倍。
- 优化了查询逻辑，减少对数据库锁的依赖。
- 动态分区进一步支持将分区单位设置为年。[#28386](https://github.com/StarRocks/starrocks/pull/28386)

#### SQL参考

- 条件函数`case`、`coalesce`、`if`、`ifnull`和`nullif`支持数组、映射、结构和JSON数据类型。
- 以下数组函数支持嵌套类型MAP、STRUCT和ARRAY：
  - array_agg
  - array_contains、array_contains_all、array_contains_any
  - array_slice、array_concat
  - array_length、array_append、array_remove、array_position
  - reverse、array_distinct、array_intersect、arrays_overlap
  - array_sortby
- 以下数组函数支持快速Decimal数据类型：
  - array_agg
  - array_append、array_remove、array_position、array_contains
  - array_length
  - array_max、array_min、array_sum、array_avg
  - arrays_overlap、array_difference
  - array_slice、array_distinct、array_sort、reverse、array_intersect、array_concat
  - array_sortby、array_contains_all、array_contains_any

### Bug修复

解决了以下问题：

- 不能正确处理 Routine Load 作业重新连接到Kafka的请求。[#23477](https://github.com/StarRocks/starrocks/issues/23477)
- 对涉及多个表并包含 `WHERE` 子句的SQL查询，如果这些SQL查询具有相同的语义但每个SQL查询中表的顺序不同，其中一些SQL查询可能无法重写以获益于相关物化视图。[#22875](https://github.com/StarRocks/starrocks/issues/22875)
- 查询中包含 `GROUP BY` 子句的查询返回重复记录。[#19640](https://github.com/StarRocks/starrocks/issues/19640)
- 调用`lead()`或`lag()`函数可能导致BE崩溃。[#22945](https://github.com/StarRocks/starrocks/issues/22945)
- 基于在外部目录表上创建的物化视图的偏向部分分区查询失败。[#19011](https://github.com/StarRocks/starrocks/issues/19011)
- 无法正确解析同时包含反斜杠（`\`）和分号（`;`）的SQL语句。[#16552](https://github.com/StarRocks/starrocks/issues/16552)
- 如果移除了在表上创建的物化视图，则无法截断表。[#19802](https://github.com/StarRocks/starrocks/issues/19802)

### 行为更改

- 从用于共享数据StarRocks集群的表创建语法中删除了`storage_cache_ttl`参数。现在根据LRU算法清除本地缓存中的数据。
- 将BE配置项`disable_storage_page_cache`和`alter_tablet_worker_count`以及FE配置项`lake_compaction_max_tasks`从不可变参数更改为可变参数。
- BE配置项`block_cache_checksum_enable`的默认值从`true`更改为`false`。
- BE配置项`enable_new_load_on_memory_limit_exceeded`的默认值从`false`更改为`true`。
- FE配置项`max_running_txn_num_per_db`的默认值从`100`更改为`1000`。
- FE配置项`http_max_header_size`的默认值从`8192`更改为`32768`。
- FE配置项`tablet_create_timeout_second`的默认值从`1`更改为`10`。
- FE配置项`max_routine_load_task_num_per_be`的默认值从`5`更改为`16`，且如果创建了大量 Routine Load 任务，将返回错误信息。
- FE配置项`quorom_publish_wait_time_ms`更名为`quorum_publish_wait_time_ms`，FE配置项`async_load_task_pool_size`更名为`max_broker_load_job_concurrency`。
- BE配置项`routine_load_thread_pool_size`已弃用。现在仅通过FE配置项`max_routine_load_task_num_per_be`来控制每个BE节点的例行作业线程池大小。
- BE配置项`txn_commit_rpc_timeout_ms`和系统变量`tx_visible_wait_timeout`已弃用。
- FE配置项`max_broker_concurrency`和`load_parallel_instance_num`已弃用。
- FE配置项`max_routine_load_job_num`已弃用。现在，StarRocks根据`max_routine_load_task_num_per_be`参数动态推断每个个别BE节点支持的 Routine Load 任务的最大数量，并提供任务失败的建议。
- CN配置项`thrift_port`更名为`be_port`。
- 添加了两个Routine Load作业属性，`task_consume_second`和`task_timeout_second`，用于控制在Routine Load作业内消耗数据的最长时间和个别加载任务的超时持续时间，使作业调整更加灵活。如果用户在其Routine Load作业中未指定这两个属性，则将使用FE配置项`routine_load_task_consume_second`和`routine_load_task_timeout_second`。
- 会话变量`enable_resource_group`已经废弃，因为自v3.1.0以来默认启用了[资源组](../administration/resource_group.md)功能。
- 添加了两个新的保留关键字，COMPACTION和TEXT。