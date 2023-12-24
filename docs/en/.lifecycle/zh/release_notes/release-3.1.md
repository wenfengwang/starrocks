---
displayed_sidebar: English
---

# StarRocks 3.1 版本

## 3.1.6

发布日期：2023年12月18日

### 新特性

- 添加了 [now(p)](https://docs.starrocks.io/zh/docs/3.2/sql-reference/sql-functions/date-time-functions/now/) 函数，返回具有指定小数秒精度（精确到微秒）的当前日期和时间。如果未指定 `p`，此函数仅返回精确到秒的日期和时间。 [#36676](https://github.com/StarRocks/starrocks/pull/36676)
- 添加了一个新的指标 `max_tablet_rowset_num`，用于设置允许的最大行集数。该指标有助于检测可能的压缩问题，从而减少“版本过多”错误的发生。 [#36539](https://github.com/StarRocks/starrocks/pull/36539)
- 支持使用命令行工具获取堆配置文件，使故障排除更加容易。[#35322](https://github.com/StarRocks/starrocks/pull/35322)
- 支持使用公共表达式（CTE）创建异步物化视图。 [#36142](https://github.com/StarRocks/starrocks/pull/36142)
- 添加了以下位图函数：[subdivide_bitmap](https://docs.starrocks.io/zh/docs/sql-reference/sql-functions/bitmap-functions/subdivide_bitmap/)、[bitmap_from_binary](https://docs.starrocks.io/zh/docs/sql-reference/sql-functions/bitmap-functions/bitmap_from_binary/) 和 [bitmap_to_binary](https://docs.starrocks.io/zh/docs/sql-reference/sql-functions/bitmap-functions/bitmap_to_binary/)。 [#35817](https://github.com/StarRocks/starrocks/pull/35817) [#35621](https://github.com/StarRocks/starrocks/pull/35621)

### 参数变更

- 将 FE 动态参数 `enable_new_publish_mechanism` 更改为静态参数。修改参数设置后，需要重新启动 FE。[#35338](https://github.com/StarRocks/starrocks/pull/35338)
- 将回收站文件的默认保留期从原来的3天更改为1天。 [#37113](https://github.com/StarRocks/starrocks/pull/37113)
- 新增 [FE 配置项](https://docs.starrocks.io/zh/docs/administration/Configuration/#配置-fe-动态参数) `routine_load_unstable_threshold_second`。 [#36222](https://github.com/StarRocks/starrocks/pull/36222)
- 新增 BE 配置项 `enable_stream_load_verbose_log`。默认值为 `false`。当该参数设置为 `true` 时，StarRocks 可以记录 Stream Load 作业的 HTTP 请求和响应，从而简化故障排查。 [#36113](https://github.com/StarRocks/starrocks/pull/36113)
- 新增 BE 配置项 `enable_lazy_delta_column_compaction`。默认值为 `true`，表示 StarRocks 不对增量列进行频繁的压缩操作。 [#36654](https://github.com/StarRocks/starrocks/pull/36654)

### 改进

- 在 session 变量 `sql_mode` 中新增了 value 选项 `GROUP_CONCAT_LEGACY`，以提供与 v2.5 之前版本中 `group_concat` 函数实现逻辑的兼容性。[#36150](https://github.com/StarRocks/starrocks/pull/36150)
- SHOW DATA 语句返回的主键表大小包括 **.cols** 文件（这些是与部分列更新和生成的列相关的文件）和持久索引文件的大小。[#34898](https://github.com/StarRocks/starrocks/pull/34898)
- 对 MySQL 外部表和 JDBC 目录中的外部表的查询支持在 WHERE 子句中包含关键字。[#35917](https://github.com/StarRocks/starrocks/pull/35917)
- 插件加载失败将不再导致错误或导致 FE 启动失败。相反，FE 可以正常启动，并且可以使用 [SHOW PLUGINS 查询插件的错误状态](https://docs.starrocks.io/docs/sql-reference/sql-statements/Administration/SHOW_PLUGINS/)。[#36566](https://github.com/StarRocks/starrocks/pull/36566)
- 动态分区支持随机分布。[#35513](https://github.com/StarRocks/starrocks/pull/35513)
- SHOW ROUTINE LOAD 语句返回的结果提供了一个新 `OtherMsg` 字段，该字段显示有关上次失败任务的信息。[#35806](https://github.com/StarRocks/starrocks/pull/35806)
- Broker Load 作业中的 AWS S3 `aws.s3.access_key` 的身份验证信息和 `aws.s3.access_secret` 信息隐藏在审核日志中。[#36571](https://github.com/StarRocks/starrocks/pull/36571)
- 数据库中的 `be_tablets` 视图 `information_schema` 提供了一个新字段 `INDEX_DISK`，用于记录持久索引的磁盘使用情况（以字节为单位）。[#35615](https://github.com/StarRocks/starrocks/pull/35615)

### Bug 修复

修复以下问题：

- 如果用户在数据损坏时创建持久索引，则 BE 会崩溃。[#30841](https://github.com/StarRocks/starrocks/pull/30841)
- 如果用户创建包含嵌套查询的异步实例化视图，则会报告错误“解析分区列失败”。[#26078](https://github.com/StarRocks/starrocks/issues/26078)
- 如果用户在数据损坏的基表上创建异步实例化视图，则会报告错误“意外异常：null”。[#30038](https://github.com/StarRocks/starrocks/pull/30038)
- 如果用户运行包含窗口函数的查询，则会报 SQL 错误 “[1064] [42000] ： Row count of const column reach limit： 4294967296”。[#33561](https://github.com/StarRocks/starrocks/pull/33561)
- 将 FE 配置项设置为 `true` 后，FE 性能骤降。[#35945](https://github.com/StarRocks/starrocks/pull/35945)
- 在 StarRocks 共享数据模式下，当用户尝试从对象存储中删除文件时，可能会报错“降低请求速率”。[#35566](https://github.com/StarRocks/starrocks/pull/35566)
- 当用户刷新实例化视图时，可能会发生死锁。[#35736](https://github.com/StarRocks/starrocks/pull/35736)
- 开启 DISTINCT 窗口运算符下推功能后，如果对窗口函数计算的列的复杂表达式执行 SELECT DISTINCT 操作，则报错。[#36357](https://github.com/StarRocks/starrocks/pull/36357)
- 如果源数据文件采用 ORC 格式并包含嵌套数组，则 BE 会崩溃。[#36127](https://github.com/StarRocks/starrocks/pull/36127)
- 某些与 S3 兼容的对象存储会返回重复文件，从而导致 BE 崩溃。[#36103](https://github.com/StarRocks/starrocks/pull/36103)

## 3.1.5

发布日期：2023年11月28日

### 新特性

- StarRocks 共享数据集群的 CN 节点现在支持数据导出。[#34018](https://github.com/StarRocks/starrocks/pull/34018)

### 改进

- 系统数据库 `INFORMATION_SCHEMA` 中的 `COLUMNS` 视图现在可以显示 ARRAY、MAP 和 STRUCT 列。[#33431](https://github.com/StarRocks/starrocks/pull/33431)
- 支持对使用 LZO 压缩并存储在 Hive 中的 Parquet、ORC 和 CSV 格式文件的查询。[#30923](https://github.com/StarRocks/starrocks/pull/30923)  [#30721](https://github.com/StarRocks/starrocks/pull/30721)
- 支持对自动分区表的指定分区进行更新。如果指定的分区不存在，则返回错误。[#34777](https://github.com/StarRocks/starrocks/pull/34777)
- 当对创建这些实例化视图的表和视图（包括与这些视图关联的其他表和实例化视图）执行交换、删除或架构更改操作时，支持自动刷新实例化视图。[#32829](https://github.com/StarRocks/starrocks/pull/32829)
- 优化部分位图相关操作的性能，包括：
  - 优化的嵌套循环连接。[#340804](https://github.com/StarRocks/starrocks/pull/34804)  [#35003](https://github.com/StarRocks/starrocks/pull/35003)
  - 优化 `bitmap_xor` 功能。[#34069](https://github.com/StarRocks/starrocks/pull/34069)
  - 支持写入时复制，以优化位图性能并减少内存消耗。[#34047](https://github.com/StarRocks/starrocks/pull/34047)

### Bug 修复

修复以下问题：

- 如果在 Broker Load 作业中指定了过滤条件，那么在某些情况下，BE 可能会在数据加载期间崩溃。[#29832](https://github.com/StarRocks/starrocks/pull/29832)
- 执行 SHOW GRANTS 时报告未知错误。[#30100](https://github.com/StarRocks/starrocks/pull/30100)
- 将数据加载到使用基于表达式的自动分区的表中时，可能会引发错误“错误：自运行时错误以来行创建分区失败：无法分析分区值”。[#33513](https://github.com/StarRocks/starrocks/pull/33513)

- 查询返回错误“get_applied_rowsets失败，平板更新处于错误状态：tablet:18849 压缩后实际行大小发生变化”。 [#33246](https://github.com/StarRocks/starrocks/pull/33246)
- 在 StarRocks 无共享集群中，对 Iceberg 或 Hive 表的查询可能导致 BE 崩溃。 [#34682](https://github.com/StarRocks/starrocks/pull/34682)
- 在 StarRocks 无共享集群中，如果在数据加载过程中自动创建了多个分区，则加载的数据可能偶尔写入不匹配的分区。 [#34731](https://github.com/StarRocks/starrocks/pull/34731)
- 长时间、频繁地将数据加载到启用了持久索引的主键表中可能导致 BE 崩溃。 [#33220](https://github.com/StarRocks/starrocks/pull/33220)
- 查询返回错误“Exception: java.lang.IllegalStateException: null”。 [#33535](https://github.com/StarRocks/starrocks/pull/33535)
- 执行`show proc '/current_queries';`查询时，同时开始执行查询时，BE 可能会崩溃。 [#34316](https://github.com/StarRocks/starrocks/pull/34316)
- 如果将大量数据加载到启用了持久索引的主键表中，则可能引发错误。 [#34352](https://github.com/StarRocks/starrocks/pull/34352)
- StarRocks 从 v2.4 或更早版本升级到后续版本后，压实分数可能意外上升。 [#34618](https://github.com/StarRocks/starrocks/pull/34618)
- 如果使用数据库驱动程序 MariaDB ODBC 查询`INFORMATION_SCHEMA`，则`Schemata`视图中返回的`CATALOG_NAME`列仅包含`null`值。 [#34627](https://github.com/StarRocks/starrocks/pull/34627)
- FE 由于加载异常数据而崩溃，无法重新启动。 [#34590](https://github.com/StarRocks/starrocks/pull/34590)
- 如果在流加载作业处于 **PREPARD** 状态时执行架构更改，则该作业要加载的部分源数据将丢失。 [#34381](https://github.com/StarRocks/starrocks/pull/34381)
- 在 HDFS 存储路径的末尾包含两个或多个斜杠(`/`)会导致从 HDFS 备份和恢复数据失败。 [#34601](https://github.com/StarRocks/starrocks/pull/34601)
- 将会话变量`enable_load_profile`设置为`true`会使流加载作业易于失败。 [#34544](https://github.com/StarRocks/starrocks/pull/34544)
- 在列模式下对主键表执行部分更新会导致表的某些平板电脑显示其副本之间的数据不一致。 [#34555](https://github.com/StarRocks/starrocks/pull/34555)
- 使用 ALTER TABLE 语句添加的`partition_live_number`属性不会生效。 [#34842](https://github.com/StarRocks/starrocks/pull/34842)
- FE 无法启动并报告错误“无法加载日志类型 118”。 [#34590](https://github.com/StarRocks/starrocks/pull/34590)
- 将 FE 参数`recover_with_empty_tablet`设置为`true`可能导致 FE 崩溃。 [#33071](https://github.com/StarRocks/starrocks/pull/33071)
- 复制操作重放失败可能导致 FE 崩溃。 [#32295](https://github.com/StarRocks/starrocks/pull/32295)

### 兼容性更改

#### 参数

- 新增了一个 FE 配置项[`enable_statistics_collect_profile`](../administration/FE_configuration.md#enable_statistics_collect_profile)，用于控制是否生成统计查询的配置文件。默认值为`false`。[#33815](https://github.com/StarRocks/starrocks/pull/33815)
- FE 配置项[`mysql_server_version`](../administration/FE_configuration.md#mysql_server_version)现在是可变的。新设置可以对当前会话生效，而无需重新启动 FE。[#34033](https://github.com/StarRocks/starrocks/pull/34033)
- 新增了一个 BE/CN 配置项[`update_compaction_ratio_threshold`](../administration/BE_configuration.md#update_compaction_ratio_threshold)，用于控制 StarRocks 共享数据集群中主键表的压实可以合并的最大数据比例。默认值为`0.5`。如果单个平板电脑变得过大，我们建议缩小此值。对于 StarRocks 无共享集群，主键表的压实可以合并的数据比例仍会自动调整。[#35129](https://github.com/StarRocks/starrocks/pull/35129)

#### 系统变量

- 添加了一个会话变量`cbo_decimal_cast_string_strict`，用于控制 CBO 如何将数据从 DECIMAL 类型转换为 STRING 类型。如果该变量设置为`true`，则以 v2.5.x 及之后版本内置的逻辑为准，系统实现严格转换（即系统截断生成的字符串，并根据刻度长度填充 0s）。如果此变量设置为`false`，则以 v2.5.x 之前版本内置的逻辑为准，系统处理所有有效数字以生成字符串。默认值为`true`。[#34208](https://github.com/StarRocks/starrocks/pull/34208)
- 添加了一个会话变量`cbo_eq_base_type`，用于指定用于 DECIMAL 类型数据和 STRING 类型数据之间的数据比较的数据类型。默认值为`VARCHAR`，也是`DECIMAL`有效值。[#34208](https://github.com/StarRocks/starrocks/pull/34208)
- 添加了会话变量`big_query_profile_second_threshold`。当会话变量[`enable_profile`](../reference/System_variable.md#enable_profile)设置为`false`并且查询所花费的时间超过该变量指定的阈值时`big_query_profile_second_threshold`，将为该查询生成配置文件。[#33825](https://github.com/StarRocks/starrocks/pull/33825)

## 3.1.4

发布日期：2023年11月2日

### 新功能

- 支持对共享数据 StarRocks 集群中创建的主键表进行排序。
- 支持使用 str2date 函数为异步物化视图指定分区表达式。这有助于对驻留在外部目录中并使用 STRING 类型数据作为其分区表达式的表上创建的异步实例化视图进行增量更新和查询重写。[#29923](https://github.com/StarRocks/starrocks/pull/29923) [#31964](https://github.com/StarRocks/starrocks/pull/31964)
- 添加了一个新的会话变量`enable_query_tablet_affinity`，用于控制是否将针对同一平板电脑的多个查询定向到固定副本。默认情况下，此会话变量设置为`false`。[#33049](https://github.com/StarRocks/starrocks/pull/33049)
- 支持设置资源组级查询队列，该队列由全局变量`enable_group_level_query_queue`（默认值：`false`）控制。当全局级别或资源组级别的资源消耗达到预定义的阈值时，新查询将放入队列中，并在全局级别资源消耗和资源组级别资源消耗都低于其阈值时运行。
  - 用户可以为每个资源组设置`concurrency_limit`，以限制每个 BE 允许的最大并发查询数。
  - 用户可以为每个资源组设置`max_cpu_cores`，以限制每个 BE 允许的最大 CPU 消耗。
- 为资源组分类器添加了两个参数`plan_cpu_cost_range`和`plan_mem_cost_range`。
  - `plan_cpu_cost_range`：系统估计的 CPU 消耗范围。默认值`NULL`表示不施加任何限制。
  - `plan_mem_cost_range`：系统估计的内存消耗范围。默认值`NULL`表示不施加任何限制。

### 改进

- 窗口函数 COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD 和 STDDEV_SAMP 现在支持 ORDER BY 子句和 Window 子句。[#30786](https://github.com/StarRocks/starrocks/pull/30786)
- 如果在对 DECIMAL 类型数据进行查询期间发生十进制溢出，则返回错误而不是 NULL。[#30419](https://github.com/StarRocks/starrocks/pull/30419)
- 查询队列中允许的并发查询数现在由领导者 FE 管理。当查询开始和结束时，每个跟随者 FE 都会通知领导者 FE。如果并发查询数达到全局级别或资源组级别`concurrency_limit`，则新查询将被拒绝或放入队列中。

### Bug 修复

修复以下问题：

- Spark 或 Flink 可能会由于内存使用统计不准确而报告数据读取错误。[#30702](https://github.com/StarRocks/starrocks/pull/30702)  [#30751](https://github.com/StarRocks/starrocks/pull/30751)
- 元数据缓存的内存使用统计信息不准确。[#31978](https://github.com/StarRocks/starrocks/pull/31978)
- 调用 libcurl 时 BE 崩溃。[#31667](https://github.com/StarRocks/starrocks/pull/31667)
- 刷新在 Hive 视图上创建的 StarRocks 物化视图时，会返回错误"java.lang.ClassCastException: com.starrocks.catalog.HiveView 无法转换为 com.starrocks.catalog.HiveMetaStoreTable"。 [#31004](https://github.com/StarRocks/starrocks/pull/31004)
- 如果 ORDER BY 子句包含聚合函数，则会返回错误"java.lang.IllegalStateException: null"。 [#30108](https://github.com/StarRocks/starrocks/pull/30108)
- 在共享数据的 StarRocks 集群中，表键的信息不会记录在 `information_schema.COLUMNS` 中。因此，使用 Flink Connector 加载数据时，无法执行 DELETE 操作。 [#31458](https://github.com/StarRocks/starrocks/pull/31458)
- 使用 Flink Connector 加载数据时，如果存在高并发加载作业，并且 HTTP 线程数和扫描线程数均达到上限，则加载作业会意外挂起。 [#32251](https://github.com/StarRocks/starrocks/pull/32251)
- 当添加仅几个字节的字段时，在数据更改完成之前执行 SELECT COUNT(*) 将返回一个错误，内容为"错误: 无效字段名"。 [#33243](https://github.com/StarRocks/starrocks/pull/33243)
- 启用查询缓存后，查询结果不正确。 [#32781](https://github.com/StarRocks/starrocks/pull/32781)
- 在哈希联接期间，查询失败，导致 BE 崩溃。 [#32219](https://github.com/StarRocks/starrocks/pull/32219)
- `BINARY` 或 `VARBINARY` 数据类型的 `DATA_TYPE` 和 `COLUMN_TYPE` 在 `information_schema.columns` 视图中显示为 `unknown`。 [#32678](https://github.com/StarRocks/starrocks/pull/32678)

### 行为更改

- 从 v3.1.4 开始，默认情况下为在新 StarRocks 集群中创建的主键表启用持久索引（不适用于将现有 StarRocks 集群从早期版本升级到 v3.1.4）。 [#33374](https://github.com/StarRocks/starrocks/pull/33374)
- 新增 FE 参数 `enable_sync_publish`，默认设置为 `true`。当此参数设置为 `true` 时，将数据加载到主键表的“发布”阶段仅在“应用”任务完成后返回执行结果。因此，在加载作业返回成功消息后，可以立即查询加载的数据。但是，将此参数设置为 `true` 可能会导致数据加载到主键表中需要更长的时间。（在添加此参数之前，“应用”任务与“发布”阶段是异步的。）[#27055](https://github.com/StarRocks/starrocks/pull/27055)

## 3.1.3

发布日期：2023年9月25日

### 新功能

- 在共享数据的 StarRocks 集群中创建的主键表支持将索引持久化到本地磁盘上，其方式与在无共享 StarRocks 集群中一样。
- 聚合函数 [group_concat](../sql-reference/sql-functions/string-functions/group_concat.md) 支持 DISTINCT 关键字和 ORDER BY 子句。 [#28778](https://github.com/StarRocks/starrocks/pull/28778)
- [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Kafka Connector](../loading/Kafka-connector-starrocks.md)、[Flink Connector](../loading/Flink-connector-starrocks.md) 和 [Spark Connector](../loading/Spark-connector-starrocks.md) 支持在主键表的列模式下进行部分更新。 [#28288](https://github.com/StarRocks/starrocks/pull/28288)
- 分区中的数据可以随时间自动冷却。（此功能不支持 [列表分区](../table_design/list_partitioning.md)。）[#29335](https://github.com/StarRocks/starrocks/pull/29335) [#29393](https://github.com/StarRocks/starrocks/pull/29393)

### 改进

现在，执行带有无效注释的 SQL 命令将返回与 MySQL 一致的结果。 [#30210](https://github.com/StarRocks/starrocks/pull/30210)

### Bug 修复

修复以下问题：

- 如果在要执行的 [DELETE](../sql-reference/sql-statements/data-types/BITMAP.md) 语句的 WHERE 子句中指定[了 BITMAP](../sql-reference/sql-statements/data-types/HLL.md) 或 [HLL](../sql-reference/sql-statements/data-manipulation/DELETE.md) 数据类型，则该语句无法正确执行。 [#28592](https://github.com/StarRocks/starrocks/pull/28592)
- 重启跟随 FE 后，CpuCores 统计信息不是最新的，导致查询性能下降。 #[28472](https://github.com/StarRocks/starrocks/pull/28472) [#30434](https://github.com/StarRocks/starrocks/pull/30434)
- to_bitmap（）[ 函数的执行成本 ](../sql-reference/sql-functions/bitmap-functions/to_bitmap.md) 计算不正确。因此，在重写实例化视图后，会为函数选择不适当的执行计划。[#29961](https://github.com/StarRocks/starrocks/pull/29961)
- 在共享数据架构的某些用例中，重启跟随者 FE 后，提交给跟随者 FE 的查询会返回一个错误，内容为“未找到后端节点。检查是否有任何后端节点关闭”。 [#28615](https://github.com/StarRocks/starrocks/pull/28615)
- 如果使用 ALTER TABLE[ 语句将数据连续加载到正在更改的表](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)中，则可能会引发错误“平板电脑处于错误状态”。 [#29364](https://github.com/StarRocks/starrocks/pull/29364)
-  使用命令`max_broker_load_job_concurrency`修改 FE 动态参数`ADMIN SET FRONTEND CONFIG`不生效。 #[29964](https://github.com/StarRocks/starrocks/pull/29964) [#29720](https://github.com/StarRocks/starrocks/pull/29720)
- 如果 date_diff（）[函数中的时间单位是常量，但日期不是常量](../sql-reference/sql-functions/date-time-functions/date_diff.md)，则 BE 会崩溃。[#29937](https://github.com/StarRocks/starrocks/issues/29937)
- 在共享数据架构中，开启异步加载后，自动分区不会生效。 [#29986](https://github.com/StarRocks/starrocks/issues/29986)
- 如果用户使用 CREATE TABLE LIKE[ 语句](../sql-reference/sql-statements/data-definition/CREATE_TABLE_LIKE.md)创建主键表，`Unexpected exception: Unknown properties: {persistent_index_type=LOCAL}`则会引发错误。 [#30255](https://github.com/StarRocks/starrocks/pull/30255)
- 恢复主键表会导致重启 BE 后元数据不一致。 [#30135](https://github.com/StarRocks/starrocks/pull/30135)
- 如果用户将数据加载到同时执行截断操作和查询的主键表中，则在某些情况下会抛出错误“java.lang.NullPointerException”。 [#30573](https://github.com/StarRocks/starrocks/pull/30573)
- 如果在实例化视图创建语句中指定了谓词表达式，则这些实例化视图的刷新结果不正确。 [#29904](https://github.com/StarRocks/starrocks/pull/29904)
- 用户将 StarRocks 集群升级到 v3.1.2 后，升级前创建的表的存储卷属性将重置为 `null`。[#30647](https://github.com/StarRocks/starrocks/pull/30647)
- 如果同时对平板电脑元数据执行检查点和还原，则某些平板电脑副本将丢失且无法检索。 [#30603](https://github.com/StarRocks/starrocks/pull/30603)
- 如果用户使用 CloudCanal 将数据加载到已设置但`NOT NULL`未指定默认值的表列中，则会抛出错误“Unsupported dataFormat value is : \N”。 [#30799](https://github.com/StarRocks/starrocks/pull/30799)

### 行为更改

- 使用 group_concat[ 函数时 ](../sql-reference/sql-functions/string-functions/group_concat.md) ，用户必须使用 SEPARATOR 关键字来声明分隔符。
- 控制 group_concat[`group_concat_max_len`](../reference/System_variable.md#group_concat_max_len) 函数返回的字符串的缺省最大长度的[会话变量的缺省值 ](../sql-reference/sql-functions/string-functions/group_concat.md) 从 unlimited 更改为 `1024`。

## 3.1.2

发布日期：2023年8月25日

### Bug 修复

修复以下问题：

- 如果用户指定默认情况下要连接的数据库，并且该用户仅对数据库中的表具有权限，但对数据库没有权限，则会引发错误，指出该用户对数据库没有权限。 [#29767](https://github.com/StarRocks/starrocks/pull/29767)
-  云原生表  `show_data`的 RESTful API 操作返回的值不正确。[#29473](https://github.com/StarRocks/starrocks/pull/29473)
- 如果在运行 array_agg（）[ 函数 ](../sql-reference/sql-functions/array-functions/array_agg.md) 时取消查询，则 BE 会崩溃。[#29400](https://github.com/StarRocks/starrocks/issues/29400)
-  `Default`BITMAP[ 或 ](../sql-reference/sql-statements/Administration/SHOW_FULL_COLUMNS.md)HLL 数据类型的[列的 ](../sql-reference/sql-statements/data-types/BITMAP.md)SHOW FULL COLUMNS[ 语句返回的字段值 ](../sql-reference/sql-statements/data-types/HLL.md) 不正确。 [#29510](https://github.com/StarRocks/starrocks/pull/29510)
- 如果查询中的 [array_map（）](../sql-reference/sql-functions/array-functions/array_map.md) 函数涉及多个表，则查询会因下推策略问题而失败。 [#29504](https://github.com/StarRocks/starrocks/pull/29504)
- 对 ORC 格式文件的查询失败，因为 Apache ORC 中的错误修复 ORC-1304 （[apache/orc#1299](https://github.com/apache/orc/pull/1299)） 未合并。 [#29804](https://github.com/StarRocks/starrocks/pull/29804)

### 行为更改

对于新部署的 StarRocks v3.1 集群，如果要运行 [SET CATALOG](../sql-reference/sql-statements/data-definition/SET_CATALOG.md) 切换到该目录，您必须具有目标外部目录的 USAGE 权限。您可以使用 [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) 授予所需的权限。

对于从早期版本升级的 v3.1 集群，您可以使用继承的权限运行 SET CATALOG。

## 3.1.1

发布日期：2023年8月18日

### 新功能

- 支持 Azure Blob 存储用于[共享数据集群](../deployment/shared_data/s3.md)。
- 支持[共享数据集群](../deployment/shared_data/s3.md)的列表分区。
- 支持聚合函数 [COVAR_SAMP](../sql-reference/sql-functions/aggregate-functions/covar_samp.md)、[COVAR_POP](../sql-reference/sql-functions/aggregate-functions/covar_pop.md) 和 [CORR](../sql-reference/sql-functions/aggregate-functions/corr.md)。
- 支持以下[窗口函数](../sql-reference/sql-functions/Window_function.md)：COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD 和 STDDEV_SAMP。

### 改进

支持对 WHERE 子句中的所有复合谓词和所有表达式进行隐式转换。您可以使用[会话变量](../reference/System_variable.md) `enable_strict_type` 启用或禁用隐式转换。此会话变量的默认值为 `false`。

### Bug 修复

已修复以下问题：

- 当数据加载到具有多个副本的表中时，如果表的某些分区为空，则会写入大量无效的日志记录。 [#28824](https://github.com/StarRocks/starrocks/issues/28824)
- 对平均行大小的不准确估计会导致主键表在列模式下的部分更新占用过大的内存。  [#27485](https://github.com/StarRocks/starrocks/pull/27485)
- 如果在处于错误状态的平板电脑上触发克隆操作，则磁盘使用率会增加。 [#28488](https://github.com/StarRocks/starrocks/pull/28488)
- 压缩会导致冷数据写入本地缓存。 [#28831](https://github.com/StarRocks/starrocks/pull/28831)

## 3.1.0

发布日期：2023年8月7日

### 新功能

#### 共享数据集群

- 添加了对主键表的支持，无法启用持久索引。
- 支持 [AUTO_INCREMENT](../sql-reference/sql-statements/auto_increment.md) 列属性，为每个数据行启用全局唯一 ID，从而简化数据管理。
- 支持[加载时自动创建分区，使用分区表达式定义分区规则，](../table_design/expression_partitioning.md)使分区创建更易用、更灵活。
- 支持[对共享数据 StarRocks 集群中的存储卷进行抽象](../deployment/shared_data/s3.md#use-your-shared-data-starrocks-cluster)，用户可以在其中配置存储位置和认证信息。用户可以在创建数据库或表时直接引用已有的存储卷，使身份验证配置更加容易。

#### 数据湖分析

- 支持访问在 Hive 目录中[的表上创建的视图](../data_source/catalog/hive_catalog.md)。
- 支持访问 Parquet 格式的 Iceberg v2 表。
- 支持 [下沉数据到 Parquet 格式的 Iceberg 表](../data_source/catalog/iceberg_catalog.md#sink-data-to-an-iceberg-table)。
- [预览]支持使用 Elasticsearch 目录[访问 Elasticsearch 中存储的数据](../data_source/catalog/elasticsearch_catalog.md)。这简化了 Elasticsearch 外部表的创建。
- [预览] 支持使用 Paimon 目录对 Apache Paimon 中存储的流数据进行分析[](../data_source/catalog/paimon_catalog.md)。

#### 存储引擎、数据引入和查询

- 将自动分区升级为 [表达式分区](../table_design/expression_partitioning.md)。用户只需在创建表时使用简单的分区表达式（时间函数表达式或列表达式）指定分区方式，StarRocks 在加载数据时会根据数据特征和分区表达式中定义的规则自动创建分区。这种分区创建方式适用于大多数场景，并且更加灵活和用户友好。
- 支持 [列表分区](../table_design/list_partitioning.md)。根据为特定列预定义的值列表对数据进行分区，这可以加快查询速度并更有效地管理明确分类的数据。
- 向数据库添加了一个名为的新表 `loads` `Information_schema` 。用户可以从表中查询 [Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 和 [Insert](../sql-reference/sql-statements/data-manipulation/INSERT.md) 作业的结果`loads`。
- 支持记录按 Stream Load[、](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)Broker[ Load 和 ](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)Spark Load[ 作业过滤掉的不合格数据行 ](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) 。用户可以 `log_rejected_record_num` 在其加载作业中使用该参数来指定可以记录的最大数据行数。
- 支持[随机分桶](../table_design/Data_distribution.md#how-to-choose-the-bucketing-columns)。有了这个功能，用户在创建表时就不需要配置桶列，StarRocks 会将加载到其中的数据随机分发到桶中。结合`BUCKETS` StarRocks 自 v2.5.7 起提供的自动设置 Bucket 数量（）功能，用户不再需要考虑 Bucket 配置，大大简化了建表语句。但是，在大数据和高性能要求较高的场景中，我们建议用户继续使用哈希分桶，因为这样他们就可以使用分桶修剪来加速查询。
- 支持在 INSERT INTO [中使用表函数 FILES（） ](../loading/InsertInto.md) 直接加载存储在 AWS S3 中的 Parquet 或 ORC 格式数据文件的数据。FILES（） 函数可以自动推断表架构，从而减轻了在数据加载前创建外部目录或文件外部表的需要，因此大大简化了数据加载过程。
- 支持 [生成的列](../sql-reference/sql-statements/generated_columns.md)。通过生成列功能，StarRocks 可以自动生成和存储列表达式的值，并自动重写查询，提高查询性能。
- 支持使用 Spark 连接器将数据从 Spark 加载到 StarRocks[](../loading/Spark-connector-starrocks.md)。与 [Spark Load](../loading/SparkLoad.md) 相比，Spark 连接器提供了更全面的功能。用户可以定义 Spark 作业来对数据执行 ETL 操作，Spark 连接器充当 Spark 作业中的接收器。
- 支持将数据加载到 [](../sql-reference/sql-statements/data-types/Map.md) MAP 和 STRUCT 数据类型的列中，并支持在 ARRAY、MAP 和 STRUCT 中嵌套快速十进制值。[ ](../sql-reference/sql-statements/data-types/STRUCT.md)

#### SQL 参考

- 添加了以下与存储卷相关的语句：[CREATE STORAGE VOLUME、ALTER STORAGE VOLUME、DROP STORAGE VOLUME、](../sql-reference/sql-statements/Administration/CREATE_STORAGE_VOLUME.md)SET DEFAULT[ STORAGE VOLUME、](../sql-reference/sql-statements/Administration/ALTER_STORAGE_VOLUME.md)DESC[ STORAGE  ](../sql-reference/sql-statements/Administration/DROP_STORAGE_VOLUME.md)VOLUME[](../sql-reference/sql-statements/Administration/SET_DEFAULT_STORAGE_VOLUME.md)[、 ](../sql-reference/sql-statements/Administration/DESC_STORAGE_VOLUME.md) [SHOW STORAGE VOLUMES](../sql-reference/sql-statements/Administration/SHOW_STORAGE_VOLUMES.md)。

- 支持使用 ALTER TABLE[ 更改表注释](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)。 [#21035](https://github.com/StarRocks/starrocks/pull/21035)

- 新增以下功能：

  - 结构函数： [struct （row）](../sql-reference/sql-functions/struct-functions/row.md)、 [named_struct](../sql-reference/sql-functions/struct-functions/named_struct.md)
  - 映射函数： [str_to_map](../sql-reference/sql-functions/string-functions/str_to_map.md)、 [map_concat](../sql-reference/sql-functions/map-functions/map_concat.md)、 [map_from_arrays](../sql-reference/sql-functions/map-functions/map_from_arrays.md)、 [element_at](../sql-reference/sql-functions/map-functions/element_at.md)、 [distinct_map_keys](../sql-reference/sql-functions/map-functions/distinct_map_keys.md)、 [cardinality](../sql-reference/sql-functions/map-functions/cardinality.md)
  - 高阶映射函数： [map_filter](../sql-reference/sql-functions/map-functions/map_filter.md)、 [map_apply](../sql-reference/sql-functions/map-functions/map_apply.md)、 [transform_keys](../sql-reference/sql-functions/map-functions/transform_keys.md)、 [transform_values](../sql-reference/sql-functions/map-functions/transform_values.md)
  - 数组函数：[array_agg](../sql-reference/sql-functions/array-functions/array_agg.md)支持`ORDER BY`、[array_generate](../sql-reference/sql-functions/array-functions/array_generate.md)、[element_at](../sql-reference/sql-functions/array-functions/element_at.md)[、cardinality](../sql-reference/sql-functions/array-functions/cardinality.md)
  - 高阶数组函数： [all_match](../sql-reference/sql-functions/array-functions/all_match.md)、 [any_match](../sql-reference/sql-functions/array-functions/any_match.md)
  - 聚合函数： [min_by](../sql-reference/sql-functions/aggregate-functions/min_by.md)、 [percentile_disc](../sql-reference/sql-functions/aggregate-functions/percentile_disc.md)
  - 表函数： [FILES](../sql-reference/sql-functions/table-functions/files.md)、 [generate_series](../sql-reference/sql-functions/table-functions/generate_series.md)
  - 日期函数: [next_day](../sql-reference/sql-functions/date-time-functions/next_day.md), [previous_day](../sql-reference/sql-functions/date-time-functions/previous_day.md), [last_day](../sql-reference/sql-functions/date-time-functions/last_day.md), [makedate](../sql-reference/sql-functions/date-time-functions/makedate.md), [date_diff](../sql-reference/sql-functions/date-time-functions/date_diff.md)
  - 位图函数: [bitmap_subset_limit](../sql-reference/sql-functions/bitmap-functions/bitmap_subset_limit.md), [bitmap_subset_in_range](../sql-reference/sql-functions/bitmap-functions/bitmap_subset_in_range.md)
  - 数学函数: [cosine_similarity](../sql-reference/sql-functions/math-functions/cos_similarity.md), [cosine_similarity_norm](../sql-reference/sql-functions/math-functions/cos_similarity_norm.md)

#### 权限和安全

新增了与存储卷相关的[权限项](../administration/privilege_item.md#storage-volume)和与外部目录相关的[权限项](../administration/privilege_item.md#catalog)，并支持使用[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)和[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md)来授予和撤销这些权限。

### 改进

#### 共享数据集群

优化了共享数据 StarRocks 集群中的数据缓存。优化后的数据缓存允许指定热数据的范围。它还可以防止对冷数据的查询占用本地磁盘缓存，从而确保对热数据的查询性能。

#### 物化视图

- 优化了异步物化视图的创建：
  - 支持随机分桶。如果用户未指定分桶列，StarRocks 默认采用随机分桶。
  - 支持使用`ORDER BY`来指定排序键。
  - 支持指定`colocate_group`、`storage_medium`和`storage_cooldown_time`等属性。
  - 支持使用会话变量。用户可以通过使用`properties("session.<variable_name>" = "<value>")`语法来灵活调整视图刷新策略。
  - 为所有异步物化视图启用了溢出功能，并默认实现了1小时的查询超时时长。
  - 支持基于视图创建物化视图。这使得在数据建模场景中更容易使用物化视图，因为用户可以根据需求灵活使用视图和基于视图的物化视图来实现分层建模。
- 优化了使用异步物化视图进行查询重写：
  - 支持过时重写，允许使用未在指定时间间隔内刷新的物化视图进行查询重写，而不管物化视图的基表是否已更新。用户可以在创建物化视图时使用`mv_rewrite_staleness_second`属性来指定时间间隔。
  - 支持针对在Hive目录表上创建的物化视图重写视图增量联接查询（必须定义主键和外键）。
  - 优化了重写包含联合操作的查询机制，并支持重写包含联接或函数（如COUNT DISTINCT和time_slice）的查询。
- 优化了异步物化视图的刷新：
  - 优化了在Hive目录表上创建的物化视图的刷新机制。StarRocks现在可以感知分区级别的数据变化，并在每次自动刷新时只刷新有数据变化的分区。
  - 支持使用`REFRESH MATERIALIZED VIEW WITH SYNC MODE`语法来同步调用物化视图刷新任务。
- 增强了异步物化视图的使用：
  - 支持使用`ALTER MATERIALIZED VIEW {ACTIVE | INACTIVE}`来启用或禁用物化视图。处于禁用状态（`INACTIVE`）的物化视图无法刷新或用于查询重写，但可以直接查询。
  - 支持使用`ALTER MATERIALIZED VIEW SWAP WITH`来交换两个物化视图。用户可以创建新的物化视图，然后与现有的物化视图执行原子交换，以实现对现有物化视图的架构更改。
- 优化了同步物化视图：
  - 支持使用SQL提示`[_SYNC_MV_]`直接查询同步物化视图，从而解决了极少数情况下无法正确重写某些查询的问题。
  - 支持更多表达式，如`CASE-WHEN`、`CAST`和数学运算，使物化视图适用于更多业务场景。

#### 数据湖分析

- 优化了Iceberg的元数据缓存和访问，以提高Iceberg数据查询性能。
- 优化了数据缓存，进一步提升了数据湖分析性能。

#### 存储引擎、数据引入和查询

- 宣布了[spill](../administration/spill_to_disk.md)功能的一般可用性，该功能支持将某些阻塞算子的中间计算结果溢出到磁盘。启用溢出功能后，当查询包含聚合、排序或连接算子时，StarRocks可以将算子的中间计算结果缓存到磁盘上，以减少内存消耗，从而最小化因内存限制导致的查询失败。
- 支持保留基数的联接修剪。如果用户维护大量以星型架构（例如SSB）或雪花架构（例如TCP-H）组织的表，但只查询其中的少量表，此功能有助于修剪不必要的表，以提高联接的性能。
- 支持列模式下的部分更新。用户可以在对主键表执行部分更新时启用列模式，适用于更新少量列但大量行的情况，并且可以将更新性能提高多达10倍。
- 优化了CBO的统计信息收集，减少了统计信息收集对数据引入的影响，并提高了统计信息收集的性能。
- 优化了合并算法，使排列场景下的整体性能提高了最多2倍。
- 优化了查询逻辑，减少了对数据库锁的依赖。
- 动态分区进一步支持将分区单元设置为年。[#28386](https://github.com/StarRocks/starrocks/pull/28386)

#### SQL参考

- 条件函数case、coalesce、if、ifnull和nullif支持ARRAY、MAP、STRUCT和JSON数据类型。
- 以下Array函数支持嵌套类型MAP、STRUCT和ARRAY：
  - array_agg
  - array_contains、array_contains_all、array_contains_any
  - array_slice，array_concat
  - array_length、array_append、array_remove、array_position
  - reverse、array_distinct、array_intersect、arrays_overlap
  - array_sortby
- 以下Array函数支持Fast Decimal数据类型：
  - array_agg
  - array_append、array_remove、array_position、array_contains
  - array_length
  - array_max、array_min、array_sum、array_avg
  - arrays_overlap、array_difference
  - array_slice、array_distinct、array_sort、reverse、array_intersect、array_concat
  - array_sortby、array_contains_all、array_contains_any

### Bug修复

修复了以下问题：

- 无法正确处理重新连接到Kafka进行例行加载作业的请求。[#23477](https://github.com/StarRocks/starrocks/issues/23477)
- 对于涉及多个表并包含`WHERE`子句的SQL查询，如果这些SQL查询具有相同的语义，但每个SQL查询中表的顺序不同，则其中一些SQL查询可能无法重写以受益于相关的物化视图。[#22875](https://github.com/StarRocks/starrocks/issues/22875)
- 对于包含`GROUP BY`子句的查询，将返回重复的记录。[#19640](https://github.com/StarRocks/starrocks/issues/19640)
- 调用lead（）或lag（）函数可能会导致BE崩溃。[#22945](https://github.com/StarRocks/starrocks/issues/22945)
- 基于在外部目录表上创建的物化视图重写部分分区查询失败。[#19011](https://github.com/StarRocks/starrocks/issues/19011)
- 无法正确解析包含反斜杠（`\`）和分号（`;`）的SQL语句。[#16552](https://github.com/StarRocks/starrocks/issues/16552)
- 如果删除了在表上创建的物化视图，则无法截断表。[#19802](https://github.com/StarRocks/starrocks/issues/19802)

### 行为更改

- 从共享数据StarRocks集群的建表语法中删除了`storage_cache_ttl`参数。现在，本地缓存中的数据将基于LRU算法进行逐出。
- 将BE配置项`disable_storage_page_cache`和`alter_tablet_worker_count`以及FE配置项`lake_compaction_max_tasks`从不可变参数更改为可变参数。
- 将BE配置项`block_cache_checksum_enable`的默认值从`true`更改为`false`。
- 将BE配置项`enable_new_load_on_memory_limit_exceeded`的默认值从`false`更改为`true`。
- 将FE配置项`max_running_txn_num_per_db`的默认值从`100`更改为`1000`。
- 将FE配置项`http_max_header_size`的默认值从`8192`更改为`32768`。
- 将FE配置项`tablet_create_timeout_second`的默认值从`1`更改为`10`。
- 将FE配置项`max_routine_load_task_num_per_be`的默认值从`5`更改为`16`，如果创建了大量的例行加载任务，将返回错误信息。
- 将FE配置项`quorom_publish_wait_time_ms`重命名为`quorum_publish_wait_time_ms`，将FE配置项`async_load_task_pool_size`重命名为`max_broker_load_job_concurrency`。
- 不再推荐使用BE配置项`routine_load_thread_pool_size`。现在，每个BE节点的例行加载线程池大小仅由FE配置项`max_routine_load_task_num_per_be`控制。
- 不推荐使用 BE 配置项 `txn_commit_rpc_timeout_ms` 和系统变量 `tx_visible_wait_timeout`。
- 已弃用 FE 配置项 `max_broker_concurrency` 和 `load_parallel_instance_num`。
- 已弃用 FE 配置项 `max_routine_load_job_num`。现在，StarRocks 根据 `max_routine_load_task_num_per_be` 参数动态推断每个 BE 节点支持的最大 Routine Load 任务数，并提供任务失败建议。
- CN 配置项 `thrift_port` 被重命名为 `be_port`。
- 添加了两个新的 Routine Load 作业属性，`task_consume_second` 和 `task_timeout_second`，用于控制例行加载作业中各个加载任务的最大数据消耗时间和超时持续时间，从而使作业调整更加灵活。如果用户在例行加载作业中没有指定这两个属性，则以 FE 配置项 `routine_load_task_consume_second` 和 `routine_load_task_timeout_second` 为准。
- 会话变量 `enable_resource_group` 已弃用，因为自 v3.1.0 起默认启用 [资源组](../administration/resource_group.md) 功能。
- 添加了两个新的保留关键字：COMPACTION 和 TEXT。