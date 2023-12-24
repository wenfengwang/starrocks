---
displayed_sidebar: English
---

# StarRocks 2.3 版本

## 2.3.18

发布日期 2023年10月11日

### Bug 修复

修复了以下问题：

- 第三方库 librdkafka 中的 bug 导致 Routine Load 作业的加载任务在数据加载过程中卡住，新创建的加载任务也无法执行。 [#28301](https://github.com/StarRocks/starrocks/pull/28301)
- 由于内存统计信息不准确，Spark 或 Flink 连接器可能无法导出数据。 [#31200](https://github.com/StarRocks/starrocks/pull/31200) [#30751](https://github.com/StarRocks/starrocks/pull/30751)
- 当流加载作业使用关键字 `if` 时，BE 崩溃。 [#31926](https://github.com/StarRocks/starrocks/pull/31926)
- 在将数据加载到分区的 StarRocks 外部表中时报错 `"get TableMeta failed from TNetworkAddress"`。 [#30466](https://github.com/StarRocks/starrocks/pull/30466)

## 2.3.17

发布日期 2023年9月4日

### Bug 修复

修复了以下问题：

- 例程加载作业无法使用数据。 [#29883](https://github.com/StarRocks/starrocks/issues/29883) [#18550](https://github.com/StarRocks/starrocks/pull/18550)

## 2.3.16

发布日期 2023年8月4日

### Bug 修复

修复了以下问题：

FE 内存泄漏，由阻塞的 LabelCleaner 线程引起。 #28311 [ ](https://github.com/StarRocks/starrocks/pull/28311)#28636[ ](https://github.com/StarRocks/starrocks/pull/28636)

## 2.3.15

发布日期 2023年7月31日

## 改进

- 优化 tablet 调度逻辑，防止 tablet 长时间挂起或 FE 在特定情况下崩溃。 #[21647](https://github.com/StarRocks/starrocks/pull/21647) [ #23062](https://github.com/StarRocks/starrocks/pull/23062) [#25785](https://github.com/StarRocks/starrocks/pull/25785)
- 优化 TabletChecker 调度逻辑，防止 checker 重复调度未修复的 Tablet。 [#27648](https://github.com/StarRocks/starrocks/pull/27648)
- 分区元数据记录 visibleTxnId，对应于平板电脑副本的可见版本。当副本的版本与其他版本不一致时，可以更轻松地跟踪创建此版本的事务。 [#27924](https://github.com/StarRocks/starrocks/pull/27924)

### Bug 修复

修复了以下问题：

- FE 中不正确的表级扫描统计信息会导致与表查询和加载相关的指标不准确。 [#28022](https://github.com/StarRocks/starrocks/pull/28022)
- 如果 Join 键是一个较大的 BINARY 列，则 BE 可能会崩溃。 [#25084](https://github.com/StarRocks/starrocks/pull/25084)
- 在某些情况下，聚合运算符可能会触发线程安全问题，从而导致 BE 崩溃。 [#26092](https://github.com/StarRocks/starrocks/pull/26092)
- 使用 RESTORE 恢复数据后，平板电脑的 BE 和 FE 版本号不一致[](../sql-reference/sql-statements/data-definition/RESTORE.md)。 [#26518](https://github.com/StarRocks/starrocks/pull/26518/files)
- 使用 RECOVER 恢复表后，无法自动创建分区[](../sql-reference/sql-statements/data-definition/RECOVER.md)。 [#26813](https://github.com/StarRocks/starrocks/pull/26813)
- 如果使用 INSERT INTO 加载的数据不符合质量要求，并且数据加载开启了严格模式，则加载事务会停滞在 Pending 状态，DDL 语句将挂起。 [#27140](https://github.com/StarRocks/starrocks/pull/27140)
-  如果启用了低基数优化，  `[42000][1064] Dict Decode failed, Dict can't take cover all key :0`则某些 INSERT 作业将返回。[#27395](https://github.com/StarRocks/starrocks/pull/27395)
- 在某些情况下，当未启用管道时，INSERT INTO SELECT 操作会超时。 [#26594](https://github.com/StarRocks/starrocks/pull/26594)
- 当查询条件为  且 in 的值 `WHERE partition_column < xxx` 仅精确到小时，而不是精确到分钟和秒`xxx`时，查询不返回任何数据，例如 `2023-7-21 22`。 [#27780](https://github.com/StarRocks/starrocks/pull/27780)

## 2.3.14

发布日期 2023年6月28日

### 改进

- 优化 CREATE TABLE 超时时返回的错误信息，增加参数调优提示。 [#24510](https://github.com/StarRocks/starrocks/pull/24510)
- 优化了累积大量 tablet 版本的主键表的内存使用。 [#20760](https://github.com/StarRocks/starrocks/pull/20760)
- StarRocks 外部表元数据的同步已更改为在数据加载期间进行。 [#24739](https://github.com/StarRocks/starrocks/pull/24739)
- 删除了 NetworkTime 对系统时钟的依赖性，以修复由于服务器之间的系统时钟不一致而导致的不正确的 NetworkTime。 [#24858](https://github.com/StarRocks/starrocks/pull/24858)

### Bug 修复

修复了以下问题：

- 当对频繁执行截断操作的小表应用低基数字典优化时，查询可能会遇到错误。 [#23185](https://github.com/StarRocks/starrocks/pull/23185)
- 当查询包含第一个子项为常量 NULL 的 UNION 的视图时，BE 可能会崩溃。 [#13792](https://github.com/StarRocks/starrocks/pull/13792)  
- 在某些情况下，基于位图索引的查询可能会返回错误。 [#23484](https://github.com/StarRocks/starrocks/pull/23484)
- 在BE中，将DOUBLE或FLOAT值四舍五入为DECIMAL值的结果与FE中的结果不一致。 [#23152](https://github.com/StarRocks/starrocks/pull/23152)
- 如果数据加载与架构更改同时发生，则架构更改有时可能会挂起。 [#23456](https://github.com/StarRocks/starrocks/pull/23456)
- 使用 Broker Load、Spark 连接器或 Flink 连接器将 Parquet 文件加载到 StarRocks 中时，可能会出现 BE OOM 问题。 [#25254](https://github.com/StarRocks/starrocks/pull/25254)
-  如果在 ORDER BY 子句中指定了常量，并且查询中存在 LIMIT 子句，则返回 `unknown error` 错误消息。[#25538](https://github.com/StarRocks/starrocks/pull/25538)

## 2.3.13

发布日期 2023年6月1日

### 改进

- 优化 INSERT INTO ...SELECT 由于值较小而过期 `thrift_server_max_worker_thread` 。 [排名#21964](https://github.com/StarRocks/starrocks/pull/21964)
- 减少了内存消耗，并优化了使用该函数的多表联接的性能 `bitmap_contains` 。 #20617 [ ](https://github.com/StarRocks/starrocks/pull/20617)#20653[ ](https://github.com/StarRocks/starrocks/pull/20653)

### Bug 修复

修复了以下错误：

- 截断分区失败，因为 TRUNCATE 操作对分区名称区分大小写。 [#21809](https://github.com/StarRocks/starrocks/pull/21809)
- 从 Parquet 文件加载 int96 时间戳数据会导致数据溢出。 [#22355](https://github.com/StarRocks/starrocks/issues/22355)
- 删除实例化视图后，停用 BE 失败。 [#22743](https://github.com/StarRocks/starrocks/issues/22743)
- 当查询的执行计划包含 Broadcast Join 后跟  Bucket Shuffle Join（如  `SELECT * FROM t1 JOIN [Broadcast] t2 ON t1.a = t2.b JOIN [Bucket] t3 ON t2.b = t3.c`）时，如果 Broadcast Join 中左侧表的 equijoin 键的数据在发送到 Bucket Shuffle Join 之前被删除，则 BE 可能会崩溃。 [#23227](https://github.com/StarRocks/starrocks/pull/23227)
- 当查询的执行计划包含 Cross Join 后跟 Hash Join，并且 Fragment 实例中 Hash Join 的右表为空时，返回结果可能不正确。 [#23877](https://github.com/StarRocks/starrocks/pull/23877)
- 由于为实例化视图创建临时分区失败，停用 BE 失败。 [#22745](https://github.com/StarRocks/starrocks/pull/22745)
- 如果 SQL 语句包含包含多个转义字符的 STRING 值，则无法解析该 SQL 语句。 [#23119](https://github.com/StarRocks/starrocks/issues/23119)
- 查询分区列中最大值的数据失败。 [#23153](https://github.com/StarRocks/starrocks/issues/23153)
- StarRocks 从 v2.4 回滚到 v2.3 后，加载作业失败。 [#23642](https://github.com/StarRocks/starrocks/pull/23642)
- 与列修剪和重用相关的问题。 [#16624](https://github.com/StarRocks/starrocks/issues/16624)

## 2.3.12

发布日期 2023年4月25日

### 改进

如果表达式的返回值可以转换为有效的布尔值，则支持隐式转换。 [# 21792](https://github.com/StarRocks/starrocks/pull/21792)

### Bug 修复

修复了以下错误：

- 如果在表级别授予用户LOAD_PRIV，则 `Access denied; you need (at least one of) the LOAD privilege(s) for this operation`  在加载作业失败时，事务回滚时将返回错误消息。 [# 21129](https://github.com/StarRocks/starrocks/issues/21129)
- 执行 ALTER SYSTEM DROP BACKEND 删除 BE 后，该 BE 上复制号设置为 2 的表副本无法修复。在此情况下，数据加载到这些表中失败。 [# 20681](https://github.com/StarRocks/starrocks/pull/20681)
- 当在 CREATE TABLE 中使用不支持的数据类型时，将返回 NPE。 [# 20999](https://github.com/StarRocks/starrocks/issues/20999)
- Broadcast Join的shortcircut逻辑异常，导致查询结果不正确。 [# 20952](https://github.com/StarRocks/starrocks/issues/20952)
- 使用实例化视图后，磁盘使用率可能会显著增加。 [# 20590](https://github.com/StarRocks/starrocks/pull/20590)
- Audit Loader 插件无法完全卸载。 [# 20468](https://github.com/StarRocks/starrocks/issues/20468)
- 结果中显示的行数 `INSERT INTO XXX SELECT` 可能与 的结果不匹配 `SELECT COUNT(*) FROM XXX`。 [# 20084](https://github.com/StarRocks/starrocks/issues/20084)

- 如果子查询使用窗口函数，而其父查询使用 GROUP BY 子句，则无法对查询结果进行聚合。 [#19725](https://github.com/StarRocks/starrocks/issues/19725)
- 启动 BE 后，BE 进程存在，但无法打开所有 BE 的端口。 [#19347](https://github.com/StarRocks/starrocks/pull/19347)
- 如果磁盘 I/O 过高，则主键表上的事务提交速度会变慢，因此对这些表的查询可能会返回错误"找不到后端"。 [#18835](https://github.com/StarRocks/starrocks/issues/18835)

## 2.3.11

发布日期：2023年3月28日

### 改进

- 执行包含许多表达式的复杂查询通常会生成大量的`ColumnRefOperators`。最初，StarRocks使用`BitSet`来存储`ColumnRefOperator::id`，这会消耗大量内存。为了减少内存使用，StarRocks现在使用`RoaringBitMap`来存储`ColumnRefOperator::id`。[#16499](https://github.com/StarRocks/starrocks/pull/16499)
- 引入了一种新的 I/O 调度策略，以降低大型查询对小型查询的性能影响。要启用新的 I/O 调度策略，请在**be.conf**中配置 BE 静态参数`pipeline_scan_queue_mode=1`，然后重新启动 BE。[#19009](https://github.com/StarRocks/starrocks/pull/19009)

### Bug 修复

修复了以下错误：

- 未正确回收过期数据的表占用了相当大的磁盘空间。[#19796](https://github.com/StarRocks/starrocks/pull/19796)
- 在以下场景中显示的错误信息不具备信息性：Broker Load 作业将 Parquet 文件加载到 StarRocks 中，并将`NULL`值加载到 NOT NULL 列中。[#19885](https://github.com/StarRocks/starrocks/pull/19885)
- 频繁创建大量临时分区来替换现有分区会导致 FE 节点内存泄漏和 Full GC。[#19283](https://github.com/StarRocks/starrocks/pull/19283)
- 对于共置表，可以使用类似语句手动指定副本状态为`bad``ADMIN SET REPLICA STATUS PROPERTIES ("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");`。如果 BE 数量小于或等于副本数量，则无法修复损坏的副本。[#19443](https://github.com/StarRocks/starrocks/pull/19443)
- 当请求`INSERT INTO SELECT`发送到 Follower FE 时，参数`parallel_fragment_exec_instance_num`不生效。[#18841](https://github.com/StarRocks/starrocks/pull/18841)
- 当使用运算符`<=>`将值与`NULL`值进行比较时，比较结果不正确。[#19210](https://github.com/StarRocks/starrocks/pull/19210)
- 当持续达到资源组的并发限制时，查询并发指标会缓慢下降。[#19363](https://github.com/StarRocks/starrocks/pull/19363)
- 高并发数据加载作业可能会导致错误`"get database read lock timeout, database=xxx"`。[#16748](https://github.com/StarRocks/starrocks/pull/16748) [#18992](https://github.com/StarRocks/starrocks/pull/18992)

## 2.3.10

发布日期：2023年3月9日

### 改进

优化了`storage_medium`的推理。当 BE 同时使用 SSD 和 HDD 作为存储设备时，如果指定了属性`storage_cooldown_time`，则 StarRocks将`storage_medium`设置为`SSD`。否则，StarRocks将`storage_medium`设置为`HDD`。[#18649](https://github.com/StarRocks/starrocks/pull/18649)

### Bug 修复

修复了以下错误：

- 如果查询数据湖中 Parquet 文件中的 ARRAY 数据，则查询可能会失败。[#17626](https://github.com/StarRocks/starrocks/pull/17626) [#17788](https://github.com/StarRocks/starrocks/pull/17788) [#18051](https://github.com/StarRocks/starrocks/pull/18051)
- 程序发起的 Stream Load 作业挂起，FE 没有收到程序发送的 HTTP 请求。[#18559](https://github.com/StarRocks/starrocks/pull/18559)
- 查询Elasticsearch外部表时，可能会报错。[#13727](https://github.com/StarRocks/starrocks/pull/13727)
- 如果表达式在初始化过程中遇到错误，则 BE 可能会崩溃。[#11396](https://github.com/StarRocks/starrocks/pull/11396)
- 如果 SQL 语句使用空数组文本，则查询可能会失败`[]`。[#18550](https://github.com/StarRocks/starrocks/pull/18550)
- StarRocks从 2.2 及以上版本升级至 2.3.9 及以上版本后，使用参数中指定的计算表达式创建例程加载作业时，可能会出现错误`COLUMN`。[#17856](https://github.com/StarRocks/starrocks/pull/17856)
- BE 重启后，加载作业挂起。[#18488](https://github.com/StarRocks/starrocks/pull/18488)
- 当 SELECT 语句在 WHERE 子句中使用 OR 运算符时，将扫描额外的分区。[#18610](https://github.com/StarRocks/starrocks/pull/18610)

## 2.3.9

发布日期：2023年2月20日

### Bug 修复

- 在架构更改期间，如果触发了 tablet 克隆，并且 tablet 副本所在的 BE 节点发生更改，则架构更改将失败。[#16948](https://github.com/StarRocks/starrocks/pull/16948)
- group_concat（）函数返回的字符串被截断。[#16948](https://github.com/StarRocks/starrocks/pull/16948)
- 使用Broker Load通过腾讯大数据套件（Tencent Big Data Suite，TBDS）从HDFS加载数据时`invalid hadoop.security.authentication.tbds.securekey`，出现错误，提示StarrRocks无法通过TBDS提供的认证信息访问HDFS。[#14125](https://github.com/StarRocks/starrocks/pull/14125) [#15693](https://github.com/StarRocks/starrocks/pull/15693)
- 在某些情况下，CBO 可能会使用不正确的逻辑来比较两个运算符是否等效。[#17227](https://github.com/StarRocks/starrocks/pull/17227) [#17199](https://github.com/StarRocks/starrocks/pull/17199)
- 当您连接到非 Leader FE 节点并发送 SQL 语句时，非 Leader FE 节点`USE <catalog_name>.<database_name>`会将 SQL 语句转发 `<catalog_name>`给 Leader FE 节点。结果，Leader FE 节点选择使用`default_catalog`但最终无法找到指定的数据库。[#17302](https://github.com/StarRocks/starrocks/pull/17302)

## 2.3.8

发布日期：2023年2月2日

### Bug 修复

修复了以下错误：

- 当大型查询完成后释放资源时，其他查询速度变慢的可能性很低。如果启用了资源组或大型查询意外结束，则更有可能出现此问题。[#16454](https://github.com/StarRocks/starrocks/pull/16454) [#16602](https://github.com/StarRocks/starrocks/pull/16602)
- 对于主键表，如果某个副本的元数据版本滞后，StarRocks会将其他副本中缺失的元数据增量克隆到该副本中。在此过程中，StarRocks会拉取大量版本的元数据，如果元数据版本累积过多且 GC不及时，可能会消耗过多的内存，导致 BE 遇到 OOM 异常。[#15935](https://github.com/StarRocks/starrocks/pull/15935)
- 如果 FE 偶尔向 BE 发送心跳，并且心跳连接超时，FE会认为 BE 不可用，导致 BE 上的事务失败。[#16386](https://github.com/StarRocks/starrocks/pull/16386)
- 使用 StarRocks 外部表在 StarRocks 集群之间加载数据时，如果源 StarRocks 集群为早期版本，目标 StarRocks 集群为更高版本（2.2.8 ~ 2.2.11、2.3.4 ~ 2.3.7、2.4.1 或 2.4.2），则数据加载失败。[#16173](https://github.com/StarRocks/starrocks/pull/16173)
- 当多个查询并发运行且内存使用率相对较高时，BE会崩溃。[#16047](https://github.com/StarRocks/starrocks/pull/16047)
- 当为表启用动态分区并动态删除某些分区时，如果执行 TRUNCATE TABLE，`NullPointerException`则返回错误。同时，如果将数据加载到表中，FE会崩溃，无法重启。[#16822](https://github.com/StarRocks/starrocks/pull/16822)

## 2.3.7

发布日期：2022年12月30日

### Bug 修复

修复了以下错误：

- StarRocks表中允许为NULL的列，在通过该表创建的视图中被错误地设置为NOT NULL。[#15749](https://github.com/StarRocks/starrocks/pull/15749)
- 当数据加载到StarRocks时，会生成一个新的Tablet版本。但是，FE可能尚未检测到新的平板电脑版本，并且仍然需要BE读取平板电脑的历史版本。如果垃圾回收机制删除了历史版本，则查询无法找到历史版本，并返回错误“未找到：get_applied_rowsets（版本xxxx）失败的tablet：xxx #version：x [xxxxxxx]”。[#15726](https://github.com/StarRocks/starrocks/pull/15726)
- FE在频繁加载数据时占用过多内存。[#15377](https://github.com/StarRocks/starrocks/pull/15377)
- 对于聚合查询和多表JOIN查询，统计信息收集不准确，执行计划中出现CROSS JOIN，导致查询延迟过长。[#12067](https://github.com/StarRocks/starrocks/pull/12067) [#14780](https://github.com/StarRocks/starrocks/pull/14780)

## 2.3.6

发布日期：2022年12月22日

### 改进

- Pipeline执行引擎支持INSERT INTO语句。要启用它，请将FE配置项设置为`enable_pipeline_load_for_insert` `true`。[#14723](https://github.com/StarRocks/starrocks/pull/14723)
- 用于主键表的Compaction内存减少。[#13861](https://github.com/StarRocks/starrocks/pull/13861) [#13862](https://github.com/StarRocks/starrocks/pull/13862)

### 行为变更
- 弃用了 FE 参数 `default_storage_medium`。表的存储介质由系统自动推断。 [#14394](https://github.com/StarRocks/starrocks/pull/14394)

### Bug 修复

以下错误已修复：

- 当启用资源组功能并且多个资源组同时运行查询时，BE 可能会挂起。 [#14905](https://github.com/StarRocks/starrocks/pull/14905)
- 使用 CREATE MATERIALIZED VIEW AS SELECT 创建物化视图时，如果 SELECT 子句不使用聚合函数，而是使用 GROUP BY（例如，`CREATE MATERIALIZED VIEW test_view AS SELECT a,b from test group by b,a order by a;`），则所有 BE 节点都会崩溃。 [#13743](https://github.com/StarRocks/starrocks/pull/13743)
- 如果在使用 INSERT INTO 频繁加载数据到主键表中进行数据更改后立即重新启动 BE，则 BE 的重启速度可能会非常慢。 [#15128](https://github.com/StarRocks/starrocks/pull/15128)
- 如果环境中只安装了 JRE，而没有安装 JDK，则 FE 重启后查询会失败。修复 bug 后，FE 无法在该环境中重启，返回错误 `JAVA_HOME can not be jre`。要成功重启 FE，您需要在环境中安装 JDK。 [#14332](https://github.com/StarRocks/starrocks/pull/14332)
- 查询导致 BE 崩溃。 [#14221](https://github.com/StarRocks/starrocks/pull/14221)
- `exec_mem_limit` 不能设置为表达式。 [#13647](https://github.com/StarRocks/starrocks/pull/13647)
- 无法基于子查询结果创建同步刷新的物化视图。 [#13507](https://github.com/StarRocks/starrocks/pull/13507)
- 刷新 Hive 外部表后，将删除列的注释。 [#13742](https://github.com/StarRocks/starrocks/pull/13742)
- 在相关的 JOIN 期间，右表先于左表进行处理，而右表非常大。如果在处理右侧表时对左侧表执行压缩，则 BE 节点会崩溃。 [#14070](https://github.com/StarRocks/starrocks/pull/14070)
- 如果 Parquet 文件列名称区分大小写，并且查询条件使用 Parquet 文件中的大写列名称，则查询不会返回任何结果。 #[13860](https://github.com/StarRocks/starrocks/pull/13860) [#14773](https://github.com/StarRocks/starrocks/pull/14773)
- 在批量加载期间，如果与 Broker 的连接数超过默认的最大连接数，则 Broker 将断开连接，并且加载作业将失败并显示错误消息 `list path error`。 [#13911](https://github.com/StarRocks/starrocks/pull/13911)
- 当 BE 负载过高时，资源组的指标 `starrocks_be_resource_group_running_queries` 可能不正确。 [#14043](https://github.com/StarRocks/starrocks/pull/14043)
- 如果查询语句使用 OUTER JOIN，可能会导致 BE 节点崩溃。 [#14840](https://github.com/StarRocks/starrocks/pull/14840)
- 使用 StarRocks 2.4 创建异步物化视图并回滚到 2.3 后，可能会发现 FE 启动失败。 [#14400](https://github.com/StarRocks/starrocks/pull/14400)
- 当主键表使用 delete_range，性能不佳时，可能会降低 RocksDB 的数据读取速度，导致 CPU 使用率过高。 [#15130](https://github.com/StarRocks/starrocks/pull/15130)

## 2.3.5

发布日期：2022年11月30日

### 改进

- Colocate Join 支持 Equi Join。 [#13546](https://github.com/StarRocks/starrocks/pull/13546)
- 修复频繁加载数据时，由于不断追加 WAL 记录导致主键索引文件过大的问题。 [#12862](https://github.com/StarRocks/starrocks/pull/12862)
- FE 会批量扫描所有 tablet，以便在 db.readLock 持有时间过长的情况下，FE 会以扫描间隔释放 db.readLock。 [#13070](https://github.com/StarRocks/starrocks/pull/13070)

### Bug 修复

以下错误已修复：

- 如果直接基于 UNION ALL 的结果创建视图，并且 UNION ALL 运算符的输入列包含 NULL 值，则视图的架构不正确，因为列的数据类型是 NULL_TYPE 的，而不是 UNION ALL 的输入列。 [#13917](https://github.com/StarRocks/starrocks/pull/13917)
- `SELECT * FROM ...` 和 `SELECT * FROM ... LIMIT ...` 的查询结果不一致。 [#13585](https://github.com/StarRocks/starrocks/pull/13585)
- 同步到 FE 的外部 Tablet 元数据可能会覆盖本地 Tablet 元数据，导致 Flink 数据加载失败。 [#12579](https://github.com/StarRocks/starrocks/pull/12579)
- 当运行时筛选器中的 null 筛选器处理文本常量时，BE 节点崩溃。 [#13526](https://github.com/StarRocks/starrocks/pull/13526)
- 执行 CTAS 时返回错误。 [#12388](https://github.com/StarRocks/starrocks/pull/12388)
- `ScanRows` 管道引擎在审核日志中收集的指标可能错误。 [#12185](https://github.com/StarRocks/starrocks/pull/12185)
- 查询压缩的 HIVE 数据时，查询结果不正确。 [#11546](https://github.com/StarRocks/starrocks/pull/11546)

## 2.3.4

发布日期：2022年11月10日

### 改进

- 该错误信息提供了 StarRocks 因运行中的例程加载作业数量超过限制而无法创建例程加载作业时的解决方案。 [#12204]( https://github.com/StarRocks/starrocks/pull/12204)
- 当 StarRocks 从 Hive 查询数据，解析 CSV 文件失败时，查询失败。 [#13013](https://github.com/StarRocks/starrocks/pull/13013)

### Bug 修复

以下错误已修复：

- 如果 HDFS 文件路径包含 `()`。 [#12660](https://github.com/StarRocks/starrocks/pull/12660)
- ORDER BY 的结果 ... LIMIT ... OFFSET 不正确。 [排名#9698](https://github.com/StarRocks/starrocks/issues/9698)
- StarRocks 在查询 ORC 文件时不区分大小写。 [#12724](https://github.com/StarRocks/starrocks/pull/12724)
- 当 RuntimeFilter 关闭而不调用 prepare 方法时，BE 可能会崩溃。 [#12906](https://github.com/StarRocks/starrocks/issues/12906)
- BE 可能会因内存泄漏而崩溃。 [#12906](https://github.com/StarRocks/starrocks/issues/12906)
- 添加新列并立即删除数据后，查询结果可能不正确。 [#12907](https://github.com/StarRocks/starrocks/pull/12907)
- BE 可能会因为对数据进行排序而崩溃。 [#11185](https://github.com/StarRocks/starrocks/pull/11185)

## 2.3.3

发布日期：2022年9月27日

### Bug 修复

以下错误已修复：

- 查询以文本文件形式存储的 Hive 外部表时，查询结果可能不准确。 [#11546](https://github.com/StarRocks/starrocks/pull/11546)
- 查询 Parquet 文件时不支持嵌套数组。 [#10983](https://github.com/StarRocks/starrocks/pull/10983)
- 如果从 StarRocks 和外部数据源读取数据的并发查询路由到同一资源组，或者查询从 StarRocks 和外部数据源读取数据，则查询或查询可能会超时。 [#10983](https://github.com/StarRocks/starrocks/pull/10983)
- 默认开启流水线执行引擎时，参数 parallel_fragment_exec_instance_num 变为 1。这将导致使用 INSERT INTO 的数据加载速度变慢。 [#11462](https://github.com/StarRocks/starrocks/pull/11462)
- 如果在初始化表达式时出现错误，则 BE 可能会崩溃。 [#11396](https://github.com/StarRocks/starrocks/pull/11396)
- 如果执行 ORDER BY LIMIT，可能会出现错误 heap-buffer-overflow。  [#11185](https://github.com/StarRocks/starrocks/pull/11185)
- 如果在此期间重新启动 Leader FE，则架构更改将失败。 [#11561](https://github.com/StarRocks/starrocks/pull/11561)

## 2.3.2

发布日期：2022年9月7日

### 新功能

- 支持延迟具体化，以加速对 Parquet 格式的外部表进行基于范围筛选器的查询。 [#9738](https://github.com/StarRocks/starrocks/pull/9738)
- 新增 SHOW AUTHENTICATION 语句，用于显示用户认证相关信息。 [#9996](https://github.com/StarRocks/starrocks/pull/9996)

### 改进

- 提供配置项，用于控制 StarRocks 是否以递归方式遍历从中查询数据的 Bucket Hive 表的所有数据文件。 [#10239](https://github.com/StarRocks/starrocks/pull/10239)
- 将资源组类型 `realtime` 重命名为 `short_query`。 [#10247](https://github.com/StarRocks/starrocks/pull/10247)
- StarRocks 默认不再区分 Hive 外部表中的大写字母和小写字母。 [#10187](https://github.com/StarRocks/starrocks/pull/10187)

### Bug 修复

修复了以下错误：

- 当 Elasticsearch 外部表被划分为多个分片时，对该表的查询可能会意外退出。 [#10369](https://github.com/StarRocks/starrocks/pull/10369)
- StarRocks 在子查询被重写为公用表表达式（CTEs）时会抛出错误。 [#10397](https://github.com/StarRocks/starrocks/pull/10397)
- StarRocks 在加载大量数据时会抛出错误。 [#10370](https://github.com/StarRocks/starrocks/issues/10370) [#10380](https://github.com/StarRocks/starrocks/issues/10380)
- 当为多个目录配置了相同的 Thrift 服务 IP 地址时，删除一个目录会使其他目录中的增量元数据更新失效。 [#10511](https://github.com/StarRocks/starrocks/pull/10511)
- BE 的内存消耗统计不准确。 [#9837](https://github.com/StarRocks/starrocks/pull/9837)
- StarRocks 对主键表的查询会抛出错误。 [#10811](https://github.com/StarRocks/starrocks/pull/10811)
- 即使您对逻辑视图具有 SELECT 权限，也不允许对这些视图进行查询。 [#10563](https://github.com/StarRocks/starrocks/pull/10563)
- StarRocks 对逻辑视图的命名没有限制。现在，逻辑视图需要遵循与表相同的命名约定。 [#10558](https://github.com/StarRocks/starrocks/pull/10558)

### 行为改变

- 添加 BE 配置 `max_length_for_bitmap_function`，默认值为 1000000，用于位图函数，添加 `max_length_for_to_base64`，默认值为 200000，用于 base64，以防止崩溃。 [#10851](https://github.com/StarRocks/starrocks/pull/10851)

## 2.3.1

发布日期：2022年8月22日

### 改进

- Broker Load 支持将 Parquet 文件中的 List 类型转换为非嵌套的 ARRAY 数据类型。 [#9150](https://github.com/StarRocks/starrocks/pull/9150)
- 优化 JSON 相关函数（json_query、get_json_string、get_json_int）的性能。 [#9623](https://github.com/StarRocks/starrocks/pull/9623)
- 优化 Hive、Iceberg 或 Hudi 查询时，如果 StarRocks 不支持查询列的数据类型，系统会抛出异常。 [#10139](https://github.com/StarRocks/starrocks/pull/10139)
- 降低资源组调度时延，优化资源隔离性能。 [#10122](https://github.com/StarRocks/starrocks/pull/10122)

### Bug 修复

修复了以下错误：

- 由于运算符 `limit` 的下推不正确，Elasticsearch 外部表的查询返回了错误的结果。 [#9952](https://github.com/StarRocks/starrocks/pull/9952)
- 使用运算符 `limit` 时，对 Oracle 外部表的查询失败。 [#9542](https://github.com/StarRocks/starrocks/pull/9542)
- 当所有 Kafka 代理在例程加载期间停止时，BE 将被阻止。 [#9935](https://github.com/StarRocks/starrocks/pull/9935)
- BE 在查询数据类型与相应外部表的数据类型不匹配的 Parquet 文件时崩溃。 [#10107](https://github.com/StarRocks/starrocks/pull/10107)
- 查询超时，因为外部表的扫描范围为空。 [#10091](https://github.com/StarRocks/starrocks/pull/10091)
- 当子查询中包含 ORDER BY 子句时，系统会引发异常。 [#10180](https://github.com/StarRocks/starrocks/pull/10180)
- 异步重新加载 Hive 元数据时，Hive 元存储挂起。 [#10132](https://github.com/StarRocks/starrocks/pull/10132)

## 2.3.0

发布日期：2022年7月29日

### 新功能

- 主键表支持完整的 DELETE WHERE 语法。有关详细信息，请参阅 [DELETE](../sql-reference/sql-statements/data-manipulation/DELETE.md#delete-data-by-primary-key)。
- 主键表支持持久性主键索引。您可以选择将主键索引保留在磁盘上而不是内存中，从而显著减少内存使用量。有关详细信息，请参阅[主键表](../table_design/table_types/primary_key_table.md)。
- 全局字典可以在实时数据摄取过程中更新，优化查询性能，为字符串数据提供 2 倍的查询性能。
- CREATE TABLE AS SELECT 语句可以异步执行。有关详细信息，请参阅 [CREATE TABLE AS SELECT](../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)。
- 支持以下与资源组相关的功能：
  - 监控资源组：您可以在审计日志中查看查询的资源组，并通过 API 获取资源组的指标。有关详细信息，请参阅[监控和警报](../administration/Monitor_and_Alert.md#monitor-and-alerting)。
  - 限制对 CPU、内存和 I/O 资源的大型查询的消耗：您可以根据分类器或通过配置会话变量将查询路由到特定资源组。有关详细信息，请参阅 [资源组](../administration/resource_group.md)。
- JDBC 外部表可以方便地查询 Oracle、PostgreSQL、MySQL、SQLServer、ClickHouse 等数据库中的数据。StarRocks 还支持谓词下推，提升查询性能。有关更多信息，请参阅 [与 JDBC 兼容的数据库的外部表](../data_source/External_table.md#external-table-for-a-JDBC-compatible-database)。
- [预览] 发布了新的数据源连接器框架以支持外部目录。您可以使用外部目录直接访问和查询 Hive 数据，而无需创建外部表。有关详细信息，请参阅 [使用目录管理内部和外部数据](../data_source/catalog/query_external_data.md)。
- 新增以下功能：
  - [window_funnel](../sql-reference/sql-functions/aggregate-functions/window_funnel.md)
  - [ntile](../sql-reference/sql-functions/Window_function.md)
  - [bitmap_union_count](../sql-reference/sql-functions/bitmap-functions/bitmap_union_count.md)、 [base64_to_bitmap](../sql-reference/sql-functions/bitmap-functions/base64_to_bitmap.md) [array_to_bitmap](../sql-reference/sql-functions/array-functions/array_to_bitmap.md)
  - [week](../sql-reference/sql-functions/date-time-functions/week.md)， [time_slice](../sql-reference/sql-functions/date-time-functions/time_slice.md)

### 改进

- 压缩机制可以更快地合并大量元数据。这样可以防止在频繁更新数据后不久发生元数据压缩和磁盘使用过多。
- 优化加载 Parquet 文件和压缩文件的性能。
- 优化物化视图创建机制。优化后，物化视图的创建速度比以前快 10 倍。
- 优化以下算子性能：
  - TopN 和排序运算符
  - 当包含函数的等效比较运算符向下推送到扫描运算符时，可以使用这些运算符。
- 优化 Apache Hive™ 外部表。
  - 当 Apache Hive™ 表以 Parquet、ORC 或 CSV 格式存储时，当您对对应的 Hive 外部表执行 REFRESH 语句时，Hive™ 上 ADD COLUMN 或 REPLACE COLUMN 导致的 Schema 更改可以同步到 StarRocks。有关详细信息，请参阅 [Hive 外部表](../data_source/External_table.md#hive-external-table)。
  - `hive.metastore.uris` 可以针对 Hive 资源进行修改。有关详细信息，请参阅 [ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md)。
- 优化 Apache Iceberg 外表性能。自定义目录可用于创建 Iceberg 资源。有关更多信息，请参阅 [Apache Iceberg 外部表](../data_source/External_table.md#apache-iceberg-external-table)。
- 优化 Elasticsearch 外部表性能。可以禁用嗅探 Elasticsearch 集群中数据节点的地址。有关更多信息，请参阅 [Elasticsearch 外部表](../data_source/External_table.md#elasticsearch-external-table)。
- 当 sum（） 函数接受数字字符串时，它会隐式转换数字字符串。
- year（）、month（） 和 day（） 函数支持 DATE 数据类型。

### Bug 修复

修复了以下 bug：

- 由于平板电脑数量过多，CPU 使用率激增。
- 导致出现“无法准备平板电脑阅读器”的问题。
- FE 无法重新启动。[#5642](https://github.com/StarRocks/starrocks/issues/5642 )  [排名#4969](https://github.com/StarRocks/starrocks/issues/4969 )  [#5580](https://github.com/StarRocks/starrocks/issues/5580)
- 当 CTAS 语句包含 JSON 函数时，该语句无法成功运行。 [#6498](https://github.com/StarRocks/starrocks/issues/6498)

### 其他

- StarGo 是一款集群管理工具，支持集群的部署、启动、升级、回滚，支持多个集群的管理。更多信息，请参见 [使用 StarGo 部署 StarRocks](../administration/stargo.md)。
- 当您将 StarRocks 升级到 2.3 版本或部署 StarRocks 时，默认开启流水线引擎。流水线引擎可以提升简单查询在高并发场景和复杂查询中的性能。如果您在使用 StarRocks 2.3 时检测到明显的性能下降，可以通过执行 `SET GLOBAL` 设置为 的语句 `enable_pipeline_engine` 来禁用流水线引擎`false`。
- [SHOW GRANTS](../sql-reference/sql-statements/account-management/SHOW_GRANTS.md) 语句与 MySQL 语法兼容，并以 GRANT 语句的形式显示分配给用户的权限。

- 建议将memory_limitation_per_thread_for_schema_change（BE配置项）的默认值设置为2 GB，当数据量超过此限制时，数据将写入磁盘。因此，如果您之前将此参数设置为较大的值，建议将其调整为2 GB，否则结构更改任务可能会占用大量内存。

### 升级注意事项

要回滚到升级前使用的上一个版本，需要在每个FE的**fe.conf**文件中添加`ignore_unknown_log_id`参数，并将该参数设置为`true`。由于StarRocks v2.2.0中添加了新类型的日志，因此此参数是必需的。如果不添加该参数，将无法回滚到先前的版本。我们建议在创建检查点后，将每个FE的**fe.conf**文件中的`ignore_unknown_log_id`参数设置为`false`。然后，重新启动FE以恢复先前的配置。