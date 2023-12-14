---
displayed_sidebar: "Chinese"
---

# StarRocks版本2.3

## 2.3.18

发布日期：2023年10月11日

### Bug修复

修复了以下问题：

- 第三方库librdkafka的bug导致常规加载作业的加载任务在数据加载过程中被卡住，新创建的加载任务也无法执行。[#28301](https://github.com/StarRocks/starrocks/pull/28301)
- 由于内存统计不准确，Spark或Flink连接器可能无法导出数据。[#31200](https://github.com/StarRocks/starrocks/pull/31200) [#30751](https://github.com/StarRocks/starrocks/pull/30751)
- 当流加载作业使用关键词`if`时，BE会崩溃。[#31926](https://github.com/StarRocks/starrocks/pull/31926)
- 在加载数据到分区StarRocks外部表时，报告错误`"get TableMeta failed from TNetworkAddress"`。[#30466](https://github.com/StarRocks/starrocks/pull/30466)

## 2.3.17

发布日期：2023年9月4日

### Bug修复

修复了以下问题：

- 常规加载作业未能消耗数据。[#29883](https://github.com/StarRocks/starrocks/issues/29883) [#18550](https://github.com/StarRocks/starrocks/pull/18550)

## 2.3.16

发布日期：2023年8月4日

### Bug修复

修复了以下问题：

由于被阻塞的LabelCleaner线程导致的FE内存泄漏。[#28311](https://github.com/StarRocks/starrocks/pull/28311) [#28636](https://github.com/StarRocks/starrocks/pull/28636)

## 2.3.15

发布日期：2023年7月31日

## 改进

- 优化了Tablet调度逻辑，以防止Tablet在某些情况下长时间保持挂起或FE崩溃。[#21647](https://github.com/StarRocks/starrocks/pull/21647) [#23062](https://github.com/StarRocks/starrocks/pull/23062) [#25785](https://github.com/StarRocks/starrocks/pull/25785)
- 优化了TabletChecker的调度逻辑，以防止Checker反复调度未修复的Tablet。[#27648](https://github.com/StarRocks/starrocks/pull/27648)
- 分区元数据记录了visibleTxnId，它对应于Tablet副本的可见版本。当一个副本的版本与其他副本不一致时，更容易跟踪创建该版本的事务。[#27924](https://github.com/StarRocks/starrocks/pull/27924)

### Bug修复

修复了以下问题：

- FEs中表级扫描统计信息不正确，导致与表查询和加载相关的度量不准确。[#28022](https://github.com/StarRocks/starrocks/pull/28022)
- 如果Join键是大型BINARY列，则BE可能会崩溃。[#25084](https://github.com/StarRocks/starrocks/pull/25084)
- 在某些情况下，聚合操作符可能会触发线程安全问题，导致BE崩溃。[#26092](https://github.com/StarRocks/starrocks/pull/26092)
- 使用[RESTORE](../sql-reference/sql-statements/data-definition/RESTORE.md)恢复数据后，BE和FE之间的Tablet版本号不一致。[#26518](https://github.com/StarRocks/starrocks/pull/26518/files)
- 使用[RECOVER](../sql-reference/sql-statements/data-definition/RECOVER.md)恢复表后，无法自动创建分区。[#26813](https://github.com/StarRocks/starrocks/pull/26813)
- 如果使用INSERT INTO加载不符合质量要求且启用了数据加载的严格模式，加载事务将处于挂起状态，并且DDL语句将挂起。[#27140](https://github.com/StarRocks/starrocks/pull/27140)
- 如果启用了低基数优化，某些INSERT作业会返回`[42000][1064] Dict Decode failed, Dict can't take cover all key :0`。[#27395](https://github.com/StarRocks/starrocks/pull/27395)
- 在某些情况下，当Pipeline未启用时，INSERT INTO SELECT操作会超时。[#26594](https://github.com/StarRocks/starrocks/pull/26594)
- 当查询条件为`WHERE partition_column < xxx`且`xxx`中的值仅精确到小时而不是分钟和秒时（例如，`2023-7-21 22`），查询将不返回任何数据。[#27780](https://github.com/StarRocks/starrocks/pull/27780)

## 2.3.14

发布日期：2023年6月28日

### 改进

- 当CREATE TABLE超时时，优化了返回的错误消息，并添加了参数调整提示。[#24510](https://github.com/StarRocks/starrocks/pull/24510)
- 优化了累积Tablet版本数量较大的主键表的内存使用。[#20760](https://github.com/StarRocks/starrocks/pull/20760)
- StarRocks外部表元数据的同步已更改为在数据加载期间发生。[#24739](https://github.com/StarRocks/starrocks/pull/24739)
- 删除了NetworkTime对系统时钟的依赖，以修复由服务器之间系统时钟不一致引起的不正确NetworkTime。[#24858](https://github.com/StarRocks/starrocks/pull/24858)

### Bug修复

修复了以下问题：

- 对于经常进行TRUNCATE操作的小表应用低基数字典优化时，查询可能会遇到错误。[#23185](https://github.com/StarRocks/starrocks/pull/23185)
- 当查询包含包含常量NULL的UNION的视图时，BE可能会崩溃。[#13792](https://github.com/StarRocks/starrocks/pull/13792)  
- 在某些情况下，基于位图索引的查询可能会返回错误。[#23484](https://github.com/StarRocks/starrocks/pull/23484)
- 在BE中，将DOUBLE或FLOAT值四舍五入为DECIMAL值的结果与FE中的结果不一致。[#23152](https://github.com/StarRocks/starrocks/pull/23152)
- 如果同时进行数据加载和模式更改，有时模式更改可能会挂起。[#23456](https://github.com/StarRocks/starrocks/pull/23456)
- 当使用Broker Load、Spark连接器或Flink连接器将Parquet文件加载到StarRocks时，可能会导致BE OOM问题。[#25254](https://github.com/StarRocks/starrocks/pull/25254)
- 当在ORDER BY子句中指定常量并且查询中存在LIMIT子句时，会返回错误消息`unknown error`。[#25538](https://github.com/StarRocks/starrocks/pull/25538)

## 2.3.13

发布日期：2023年6月1日

### 改进

- 优化了在INSERT INTO ... SELECT由于`thrift_server_max_worker_thread`值较小而过期时报告的错误消息。[#21964](https://github.com/StarRocks/starrocks/pull/21964)
- 减少了多表连接使用`bitmap_contains`函数的内存消耗，并优化了性能。[#20617](https://github.com/StarRocks/starrocks/pull/20617) [#20653](https://github.com/StarRocks/starrocks/pull/20653)

### Bug修复

修复了以下问题：

- 由于TRUNCATE操作对分区名称区分大小写敏感，截断分区失败。[#21809](https://github.com/StarRocks/starrocks/pull/21809)
- 从Parquet文件加载int96时间戳数据会导致数据溢出。[#22355](https://github.com/StarRocks/starrocks/issues/22355)
- 删除物化视图后，取消服务BE失败。[#22743](https://github.com/StarRocks/starrocks/issues/22743)
- 如果查询的执行计划包括广播连接后跟随桶洗牌连接（例如`SELECT * FROM t1 JOIN [Broadcast] t2 ON t1.a = t2.b JOIN [Bucket] t3 ON t2.b = t3.c`），并且在数据发送到桶洗牌连接之前，左表的等连接键数据已被删除，则BE可能会崩溃。[#23227](https://github.com/StarRocks/starrocks/pull/23227)
- 当查询的执行计划包括交叉连接后跟哈希连接，并且哈希连接中的右表在片段实例内为空时，返回的结果可能不正确。[#23877](https://github.com/StarRocks/starrocks/pull/23877)
- 由于在物化视图上创建临时分区失败，因此停止BE会失败。[#22745](https://github.com/StarRocks/starrocks/pull/22745)
- 如果SQL语句包含具有多个转义字符的STRING值，则无法解析SQL语句。[#23119](https://github.com/StarRocks/starrocks/issues/23119)
- 查询分区列中的最大值的数据将失败。[#23153](https://github.com/StarRocks/starrocks/issues/23153)
- StarRocks从v2.4回退到v2.3后，加载作业失败。[#23642](https://github.com/StarRocks/starrocks/pull/23642)
- 与列修剪和重用相关的问题。[#16624](https://github.com/StarRocks/starrocks/issues/16624)

## 2.3.12

发布日期：2023年4月25日

### 改进

支持隐式转换，如果表达式的返回值可以转换为有效的布尔值。[# 21792](https://github.com/StarRocks/starrocks/pull/21792)

### 缺陷修复

以下缺陷已修复：

- 如果用户在表级别授予LOAD_PRIV权限，则在加载作业失败时，事务回滚时返回错误消息 `Access denied; you need (at least one of) the LOAD privilege(s) for this operation`。[# 21129](https://github.com/StarRocks/starrocks/issues/21129)
- 执行ALTER SYSTEM DROP BACKEND以删除BE时，设置了该BE上复制数为2的表的副本无法修复。在这种情况下，会导致这些表的数据加载失败。[# 20681](https://github.com/StarRocks/starrocks/pull/20681)
- 在CREATE TABLE中使用不受支持的数据类型时返回NPE。[# 20999](https://github.com/StarRocks/starrocks/issues/20999)
- 广播连接的shortcircut逻辑异常，导致查询结果不正确。[# 20952](https://github.com/StarRocks/starrocks/issues/20952)
- 在使用物化视图后，磁盘使用量可能大幅增加。[# 20590](https://github.com/StarRocks/starrocks/pull/20590)
- 无法完全卸载审计加载器插件。[# 20468](https://github.com/StarRocks/starrocks/issues/20468)
- `INSERT INTO XXX SELECT`结果中显示的行数可能与 `SELECT COUNT(*) FROM XXX`的结果不匹配。[# 20084](https://github.com/StarRocks/starrocks/issues/20084)
- 如果子查询使用窗口函数，其父查询使用GROUP BY子句，则无法对查询结果进行聚合。[# 19725](https://github.com/StarRocks/starrocks/issues/19725)
- 当BE启动时，BE进程存在，但是所有BE的端口无法打开。[# 19347](https://github.com/StarRocks/starrocks/pull/19347)
- 如果磁盘I/O过高，则对主键表的事务提交速度较慢，因此对这些表的查询可能会返回错误 "backend not found"。[# 18835](https://github.com/StarRocks/starrocks/issues/18835)

## 2.3.11

发布日期：2023年3月28日

### 改进

- 执行包含许多表达式的复杂查询通常会生成大量的`ColumnRefOperators`。最初，StarRocks使用`BitSet`来存储`ColumnRefOperator::id`，这会消耗大量内存。为减少内存使用，StarRocks现在使用`RoaringBitMap`来存储`ColumnRefOperator::id`。[#16499](https://github.com/StarRocks/starrocks/pull/16499)
- 引入了新的I/O调度策略，以减少大型查询对小型查询的性能影响。要启用新的I/O调度策略，在**be.conf**中配置BE静态参数`pipeline_scan_queue_mode=1`，然后重新启动BE。[#19009](https://github.com/StarRocks/starrocks/pull/19009)

### 缺陷修复

以下缺陷已修复：

- 未正确回收过期数据的表占用磁盘空间的比例较大。[#19796](https://github.com/StarRocks/starrocks/pull/19796)
- 在以下场景显示的错误消息不具备信息化：Broker加载作业将Parquet文件加载到StarRocks，且向NOT NULL列加载了`NULL`值。[#19885](https://github.com/StarRocks/starrocks/pull/19885)
- 频繁创建大量临时分区以替换现有分区导致在FE节点上内存泄漏和Full GC。[#19283](https://github.com/StarRocks/starrocks/pull/19283)
- 对于Colocation表，可以使用类似 `ADMIN SET REPLICA STATUS PROPERTIES ("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");` 这样的语句手动指定副本状态为 `bad`。如果BE的数量小于或等于副本的数量，则无法修复损坏的副本。[#19443](https://github.com/StarRocks/starrocks/pull/19443)
- 当向Follower FE发送`INSERT INTO SELECT`请求时，参数`parallel_fragment_exec_instance_num`不会生效。[#18841](https://github.com/StarRocks/starrocks/pull/18841)
- 使用操作符`<=>`将值与`NULL`值进行比较时，比较结果不正确。[#19210](https://github.com/StarRocks/starrocks/pull/19210)
- 并发限制的资源组不断达到并发限制时，查询并发度指标下降速度缓慢。[#19363](https://github.com/StarRocks/starrocks/pull/19363)
- 高并发数据加载作业可能导致错误 `"get database read lock timeout, database=xxx"`。[#16748](https://github.com/StarRocks/starrocks/pull/16748) [#18992](https://github.com/StarRocks/starrocks/pull/18992)

## 2.3.10

发布日期：2023年3月9日

### 改进

优化了`storage_medium`的推断。当BE同时使用SSD和HDD作为存储设备时，如果指定了`storage_cooldown_time`属性，则StarRocks将`storage_medium`设置为`SSD`。否则，StarRocks将`storage_medium`设置为`HDD`。[#18649](https://github.com/StarRocks/starrocks/pull/18649)

### 缺陷修复

以下缺陷已修复：

- 如果查询数据湖中Parquet文件的ARRAY数据可能会失败。[#17626](https://github.com/StarRocks/starrocks/pull/17626) [#17788](https://github.com/StarRocks/starrocks/pull/17788) [#18051](https://github.com/StarRocks/starrocks/pull/18051)
- 程序发起的Stream Load作业挂起，FE未收到程序发送的HTTP请求。[#18559](https://github.com/StarRocks/starrocks/pull/18559)
- 查询Elasticsearch外部表可能会出现错误。[#13727](https://github.com/StarRocks/starrocks/pull/13727)
- 如果在初始化期间表达式遇到错误，BE可能会崩溃。[#11396](https://github.com/StarRocks/starrocks/pull/11396)
- 如果SQL语句使用空数组字面量`[]`，查询可能会失败。[#18550](https://github.com/StarRocks/starrocks/pull/18550)
- 从版本2.2及更高版本升级到版本2.3.9及更高版本后，使用`COLUMN`参数指定计算表达式创建Routine Load作业时可能出现错误`No match for <expr> with operand types xxx and xxx`。[#17856](https://github.com/StarRocks/starrocks/pull/17856)
- BE重新启动后，加载作业挂起。[#18488](https://github.com/StarRocks/starrocks/pull/18488)
- 当SELECT语句在WHERE子句中使用OR运算符时，会扫描额外的分区。[#18610](https://github.com/StarRocks/starrocks/pull/18610)

## 2.3.9

发布日期：2023年2月20日

### 缺陷修复

- 在模式更改期间，如果触发了平板克隆并且平板复制品所在的BE节点发生了变化，则模式更改将失败。[#16948](https://github.com/StarRocks/starrocks/pull/16948)
- group_concat()函数返回的字符串被截断。[#16948](https://github.com/StarRocks/starrocks/pull/16948)
- 当您使用经纪人加载通过腾讯大数据套件（TBDS）从HDFS加载数据时，出现错误`invalid hadoop.security.authentication.tbds.securekey`，表示StarrRocks无法使用TBDS提供的认证信息访问HDFS。[#14125](https://github.com/StarRocks/starrocks/pull/14125) [#15693](https://github.com/StarRocks/starrocks/pull/15693)
- 在某些情况下，CBO可能使用不正确的逻辑来比较两个运算符是否相等。[#17227](https://github.com/StarRocks/starrocks/pull/17227) [#17199](https://github.com/StarRocks/starrocks/pull/17199)
- 当您连接到非Leader FE节点并发送SQL语句`USE <catalog_name>.<database_name>`时，非Leader FE节点会转发SQL语句，`<catalog_name>`不包括在内，到Leader FE节点。结果，Leader FE节点选择使用`default_catalog`，最终无法找到指定的数据库。[#17302](https://github.com/StarRocks/starrocks/pull/17302)

## 2.3.8

发布日期：2023年2月2日

### 问题修复

已修复以下问题：

- 在大型查询完成后释放资源时，其他查询有很低概率被放缓。如果启用资源组或大型查询意外结束，则更有可能出现此问题。[#16454](https://github.com/StarRocks/starrocks/pull/16454) [#16602](https://github.com/StarRocks/starrocks/pull/16602)
- 对于主键表，如果复制品的元数据版本落后，则StarRocks将逐步从其他复制品克隆缺失的元数据。在此过程中，StarRocks拉取大量版本的元数据，如果未及时进行GC，将会消耗大量内存，从而导致BE可能遇到OOM异常。[#15935](https://github.com/StarRocks/starrocks/pull/15935)
- 如果FE偶尔向BE发送心跳，且心跳连接超时，则FE会认为BE不可用，导致BE上的事务失败。[# 16386](https://github.com/StarRocks/starrocks/pull/16386)
- 当您使用StarRocks外部表在StarRocks集群之间加载数据时，如果源StarRocks集群处于较早版本，目标StarRocks集群处于较新版本（2.2.8 ~ 2.2.11，2.3.4 ~ 2.3.7，2.4.1或2.4.2），数据加载将失败。[#16173](https://github.com/StarRocks/starrocks/pull/16173)
- 当多个查询并发运行且内存使用率相对较高时，BE会崩溃。[#16047](https://github.com/StarRocks/starrocks/pull/16047)
- 启用表的动态分区，删除一些分区后，执行TRUNCATE TABLE时，会返回错误`NullPointerException`。同时，如果加载数据到表中，则FE将崩溃并且无法重新启动。[#16822](https://github.com/StarRocks/starrocks/pull/16822)

## 2.3.7

发布日期：2022年12月30日

### 问题修复

已修复以下问题：

- 在StarRocks表中允许为NULL的列被错误地设置为视图中不允许为NULL。[#15749](https://github.com/StarRocks/starrocks/pull/15749)
- 加载数据到StarRocks后会生成新的平板版本。然而，FE可能尚未检测到新的平板版本，仍需要BE读取平板的历史版本。如果垃圾收集机制删除了历史版本，则查询无法找到历史版本，并返回错误"Not found: get_applied_rowsets(version xxxx) failed tablet:xxx #version:x [xxxxxxx]"。[#15726](https://github.com/StarRocks/starrocks/pull/15726)
- 当频繁加载数据时，FE会占用过多内存。[#15377](https://github.com/StarRocks/starrocks/pull/15377)
- 对于聚合查询和多表JOIN查询，统计数据不能准确收集，执行计划中出现CROSS JOIN，导致查询延迟较长。[#12067](https://github.com/StarRocks/starrocks/pull/12067)  [#14780](https://github.com/StarRocks/starrocks/pull/14780)

## 2.3.6

发布日期：2022年12月22日

### 改进

- Pipeline执行引擎支持INSERT INTO语句。要启用它，请将FE配置项`enable_pipeline_load_for_insert`设置为`true`。[#14723](https://github.com/StarRocks/starrocks/pull/14723)
- 减少主键表的压缩所使用的内存。[#13861](https://github.com/StarRocks/starrocks/pull/13861)  [#13862](https://github.com/StarRocks/starrocks/pull/13862)

### 行为更改

- 弃用了FE参数`default_storage_medium`。表的存储介质现在由系统自动推断。[#14394](https://github.com/StarRocks/starrocks/pull/14394)

### 问题修复

已修复以下问题：

- 当启用资源组功能并且多个资源组同时运行查询时，BE可能会挂起。[#14905](https://github.com/StarRocks/starrocks/pull/14905)
- 当使用CREATE MATERIALIZED VIEW AS SELECT创建物化视图时，如果SELECT子句未使用聚合函数，并且使用了GROUP BY，例如`CREATE MATERIALIZED VIEW test_view AS SELECT a,b from test group by b,a order by a;`，则所有BE节点都会崩溃。[#13743](https://github.com/StarRocks/starrocks/pull/13743)
- 如果您在频繁加载数据到主键表并在数据改变后立即重新启动BE，BE可能会重新启动非常缓慢。[#15128](https://github.com/StarRocks/starrocks/pull/15128)
- 如果环境中仅安装了JRE且未安装JDK，在FE重新启动后会发生查询失败。修复后，FE无法在该环境中重新启动，并返回错误`JAVA_HOME can not be jre`。要成功重新启动FE，需要在环境中安装JDK。[#14332](https://github.com/StarRocks/starrocks/pull/14332)
- 查询导致BE崩溃。[#14221](https://github.com/StarRocks/starrocks/pull/14221)
- 无法将`exec_mem_limit`设置为表达式。[#13647](https://github.com/StarRocks/starrocks/pull/13647)
- 无法基于子查询结果创建同步刷新的物化视图。[#13507](https://github.com/StarRocks/starrocks/pull/13507)
- 刷新Hive外部表后，列的注释被删除。[#13742](https://github.com/StarRocks/starrocks/pull/13742)
- 在相关联的JOIN期间，右表在左表之前处理，右表非常大。如果在右表被处理时在左表上执行压缩，则BE节点会崩溃。[#14070](https://github.com/StarRocks/starrocks/pull/14070)
- 如果Parquet文件列名区分大小写，并且查询条件使用来自Parquet文件的大写列名，则查询不返回结果。[#13860](https://github.com/StarRocks/starrocks/pull/13860) [#14773](https://github.com/StarRocks/starrocks/pull/14773)
- 在批量加载期间，如果连接到Broker的连接数超过默认最大连接数，Broker将断开连接，并且加载作业将失败，并返回错误消息`list path error`。[#13911](https://github.com/StarRocks/starrocks/pull/13911)
- 当BE负载较高时，资源组的度量`starrocks_be_resource_group_running_queries`可能不准确。[#14043](https://github.com/StarRocks/starrocks/pull/14043)
- 如果查询语句使用OUTER JOIN，可能导致BE节点崩溃。[#14840](https://github.com/StarRocks/starrocks/pull/14840)
- 创建异步物化视图使用 StarRocks 2.4 后，将其回滚到 2.3，可能导致前端 FE 启动失败。[#14400](https://github.com/StarRocks/starrocks/pull/14400)
- 当主键表使用 delete_range，并且性能不佳时，可能会减慢从 RocksDB 读取数据的速度并导致高 CPU 使用率。[#15130](https://github.com/StarRocks/starrocks/pull/15130)

## 2.3.5

发布日期：2022年11月30日

### 改进

- Colocate Join 支持等值连接。[#13546](https://github.com/StarRocks/starrocks/pull/13546)
- 修复主键索引文件太大的问题，解决了在数据频繁加载时连续添加 WAL 记录导致主键索引文件过大的问题。[#12862](https://github.com/StarRocks/starrocks/pull/12862)
- FE 扫描所有分区的批量操作，这样在扫描间隔时 FE 可以释放 db.readLock，防止长时间持有 db.readLock。[#13070](https://github.com/StarRocks/starrocks/pull/13070)

### Bug 修复

修复了以下问题：

- 当视图直接基于 UNION ALL 的结果创建时，且 UNION ALL 运算符的输入列包含 NULL 值时，视图的模式不正确，因为列的数据类型是 NULL_TYPE 而不是 UNION ALL 的输入列。[#13917](https://github.com/StarRocks/starrocks/pull/13917)
- `SELECT * FROM ...` 和 `SELECT * FROM ... LIMIT ...` 的查询结果不一致。[#13585](https://github.com/StarRocks/starrocks/pull/13585)
- 同步到 FE 的外部分区元数据可能会覆盖本地分区元数据，导致从 Flink 加载数据失败。[#12579](https://github.com/StarRocks/starrocks/pull/12579)
- 如果 Runtime Filter 处理文本常量时出现空过滤器，BE 节点会崩溃。[#13526](https://github.com/StarRocks/starrocks/pull/13526)
- 执行 CTAS 时返回错误。[#12388](https://github.com/StarRocks/starrocks/pull/12388)
- 管道引擎在审计日志中收集的指标 `ScanRows` 可能不正确。[#12185](https://github.com/StarRocks/starrocks/pull/12185)
- 查询压缩的 HIVE 数据时，查询结果不正确。[#11546](https://github.com/StarRocks/starrocks/pull/11546)
- BE 节点崩溃后，查询超时，StarRocks 响应缓慢。[#12955](https://github.com/StarRocks/starrocks/pull/12955)
- 使用 Broker Load 加载数据时出现 Kerberos 认证失败错误。[#13355](https://github.com/StarRocks/starrocks/pull/13355)
- 太多的 OR 谓词导致统计估算时间过长。[#13086](https://github.com/StarRocks/starrocks/pull/13086)
- 如果 Broker Load 加载包含大写列名的 ORC 文件（Snappy 压缩），BE 节点会崩溃。[#12724](https://github.com/StarRocks/starrocks/pull/12724)
- 卸载或查询主键表超过 30 分钟时返回错误。[#13403](https://github.com/StarRocks/starrocks/pull/13403)
- 使用 Broker 备份大数据量到 HDFS 时备份任务失败。[#12836](https://github.com/StarRocks/starrocks/pull/12836)
- StarRocks 从 Iceberg 读取的数据可能不正确，这是由 `parquet_late_materialization_enable` 参数导致的。[#13132](https://github.com/StarRocks/starrocks/pull/13132)
- 创建视图时返回 `failed to init view stmt` 错误。[#13102](https://github.com/StarRocks/starrocks/pull/13102)
- 使用 JDBC 连接 StarRocks 并执行 SQL 语句时返回错误。[#13526](https://github.com/StarRocks/starrocks/pull/13526)
- 查询超时，因为查询涉及太多分区并使用了 tablet hint。[#13272](https://github.com/StarRocks/starrocks/pull/13272)
- BE 节点崩溃且无法重新启动，在此期间，加载作业到新建表时报错。[#13701](https://github.com/StarRocks/starrocks/pull/13701)
- 创建物化视图时所有 BE 节点崩溃。[#13184](https://github.com/StarRocks/starrocks/pull/13184)
- 使用 ALTER ROUTINE LOAD 更新消耗分区的偏移量时可能会返回错误 `The specified partition 1 is not in the consumed partitions`，并导致 followers 崩溃。[#12227](https://github.com/StarRocks/starrocks/pull/12227)

## 2.3.4

发布日期：2022年11月10日

### 改进

- 当 StarRocks 失败创建 Routine Load 作业时，错误消息将提供解决方案，因为运行的 Routine Load 作业数量超过限制。[#12204](https://github.com/StarRocks/starrocks/pull/12204)
- 当 StarRocks 查询 Hive 中的数据并无法解析 CSV 文件时，查询失败。[#13013](https://github.com/StarRocks/starrocks/pull/13013)

### Bug 修复

修复了以下问题：

- 如果 HDFS 文件路径包含 `()`，可能会导致查询失败。[#12660](https://github.com/StarRocks/starrocks/pull/12660)
- 当子查询包含 LIMIT 时，ORDER BY ... LIMIT ... OFFSET 的结果不正确。[#9698](https://github.com/StarRocks/starrocks/issues/9698)
- StarRocks 查询 ORC 文件时不区分大小写。[#12724](https://github.com/StarRocks/starrocks/pull/12724)
- 如果 RuntimeFilter 在未调用 prepare 方法的情况下关闭，BE 可能会崩溃。[#12906](https://github.com/StarRocks/starrocks/issues/12906)
- BE 可能因内存泄漏而崩溃。[#12906](https://github.com/StarRocks/starrocks/issues/12906)
- 在添加新列并立即删除数据后，查询结果可能不正确。[#12907](https://github.com/StarRocks/starrocks/pull/12907)
- BE 可能因数据排序而崩溃。[#11185](https://github.com/StarRocks/starrocks/pull/11185)
- 如果 StarRocks 和 MySQL 客户端不在同一局域网上，使用 INSERT INTO SELECT 创建的加载作业可能不能通过仅执行一次 KILL 成功终止。[#11879](https://github.com/StarRocks/starrocks/pull/11897)
- 管道引擎在审计日志中收集的指标 `ScanRows` 可能不正确。[#12185](https://github.com/StarRocks/starrocks/pull/12185)

## 2.3.3

发布日期：2022年9月27日

### Bug 修复

修复了以下问题：

- 查询 Hive 外部表存储为文本文件时，查询结果可能不准确。[#11546](https://github.com/StarRocks/starrocks/pull/11546)
- 查询 Parquet 文件时不支持嵌套数组。[#10983](https://github.com/StarRocks/starrocks/pull/10983)
- 如果并发查询从 StarRocks 和外部数据源读取数据被路由到同一资源组，或者一个查询从 StarRocks 和外部数据源读取数据，可能导致查询超时或查询超时。[#10983](https://github.com/StarRocks/starrocks/pull/10983)
- 默认启用管道执行引擎时，参数 parallel_fragment_exec_instance_num 更改为 1，导致使用 INSERT INTO 加载数据变慢。[#11462](https://github.com/StarRocks/starrocks/pull/11462)
- 如果表达式初始化时出现错误，BE 可能会崩溃。[#11396](https://github.com/StarRocks/starrocks/pull/11396)
- 执行 ORDER BY LIMIT 可能导致堆缓冲区溢出错误。[#11185](https://github.com/StarRocks/starrocks/pull/11185)
- 如果重新启动 Leader FE，模式更改将失败。[#11561](https://github.com/StarRocks/starrocks/pull/11561)

## 2.3.2

发布日期：2022年9月7日

### 新特性

- 支持延迟实例化，加速基于范围过滤的外部表在 Parquet 格式上的查询。[#9738](https://github.com/StarRocks/starrocks/pull/9738)
- 添加 SHOW AUTHENTICATION 语句来显示用户认证相关信息。[#9996](https://github.com/StarRocks/starrocks/pull/9996)

### 改进

- 提供了一个配置项，用于控制StarRocks是否递归遍历所有的数据文件，用于从中查询数据的分桶Hive表。[#10239](https://github.com/StarRocks/starrocks/pull/10239)
- 资源组类型 `realtime` 被重命名为 `short_query`。[#10247](https://github.com/StarRocks/starrocks/pull/10247)
- StarRocks 不再默认区分Hive外部表中的大写字母和小写字母。[#10187](https://github.com/StarRocks/starrocks/pull/10187)

### Bug 修复

已修复以下问题：

- 当Elasticsearch外部表被划分为多个分片时，对该表的查询可能会意外退出。[#10369](https://github.com/StarRocks/starrocks/pull/10369)
- 当子查询被重写为通用表达式（CTEs）时，StarRocks会抛出错误。[#10397](https://github.com/StarRocks/starrocks/pull/10397)
- 当加载大量数据时，StarRocks会抛出错误。[#10370](https://github.com/StarRocks/starrocks/issues/10370) [#10380](https://github.com/StarRocks/starrocks/issues/10380)
- 当为多个目录配置相同的Thrift服务IP地址时，删除一个目录会使其他目录中的增量元数据更新失效。[#10511](https://github.com/StarRocks/starrocks/pull/10511)
- BE的内存消耗统计不准确。[#9837](https://github.com/StarRocks/starrocks/pull/9837)
- 对主键表的查询会导致StarRocks抛出错误。[#10811](https://github.com/StarRocks/starrocks/pull/10811)
- 即使对这些视图具有SELECT权限，也不允许对逻辑视图进行查询。[#10563](https://github.com/StarRocks/starrocks/pull/10563)
- StarRocks不会对逻辑视图的命名施加限制，现在逻辑视图需要遵循与表相同的命名约定。[#10558](https://github.com/StarRocks/starrocks/pull/10558)

### 行为变更

- 添加了BE配置项 `max_length_for_bitmap_function`，默认值为1000000，用于位图函数，并添加了 `max_length_for_to_base64`，默认值为200000，用于base64，以防止崩溃。[#10851](https://github.com/StarRocks/starrocks/pull/10851)

## 2.3.1

发布日期：2022年8月22日

### 改进

- Broker Load支持将Parquet文件中的List类型转换为非嵌套的ARRAY数据类型。[#9150](https://github.com/StarRocks/starrocks/pull/9150)
- 优化了与JSON相关函数（json_query，get_json_string 和 get_json_int）的性能。[#9623](https://github.com/StarRocks/starrocks/pull/9623)
- 优化了错误消息：在Hive、Iceberg或Hudi的查询中，如果要查询的列的数据类型不被StarRocks支持，系统会在该列上抛出异常。[#10139](https://github.com/StarRocks/starrocks/pull/10139)
- 减少了资源组的调度延迟，以优化资源隔离性能。[#10122](https://github.com/StarRocks/starrocks/pull/10122)

### Bug 修复

已修复以下问题：

- 由于 `limit` 操作符的错误下推，对Elasticsearch外部表的查询返回了错误结果。[#9952](https://github.com/StarRocks/starrocks/pull/9952)
- 使用 `limit` 操作符对Oracle外部表进行查询时失败。[#9542](https://github.com/StarRocks/starrocks/pull/9542)
- 当所有Kafka Brokers停止时，BE被阻塞。[#9935](https://github.com/StarRocks/starrocks/pull/9935)
- 在查询Parquet文件时，如果数据类型与相应的外部表的数据类型不匹配，BE会崩溃。[#10107](https://github.com/StarRocks/starrocks/pull/10107)
- 查询超时，因为外部表的扫描范围为空。[#10091](https://github.com/StarRocks/starrocks/pull/10091)
- 当子查询中包含 ORDER BY 子句时，系统会抛出异常。[#10180](https://github.com/StarRocks/starrocks/pull/10180)
- 在异步重新加载Hive元数据时，Hive Metastore会挂起。[#10132](https://github.com/StarRocks/starrocks/pull/10132)

## 2.3.0

发布日期：2022年7月29日

### 新功能

- 主键表支持完整的DELETE WHERE语法。有关更多信息，请参见[DELETE](../sql-reference/sql-statements/data-manipulation/DELETE.md#delete-data-by-primary-key)。
- 主键表支持持久主键索引。您可以选择将主键索引持久化到磁盘上，而不是存储在内存中，这将大大减少内存使用。有关更多信息，请参见[主键表](../table_design/table_types/primary_key_table.md)。
- 全局字典可以在实时数据摄入期间进行更新，以优化查询性能，并为字符串数据提供2倍的查询性能。
- CREATE TABLE AS SELECT语句可以异步执行。有关更多信息，请参见[CREATE TABLE AS SELECT](../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)。
- 支持以下与资源组相关的功能：
  - 监视资源组：您可以在审计日志中查看查询的资源组，并通过调用API获取资源组的指标。有关更多信息，请参见[监视和报警](../administration/Monitor_and_Alert.md#monitor-and-alerting)。
  - 限制对CPU、内存和I/O资源的大型查询的消耗：您可以根据分类器或通过配置会话变量将查询路由到特定的资源组。有关更多信息，请参见[资源组](../administration/resource_group.md)。
- JDBC外部表可用于方便地查询Oracle、PostgreSQL、MySQL、SQLServer、ClickHouse和其他数据库中的数据。StarRocks还支持谓词下推，提高查询性能。有关更多信息，请参见[JDBC兼容数据库的外部表](../data_source/External_table.md#external-table-for-a-JDBC-compatible-database)。
- [预览] 新的数据源连接器框架已发布，支持外部目录。您可以使用外部目录直接访问和查询Hive数据，而无需创建外部表。有关更多信息，请参见[使用目录管理内部和外部数据](../data_source/catalog/query_external_data.md)。
- 添加了以下函数：
  - [window_funnel](../sql-reference/sql-functions/aggregate-functions/window_funnel.md)
  - [ntile](../sql-reference/sql-functions/Window_function.md)
  - [bitmap_union_count](../sql-reference/sql-functions/bitmap-functions/bitmap_union_count.md)，[base64_to_bitmap](../sql-reference/sql-functions/bitmap-functions/base64_to_bitmap.md)，[array_to_bitmap](../sql-reference/sql-functions/array-functions/array_to_bitmap.md)
  - [week](../sql-reference/sql-functions/date-time-functions/week.md)，[time_slice](../sql-reference/sql-functions/date-time-functions/time_slice.md)

### 改进

- 紧缩机制可以更快地合并大量的元数据。这可以防止频繁数据更新后不久发生的元数据挤压和过度磁盘使用。
- 优化了加载Parquet文件和压缩文件的性能。
- 优化了创建物化视图的机制。优化后，物化视图的创建速度可以比以前快10倍。
- 优化了以下操作符的性能：
  - TopN和排序操作符
  - 包含函数的等价比较操作符在下推到扫描操作符时可以使用Zone Map索引。
- 优化了Apache Hive™外部表的性能。
  - 当Apache Hive™表存储在Parquet、ORC或CSV格式中时，Hive上的ADD COLUMN或REPLACE COLUMN引起的模式变更可以在执行相应Hive外部表上的REFRESH语句时同步到StarRocks。有关更多信息，请参见[Hive外部表](../data_source/External_table.md#hive-external-table)。
  - `hive.metastore.uris` 可以修改Hive资源。有关更多信息，请参见[ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md)。
- 优化了Apache Iceberg外部表的性能。可以使用自定义目录来创建Iceberg资源。有关更多信息，请参见[Apache Iceberg外部表](../data_source/External_table.md#apache-iceberg-external-table)。
- 优化了Elasticsearch外部表的性能。可以禁用对Elasticsearch集群的数据节点地址进行嗅探。有关更多信息，请参见[Elasticsearch外部表](../data_source/External_table.md#elasticsearch-external-table)。
- 当sum()函数接受数值字符串时，会隐式转换数值字符串。
- year()、month()和day()函数支持DATE数据类型。

### Bug 修复

已修复以下问题：

- 由于大量tablet的存在，CPU利用率激增。
- 导致“无法准备平板读取器”出现的问题。
- FEs无法重启。[#5642](https://github.com/StarRocks/starrocks/issues/5642 )  [#4969](https://github.com/StarRocks/starrocks/issues/4969 )  [#5580](https://github.com/StarRocks/starrocks/issues/5580)
- 在CTAS语句中包含JSON函数时，无法成功运行该语句。[#6498](https://github.com/StarRocks/starrocks/issues/6498)

### 其他

- StarGo 是一个集群管理工具，可以部署、启动、升级和回滚集群，并管理多个集群。更多信息，请参见[使用 StarGo 部署 StarRocks](../administration/stargo.md)。
- 当您将 StarRocks 升级至 2.3 版本或部署 StarRocks 时，默认启用管道引擎。管道引擎可以提高高并发场景下简单查询和复杂查询的性能。如果在使用 StarRocks 2.3 时检测到性能显著降低，您可以通过执行 `SET GLOBAL` 语句将 `enable_pipeline_engine` 设置为 `false` 来禁用管道引擎。
- [SHOW GRANTS](../sql-reference/sql-statements/account-management/SHOW_GRANTS.md) 语句兼容 MySQL 语法，并以 GRANT 语句的形式显示用户被分配的权限。
- 建议将 memory_limitation_per_thread_for_schema_change（BE 配置项）使用默认值 2 GB，当数据量超过此限制时会将数据写入磁盘。因此，如果您之前将此参数设置为较大的值，建议您将其设置为 2 GB，否则模式更改任务可能会占用大量内存。

### 升级说明

要回滚至升级前使用的先前版本，需将 `ignore_unknown_log_id` 参数添加到每个 FE 的 **fe.conf** 文件中，并将该参数设置为 `true`。该参数是必需的，因为在 StarRocks v2.2.0 中增加了新类型的日志。如果您不添加该参数，将无法回滚至先前版本。我们建议在创建检查点后，将 `ignore_unknown_log_id` 参数设置为 `false`，然后重新启动 FEs 以将 FEs 恢复到先前的配置。