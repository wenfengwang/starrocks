---
displayed_sidebar: English
---

# StarRocks 2.5 版本

## 2.5.17

发布日期：2023年12月19日

### 新特性

- 添加了一个新的指标 `max_tablet_rowset_num`，用于设置允许的最大行集数。该指标有助于检测可能的压缩问题，从而减少“版本过多”错误的发生。[#36539](https://github.com/StarRocks/starrocks/pull/36539)
- 新增了 [subdivide_bitmap](https://docs.starrocks.io/zh/docs/sql-reference/sql-functions/bitmap-functions/subdivide_bitmap/) 函数。[#35817](https://github.com/StarRocks/starrocks/pull/35817)

### 改进

- [SHOW ROUTINE LOAD](https://docs.starrocks.io/zh/docs/sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD/) 语句返回的结果现在提供了一个新的 `OtherMsg` 字段，显示有关上次失败任务的信息。[#35806](https://github.com/StarRocks/starrocks/pull/35806)
- 回收站文件的默认保留期从原来的3天更改为1天。[#37113](https://github.com/StarRocks/starrocks/pull/37113)
- 优化了对主键表进行全行集压缩时持久索引更新的性能，从而减少了磁盘读取 I/O。[#36819](https://github.com/StarRocks/starrocks/pull/36819)
- 优化了计算主键表压缩分数的逻辑，使其与其他三种表类型的压缩分数范围更一致。[#36534](https://github.com/StarRocks/starrocks/pull/36534)
- MySQL外部表和JDBC目录中的外部表的查询现在支持在WHERE子句中包含关键字。[#35917](https://github.com/StarRocks/starrocks/pull/35917)
- 添加了bitmap_from_binary函数到Spark Load，以支持加载二进制数据。[#36050](https://github.com/StarRocks/starrocks/pull/36050)
- bRPC过期时间从1小时缩短到会话变量[`query_timeout`](https://docs.starrocks.io/zh/docs/3.2/reference/System_variable/#query_timeout)指定的持续时间。这可以防止由于RPC请求过期而导致的查询失败。[#36778](https://github.com/StarRocks/starrocks/pull/36778)

### 兼容性更改

#### 参数

- 新增了BE配置项 `enable_stream_load_verbose_log`。默认值为`false`。将此参数设置为`true`时，StarRocks可以记录Stream Load作业的HTTP请求和响应，从而简化故障排除。[#36113](https://github.com/StarRocks/starrocks/pull/36113)
- BE静态参数`update_compaction_per_tablet_min_interval_seconds`变为可变参数。[#36819](https://github.com/StarRocks/starrocks/pull/36819)

### Bug修复

修复了以下问题：

- 在哈希联接期间，查询失败导致BE崩溃。[#32219](https://github.com/StarRocks/starrocks/pull/32219)
- 将FE配置项`enable_collect_query_detail_info`设置为`true`后，FE性能急剧下降。[#35945](https://github.com/StarRocks/starrocks/pull/35945)
- 如果大量数据加载到启用了持久索引的主键表中，可能会引发错误。[#34352](https://github.com/StarRocks/starrocks/pull/34352)
- 当使用`./agentctl.sh stop be`停止BE时，starrocks_be进程可能会意外退出。[#35108](https://github.com/StarRocks/starrocks/pull/35108)
- [array_distinct](https://docs.starrocks.io/docs/sql-reference/sql-functions/array-functions/array_distinct/)函数偶尔会导致BE崩溃。[#36377](https://github.com/StarRocks/starrocks/pull/36377)
- 用户刷新材料化视图时可能会发生死锁。[#35736](https://github.com/StarRocks/starrocks/pull/35736)
- 在某些情况下，动态分区可能会遇到错误，导致FE启动失败。[#36846](https://github.com/StarRocks/starrocks/pull/36846)

## 2.5.16

发布日期：2023年12月1日

### Bug修复

修复了以下问题：

- 在某些情况下，全局运行时筛选器可能会导致BE崩溃。[#35776](https://github.com/StarRocks/starrocks/pull/35776)

## 2.5.15

发布日期：2023年11月29日

### 改进

- 新增了慢请求日志，用于跟踪慢请求。[#33908](https://github.com/StarRocks/starrocks/pull/33908)
- 优化了使用Spark Load读取Parquet和ORC文件时，当文件数量较多时的性能。[#34787](https://github.com/StarRocks/starrocks/pull/34787)
- 优化了部分位图相关操作的性能，包括：
  - 优化了嵌套循环连接。[#340804](https://github.com/StarRocks/starrocks/pull/34804) [#35003](https://github.com/StarRocks/starrocks/pull/35003)
  - 优化了`bitmap_xor`功能。[#34069](https://github.com/StarRocks/starrocks/pull/34069)
  - 支持写时复制以优化位图性能并减少内存消耗。[#34047](https://github.com/StarRocks/starrocks/pull/34047)

### 兼容性更改

#### 参数

- FE动态参数`enable_new_publish_mechanism`改为静态参数。修改参数设置后，需要重启FE。[#35338](https://github.com/StarRocks/starrocks/pull/35338)

### Bug修复

修复了以下问题：

- 如果在Broker Load作业中指定了过滤条件，那么在某些情况下，BE可能会在数据加载期间崩溃。[#29832](https://github.com/StarRocks/starrocks/pull/29832)
- 副本操作重放失败可能导致FE崩溃。[#32295](https://github.com/StarRocks/starrocks/pull/32295)
- 将FE参数设置为`recover_with_empty_tablet`可能会导致FE崩溃。[#33071](https://github.com/StarRocks/starrocks/pull/33071)
- 查询返回错误“get_applied_rowsets失败，平板电脑更新处于错误状态：tablet：18849实际行大小在压缩后更改”。[#33246](https://github.com/StarRocks/starrocks/pull/33246)
- 包含窗口函数的查询可能会导致BE崩溃。[#33671](https://github.com/StarRocks/starrocks/pull/33671)
- 运行`show proc '/statistic'`可能会导致死锁。[#34237](https://github.com/StarRocks/starrocks/pull/34237/files)
- 如果将大量数据加载到启用了持久索引的主键表中，可能会引发错误。[#34566](https://github.com/StarRocks/starrocks/pull/34566)
- 从v2.4或更早版本升级到后，压实度分数可能会意外上升。[#34618](https://github.com/StarRocks/starrocks/pull/34618)
- 如果使用数据库驱动程序MariaDB ODBC查询`INFORMATION_SCHEMA`，则`CATALOG_NAME`视图返回的列仅包含`null`值。[#34627](https://github.com/StarRocks/starrocks/pull/34627)
- 如果在流加载作业处于**PREPARD**状态时执行架构更改，则该作业要加载的部分源数据将丢失。[#34381](https://github.com/StarRocks/starrocks/pull/34381)
- 在HDFS存储路径的末尾包含两个或多个斜杠（`/`）会导致从HDFS备份和恢复数据失败。[#34601](https://github.com/StarRocks/starrocks/pull/34601)
- 运行加载任务或查询可能会导致FE挂起。[#34569](https://github.com/StarRocks/starrocks/pull/34569)

## 2.5.14

发布日期：2023年11月14日

### 改进

- `INFORMATION_SCHEMA`系统数据库中的`COLUMNS`表现在可以显示ARRAY、MAP和STRUCT列。[#33431](https://github.com/StarRocks/starrocks/pull/33431)

### 兼容性更改

#### 系统变量

- 添加了一个会话变量`cbo_decimal_cast_string_strict`，用于控制CBO如何将DECIMAL类型的数据转换为STRING类型。如果将此变量设置为`true`，则以v2.5.x及更高版本内置的逻辑为准，系统实现严格转换（即系统截断生成的字符串，并根据刻度长度填充0）。如果将此变量设置为`false`，则以v2.5.x之前版本内置的逻辑为准，系统处理所有有效数字以生成字符串。默认值为`true`。[#34208](https://github.com/StarRocks/starrocks/pull/34208)
- 添加了一个会话变量`cbo_eq_base_type`，用于指定DECIMAL类型数据和STRING类型数据之间的数据比较所使用的数据类型。默认值为`VARCHAR`，DECIMAL也是有效值。[#34208](https://github.com/StarRocks/starrocks/pull/34208)

### Bug修复

修复了以下问题：

- 如果ON条件嵌套在子查询中，则会报`java.lang.IllegalStateException: null`错。[#30876](https://github.com/StarRocks/starrocks/pull/30876)
- 如果在成功执行`INSERT INTO SELECT ... LIMIT`后立即运行COUNT（*），则不同副本之间的COUNT（*）结果不一致。[#24435](https://github.com/StarRocks/starrocks/pull/24435)
- 如果 cast() 函数中指定的目标数据类型与原始数据类型相同，则特定数据类型的 BE 可能会崩溃。 [#31465](https://github.com/StarRocks/starrocks/pull/31465)
- 如果在通过 Broker Load 加载数据期间使用特定路径格式，那么会报告错误：`msg:Fail to parse columnsFromPath, expected: [rec_dt]`。[#32721](https://github.com/StarRocks/starrocks/issues/32721)
- 在升级到 3.x 期间，如果还升级了某些列类型（例如，Decimal 升级到 Decimal v3），则在对具有特定特征的表执行压缩时，BE 会崩溃。 [#31626](https://github.com/StarRocks/starrocks/pull/31626)
- 使用 Flink Connector 加载数据时，如果存在高并发加载作业，且 HTTP 线程数和 Scan 线程数均达到上限，则加载作业会意外挂起。 [#32251](https://github.com/StarRocks/starrocks/pull/32251)
- 调用 libcurl 时 BE 崩溃。 [#31667](https://github.com/StarRocks/starrocks/pull/31667)
- 将 BITMAP 列添加到主键表失败，并显示以下错误：`Analyze columnDef error: No aggregate function specified for 'userid'`。 [#31763](https://github.com/StarRocks/starrocks/pull/31763)
- 长时间、频繁地将数据加载到启用了持久索引的主键表中可能会导致 BE 崩溃。 [#33220](https://github.com/StarRocks/starrocks/pull/33220)
- 启用查询缓存时，查询结果不正确。 [#32778](https://github.com/StarRocks/starrocks/pull/32778)
- 在创建主键表时指定可为 null 的排序键会导致压缩失败。 [#29225](https://github.com/StarRocks/starrocks/pull/29225)
- 对于复杂的 Join 查询，偶尔会出现错误：“StarRocks planner use long time 10000 ms in logical phase”。 [#34177](https://github.com/StarRocks/starrocks/pull/34177)

## 2.5.13

发布日期：2023年9月28日

### 改进

- 窗口函数 COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD 和 STDDEV_SAMP 现在支持 ORDER BY 子句和 Window 子句。 [#30786](https://github.com/StarRocks/starrocks/pull/30786)
- 如果在对 DECIMAL 类型数据进行查询期间发生十进制溢出，则返回错误而不是 NULL。 [#30419](https://github.com/StarRocks/starrocks/pull/30419)
- 现在，执行带有无效注释的 SQL 命令将返回与 MySQL 一致的结果。 [#30210](https://github.com/StarRocks/starrocks/pull/30210)
- 清理已删除的 tablet 对应的行集，减少 BE 启动期间的内存使用量。 [#30625](https://github.com/StarRocks/starrocks/pull/30625)

### Bug 修复

修复以下问题：

- 用户使用 Spark Connector 或 Flink Connector 从 StarRocks 读取数据时，出现“Set cancelled by MemoryScratchSinkOperator”错误。 [#30702](https://github.com/StarRocks/starrocks/pull/30702) [#30751](https://github.com/StarRocks/starrocks/pull/30751)
- 在使用包含聚合函数的 ORDER BY 子句进行查询时出现错误：“java.lang.IllegalStateException：null”。 [#30108](https://github.com/StarRocks/starrocks/pull/30108)
- 当存在非活动物化视图时，FE 无法重新启动。 [#30015](https://github.com/StarRocks/starrocks/pull/30015)
- 对重复分区执行 INSERT OVERWRITE 操作会损坏元数据，导致 FE 重启失败。 [#27545](https://github.com/StarRocks/starrocks/pull/27545)
- 当用户修改主键表中不存在的列时，会发生错误：“java.lang.NullPointerException：null”。 [#30366](https://github.com/StarRocks/starrocks/pull/30366)
- 当用户将数据加载到分区的 StarRocks 外部表中时，会出现“get TableMeta failed from TNetworkAddress”错误。 [#30124](https://github.com/StarRocks/starrocks/pull/30124)
- 在某些情况下，用户通过 CloudCanal 加载数据时会出现错误。 [#30799](https://github.com/StarRocks/starrocks/pull/30799)
- 当用户通过 Flink Connector 加载数据或执行 DELETE 和 INSERT 操作时，出现“当前在 db xxx 上运行的 txns 为 200，大于限制 200”错误。 [#18393](https://github.com/StarRocks/starrocks/pull/18393)
- 使用包含聚合函数的 HAVING 子句的异步实例化视图无法正确重写查询。 [#29976](https://github.com/StarRocks/starrocks/pull/29976)

## 2.5.12

发布日期：2023年9月4日

### 改进

- SQL 中的注释保留在审核日志中。 [#29747](https://github.com/StarRocks/starrocks/pull/29747)
- 在审计日志中添加了 INSERT INTO SELECT 的 CPU 和内存统计信息。 [#29901](https://github.com/StarRocks/starrocks/pull/29901)

### Bug 修复

修复以下问题：

- 使用 Broker Load 加载数据时，某些字段的 NOT NULL 属性可能会导致 BE 崩溃或出现“msg：mismatched row count”错误。 [#29832](https://github.com/StarRocks/starrocks/pull/29832)
- 对 ORC 格式文件的查询失败，因为 Apache ORC 中的错误修复 ORC-1304（[apache/orc#1299](https://github.com/apache/orc/pull/1299)）未合并。 [#29804](https://github.com/StarRocks/starrocks/pull/29804)
- 恢复主键表会导致重启 BE 后元数据不一致。 [#30135](https://github.com/StarRocks/starrocks/pull/30135)
- 在创建外部表时，禁止用户定义 NOT NULL 列（如果定义了 NOT NULL 列，升级后将会出现错误，必须重新创建表）。建议从 v2.3.0 开始使用外部目录来替换外部表。 [#25485](https://github.com/StarRocks/starrocks/pull/25441)
- 当 Broker Load 重试遇到错误时，添加了错误消息。这有助于在数据加载过程中进行故障排除和调试。 [#21982](https://github.com/StarRocks/starrocks/pull/21982)
- 当加载作业涉及 UPSERT 和 DELETE 操作时，支持大规模数据写入。 [#17264](https://github.com/StarRocks/starrocks/pull/17264)
- 优化了使用实例化视图进行查询重写。 [#27934](https://github.com/StarRocks/starrocks/pull/27934) [#25542](https://github.com/StarRocks/starrocks/pull/25542) [#22300](https://github.com/StarRocks/starrocks/pull/22300) [#27557](https://github.com/StarRocks/starrocks/pull/27557) [#22300](https://github.com/StarRocks/starrocks/pull/22300) [#26957](https://github.com/StarRocks/starrocks/pull/26957) [#27728](https://github.com/StarRocks/starrocks/pull/27728) [#27900](https://github.com/StarRocks/starrocks/pull/27900)

### Bug 修复

修复了以下问题：

- 当使用 CAST 将字符串转换为数组时，如果输入包含常量，则结果可能不正确。 [#19793](https://github.com/StarRocks/starrocks/pull/19793)
- 如果 SHOW TABLET 包含 ORDER BY 和 LIMIT，则返回的结果不正确。 [#23375](https://github.com/StarRocks/starrocks/pull/23375)
- 实例化视图的外部联接和反联接重写错误。 [#28028](https://github.com/StarRocks/starrocks/pull/28028)
- FE 中不正确的表级扫描统计信息会导致表查询和加载指标不准确。 [#27779](https://github.com/StarRocks/starrocks/pull/27779)
- 当使用当前长链接访问元数据存储时出现异常。消息：根据上次事件 ID 获取下一个通知失败：707602。如果在 HMS 上配置了事件监听器以增量更新 Hive 元数据，则在 FE 日志中报告。 [#21056](https://github.com/StarRocks/starrocks/pull/21056)
- 如果修改了分区表的排序键，则查询结果不稳定。 [#27850](https://github.com/StarRocks/starrocks/pull/27850)
- 如果使用 Spark Load 加载数据，且存储桶列为 DATE、DATETIME 或 DECIMAL 列，则数据可能会分发到错误的存储桶。 [#27005](https://github.com/StarRocks/starrocks/pull/27005)
- 在某些情况下，regex_replace 功能可能会导致 BE 崩溃。 [#27117](https://github.com/StarRocks/starrocks/pull/27117)
- 如果 sub_bitmap 函数的输入不是 BITMAP 值，则 BE 会崩溃。 [#27982](https://github.com/StarRocks/starrocks/pull/27982)
- 启用联接重新排序时，查询将返回“未知错误”。 [#27472](https://github.com/StarRocks/starrocks/pull/27472)
- 对平均行大小的不准确估计会导致主键部分更新占用过大的内存。 [#27485](https://github.com/StarRocks/starrocks/pull/27485)
- 如果启用了低基数优化，某些 INSERT 作业将返回 `[42000][1064] Dict Decode failed, Dict can't take cover all key :0`。[#26463](https://github.com/StarRocks/starrocks/pull/26463)
- 如果用户在创建的 Broker Load 作业中指定要从 HDFS 加载数据的参数 `"hadoop.security.authentication" = "simple"`，则该作业将失败。 [#27774](https://github.com/StarRocks/starrocks/pull/27774)
- 修改物化视图的刷新模式会导致 leader FE 和 follower FE 之间的元数据不一致。 [#28082](https://github.com/StarRocks/starrocks/pull/28082) [#28097](https://github.com/StarRocks/starrocks/pull/28097)
- 使用 SHOW CREATE CATALOG 和 SHOW RESOURCES 查询特定信息时，密码不会被隐藏。 [#28059](https://github.com/StarRocks/starrocks/pull/28059)
- 由于 LabelCleaner 线程阻塞导致的 FE 内存泄漏。 [#28311](https://github.com/StarRocks/starrocks/pull/28311)

## 2.5.9

发布日期：2023年7月19日

### 新功能

- 可以重写包含与实例化视图不同类型的联接的查询。 [#25099](https://github.com/StarRocks/starrocks/pull/25099)

### 改进

- 无法创建目标集群为当前 StarRocks 集群的 StarRocks 外部表。 [#25441](https://github.com/StarRocks/starrocks/pull/25441)
- 如果查询的字段未包含在实例化视图的输出列中，但包含在实例化视图的谓词中，则仍可重写查询。 [#23028](https://github.com/StarRocks/starrocks/issues/23028)
- 向 `table_id` 数据库的表 `tables_config` 中添加了一个新字段 `Information_schema`。您可以使用列 `table_id` 将 `tables_config` 与 `be_tablets` 进行连接，查询 tablet 所属的数据库和表的名称。 [#24061](https://github.com/StarRocks/starrocks/pull/24061)

### Bug 修复

修复了以下问题：

- 重复键表的 Count Distinct 结果不正确。 [#24222](https://github.com/StarRocks/starrocks/pull/24222)
- 如果 Join 键是一个较大的 BINARY 列，则 BE 可能会崩溃。 [#25084](https://github.com/StarRocks/starrocks/pull/25084)
- 如果要插入的 STRUCT 中 CHAR 数据的长度超过 STRUCT 列中定义的最大 CHAR 长度，则 INSERT 操作将挂起。 [#25942](https://github.com/StarRocks/starrocks/pull/25942)
- coalesce（） 的结果不正确。 [#26250](https://github.com/StarRocks/starrocks/pull/26250)
- 恢复数据后，平板电脑的 BE 和 FE 版本号不一致。 [#26518](https://github.com/StarRocks/starrocks/pull/26518/files)
- 无法为恢复的表自动创建分区。 [#26813](https://github.com/StarRocks/starrocks/pull/26813)

## 2.5.8

发布日期：2023年6月30日

### 改进

- 优化未分区表添加分区时报错信息。 [#25266](https://github.com/StarRocks/starrocks/pull/25266)
- 优化[表的自动平板分发策略](../table_design/Data_distribution.md#determine-the-number-of-buckets)。 [#24543](https://github.com/StarRocks/starrocks/pull/24543)
- 优化 CREATE TABLE 语句中默认注释。 [#24803](https://github.com/StarRocks/starrocks/pull/24803)
- 优化异步物化视图的手动刷新。支持使用 REFRESH MATERIALIZED VIEW WITH SYNC MODE 语法同步调用物化视图刷新任务。 [#25910](https://github.com/StarRocks/starrocks/pull/25910)

### Bug 修复

修复了以下问题：

- 如果异步实例化视图是基于联合结果构建的，则该实例化视图的 COUNT 结果可能不准确。 [#24460](https://github.com/StarRocks/starrocks/issues/24460)
- 当用户尝试强制重置 root 密码时，会报告“未知错误”。 [#25492](https://github.com/StarRocks/starrocks/pull/25492)
- 在活动 BE 少于 3 个的集群上执行 INSERT OVERWRITE 时，会显示不准确的错误消息。 [#25314](https://github.com/StarRocks/starrocks/pull/25314)

## 2.5.7

发布日期：2023年6月14日

### 新功能

- 可以使用 ALTER MATERIALIZED VIEW <mv_name> ACTIVE 手动激活非活动物化视图。您可以使用此 SQL 命令激活其基表被删除然后重新创建的实例化视图。有关详细信息，请参阅 [ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md)。 [#24001](https://github.com/StarRocks/starrocks/pull/24001)
- StarRocks 可以在您创建表或添加分区时自动设置合适的 tablet 数量，无需手动操作。有关详细信息，请参阅 [确定平板电脑的数量](../table_design/Data_distribution.md#determine-the-number-of-buckets)。 [#10614](https://github.com/StarRocks/starrocks/pull/10614)

### 改进

- 优化外表查询中使用的 Scan 节点的 I/O 并发，减少内存占用，提高外表数据加载的稳定性。 [#23617](https://github.com/StarRocks/starrocks/pull/23617) [#23624](https://github.com/StarRocks/starrocks/pull/23624) [#23626](https://github.com/StarRocks/starrocks/pull/23626)
- 优化 Broker Load 作业的错误信息。错误消息包含重试信息和错误文件的名称。 [#18038](https://github.com/StarRocks/starrocks/pull/18038) [#21982](https://github.com/StarRocks/starrocks/pull/21982)
- 优化 CREATE TABLE 超时时返回的错误信息，增加参数调优提示。 [#24510](https://github.com/StarRocks/starrocks/pull/24510)
- 优化 ALTER TABLE 因表状态不为 Normal 而失败时返回的错误信息。 [#24381](https://github.com/StarRocks/starrocks/pull/24381)
- 忽略 CREATE TABLE 语句中的全角空格。 [#23885](https://github.com/StarRocks/starrocks/pull/23885)
- 优化 Broker 访问超时，提升 Broker Load 作业成功率。 [#22699](https://github.com/StarRocks/starrocks/pull/22699)
- 对于主键表，SHOW TABLET返回的`VersionCount`字段包含处于Pending状态的行集。[#23847](https://github.com/StarRocks/starrocks/pull/23847)
- 优化了持久索引策略。[#22140](https://github.com/StarRocks/starrocks/pull/22140)

### Bug修复

修复了以下问题：

- 用户将Parquet数据加载到StarRocks时，DATETIME值在类型转换过程中会溢出，导致数据错误。[#22356](https://github.com/StarRocks/starrocks/pull/22356)
- 禁用动态分区后，存储桶信息将丢失。[#22595](https://github.com/StarRocks/starrocks/pull/22595)
- 在CREATE TABLE语句中使用不受支持的属性会导致空指针异常（NPE）。[#23859](https://github.com/StarRocks/starrocks/pull/23859)
- `information_schema`中的表权限筛选变得无效。因此，用户可以查看他们无权访问的表。[#23804](https://github.com/StarRocks/starrocks/pull/23804)
- SHOW TABLE STATUS返回的信息不完整。[#24279](https://github.com/StarRocks/starrocks/issues/24279)
- 数据加载与架构更改同时发生时，架构更改有时可能会挂起。[#23456](https://github.com/StarRocks/starrocks/pull/23456)
- RocksDB WAL flush会阻止brpc worker处理bthreads，从而中断高频数据加载到主键表中。[#22489](https://github.com/StarRocks/starrocks/pull/22489)
- StarRocks不支持的TIME类型列可以成功创建。[#23474](https://github.com/StarRocks/starrocks/pull/23474)
- 具体化视图并集重写失败。[#22922](https://github.com/StarRocks/starrocks/pull/22922)

## 2.5.6

发布日期：2023年5月19日

### 改进

- 优化了INSERT INTO ... SELECT由于`thrift_server_max_worker_thread`值较小而过期时报告的错误消息。[#21964](https://github.com/StarRocks/starrocks/pull/21964)
- 使用CTAS创建的表默认有三个副本，这与普通表的默认副本数一致。[#22854](https://github.com/StarRocks/starrocks/pull/22854)

### Bug修复

- 截断分区失败，因为TRUNCATE操作对分区名称区分大小写。[#21809](https://github.com/StarRocks/starrocks/pull/21809)
- 由于无法为物化视图创建临时分区，因此BE的停用失败。[#22745](https://github.com/StarRocks/starrocks/pull/22745)
- 需要ARRAY值的动态FE参数不能设置为空数组。[#22225](https://github.com/StarRocks/starrocks/pull/22225)
- 指定了`partition_refresh_number`属性的物化视图可能无法完全刷新。[#21619](https://github.com/StarRocks/starrocks/pull/21619)
- SHOW CREATE TABLE屏蔽云凭据信息，导致内存中的凭据信息不正确。[#21311](https://github.com/StarRocks/starrocks/pull/21311)
- 对通过外部表查询的某些ORC文件，谓词无法生效。[#21901](https://github.com/StarRocks/starrocks/pull/21901)
- min-max筛选器无法正确处理列名中的小写和大写字母。[#22626](https://github.com/StarRocks/starrocks/pull/22626)
- 后期具体化导致查询复杂数据类型（STRUCT或MAP）时出错。[#22862](https://github.com/StarRocks/starrocks/pull/22862)
- 还原主键表时出现的问题。[#23384](https://github.com/StarRocks/starrocks/pull/23384)

## 2.5.5

发布日期：2023年4月28日

### 新功能

添加了用于监控主键表的tablet状态的指标：

- 新增了FE指标`err_state_metric`。
- 在`SHOW PROC '/statistic/'`的输出中添加了`ErrorStateTabletNum`列，以显示err_state平板电脑的数量。
- 在`SHOW PROC '/statistic/<db_id>/'`的输出中添加了`ErrorStateTablets`列，以显示err_state平板电脑的ID。

有关详细信息，请参阅[SHOW PROC](../sql-reference/sql-statements/Administration/SHOW_PROC.md)。

### 改进

- 优化了添加多个BE时的磁盘平衡速度。[#19418](https://github.com/StarRocks/starrocks/pull/19418)
- 优化了存储介质的推断逻辑。当BE同时使用SSD和HDD作为存储设备时，如果指定了`storage_cooldown_time`属性，则StarRocks将`storage_medium`设置为`SSD`。否则，StarRocks将`storage_medium`设置为`HDD`。[#18649](https://github.com/StarRocks/starrocks/pull/18649)
- 优化了唯一键表的性能，禁止从值列收集统计信息。[#19563](https://github.com/StarRocks/starrocks/pull/19563)

### Bug修复

- 对于共置表，可以使用类似语句手动指定副本状态为`bad`，例如`ADMIN SET REPLICA STATUS PROPERTIES ("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");`。如果BE数小于或等于副本数，则无法修复损坏的副本。[#17876](https://github.com/StarRocks/starrocks/issues/17876)
- BE启动后，其进程存在，但BE端口无法启用。[#19347](https://github.com/StarRocks/starrocks/pull/19347)
- 对于子查询嵌套在窗口函数中的聚合查询，将返回错误的结果。[#19725](https://github.com/StarRocks/starrocks/issues/19725)
- `auto_refresh_partitions_limit`首次刷新物化视图（MV）时不生效。因此，将刷新所有分区。[#19759](https://github.com/StarRocks/starrocks/issues/19759)
- 查询数组数据嵌套有复杂数据（如MAP和STRUCT）的CSV Hive外部表时发生错误。[#20233](https://github.com/StarRocks/starrocks/pull/20233)
- 使用Spark连接器的查询超时。[#20264](https://github.com/StarRocks/starrocks/pull/20264)
- 如果双副本表的一个副本已损坏，则该表无法恢复。[#20681](https://github.com/StarRocks/starrocks/pull/20681)
- MV查询重写失败导致的查询失败。[#19549](https://github.com/StarRocks/starrocks/issues/19549)
- 指标接口因数据库锁定而过期。[#20790](https://github.com/StarRocks/starrocks/pull/20790)
- 广播联接返回错误的结果。[#20952](https://github.com/StarRocks/starrocks/issues/20952)
- 当在CREATE TABLE中使用不支持的数据类型时，将返回NPE。[#20999](https://github.com/StarRocks/starrocks/issues/20999)
- 将window_funnel（）与查询缓存功能一起使用导致的问题。[#21474](https://github.com/StarRocks/starrocks/issues/21474)
- 重写CTE后，优化计划选择需要异常长的时间。[#16515](https://github.com/StarRocks/starrocks/pull/16515)

## 2.5.4

发布日期：2023年4月4日

### 改进

- 优化了查询规划时重写物化视图查询的性能。查询规划所花费的时间减少了约70%。[#19579](https://github.com/StarRocks/starrocks/pull/19579)
- 优化了类型推断逻辑。如果查询like `SELECT sum(CASE WHEN XXX);`包含常量`0`，例如`SELECT sum(CASE WHEN k1 = 1 THEN v1 ELSE 0 END) FROM test;`，则会自动启用预聚合以加速查询。[#19474](https://github.com/StarRocks/starrocks/pull/19474)
- 支持用于`SHOW CREATE VIEW`查看物化视图的创建语句。[#19999](https://github.com/StarRocks/starrocks/pull/19999)
- 支持在BE节点之间传输单个bRPC请求的2GB或更大数据包。#20283[ ](https://github.com/StarRocks/starrocks/pull/20283)#20230[ ](https://github.com/StarRocks/starrocks/pull/20230)
- 支持使用[SHOW CREATE CATALOG](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)查询外部目录的创建语句。

### Bug修复

修复了以下错误：

- 重写物化视图查询后，低基数优化的全局字典不生效。[#19615](https://github.com/StarRocks/starrocks/pull/19615)
- 如果重写实例化视图上的查询失败，则查询将失败。[#19774](https://github.com/StarRocks/starrocks/pull/19774)
- 如果基于“主键”或“唯一键”表创建实例化视图，则无法重写对该实例化视图的查询。[#19600](https://github.com/StarRocks/starrocks/pull/19600)
- 实例化视图的列名区分大小写。但是，在创建表时，即使表创建语句中的列名不正确，也会成功创建该表，而不会显示错误消息。此外，重写在该表上创建的实例化视图上的查询也会失败。[#19780](https://github.com/StarRocks/starrocks/pull/19780)
- 重写实例化视图上的查询后，查询计划包含基于分区列的无效谓词，这会影响查询性能。[#19784](https://github.com/StarRocks/starrocks/pull/19784)
- 将数据加载到新创建的分区中时，对实例化视图的查询可能无法重写。[#20323](https://github.com/StarRocks/starrocks/pull/20323)
- 在创建物化视图时配置 `"storage_medium" = "SSD"` 会导致物化视图的刷新失败。 [#19539](https://github.com/StarRocks/starrocks/pull/19539) [#19626](https://github.com/StarRocks/starrocks/pull/19626)
- 主键表上可能会发生并发压缩。 [#19692](https://github.com/StarRocks/starrocks/pull/19692)
- 在大量执行 DELETE 操作后，不会立即进行压缩。 [#19623](https://github.com/StarRocks/starrocks/pull/19623)
- 如果语句的表达式包含多个低基数列，则表达式可能无法正确重写。因此，低基数优化的全局字典不会生效。 [#20161](https://github.com/StarRocks/starrocks/pull/20161)

## 2.5.3

发布日期：2023年3月10日

### 改进

- 优化了物化视图（MVs）的查询重写。
  - 支持使用 Outer Join 和 Cross Join 重写查询。 [#18629](https://github.com/StarRocks/starrocks/pull/18629)
  - 优化了MV的数据扫描逻辑，进一步加速了重写查询。 [#18629](https://github.com/StarRocks/starrocks/pull/18629)
  - 增强了单表聚合查询的重写功能。  [#18629](https://github.com/StarRocks/starrocks/pull/18629)
  - 增强了在 View Delta 场景中的重写功能，即查询的表是MV基表的子集。 [#18800](https://github.com/StarRocks/starrocks/pull/18800)
- 优化了窗口函数 RANK() 作为过滤器或排序键时的性能和内存使用。 [#17553](https://github.com/StarRocks/starrocks/issues/17553)

### Bug修复

已修复以下错误：

- 由于 ARRAY数据中的 null 文本 `[]` 导致的错误。[#18563](https://github.com/StarRocks/starrocks/pull/18563)
- 在一些复杂的查询场景中滥用低基数优化字典。现在，在应用字典之前添加了字典映射检查。  [#17318](https://github.com/StarRocks/starrocks/pull/17318)
- 在单个BE环境中，Local Shuffle会导致GROUP BY产生重复的结果。 [#17845](https://github.com/StarRocks/starrocks/pull/17845)
- 对未分区的MV误用与分区相关的PROPERTIES可能会导致MV刷新失败。现在，当用户创建MV时，将执行分区PROPERTIES检查。 [#18741](https://github.com/StarRocks/starrocks/pull/18741)
- 解析Parquet重复列时出错。 #[17626](https://github.com/StarRocks/starrocks/pull/17626) [ #17788](https://github.com/StarRocks/starrocks/pull/17788) [#18051](https://github.com/StarRocks/starrocks/pull/18051)
- 获取的列的可为null信息不正确。解决方案：使用CTAS创建主键表时，只有主键列不可为空；非主键列可为null。 [#16431](https://github.com/StarRocks/starrocks/pull/16431)
- 从主键表中删除数据导致的一些问题。  [#18768](https://github.com/StarRocks/starrocks/pull/18768)

## 2.5.2

发布日期：2023年2月21日

### 新功能

- 支持使用实例配置文件和基于假定角色的凭证方法访问AWS S3和AWS Glue。 [#15958](https://github.com/StarRocks/starrocks/pull/15958)
- 支持以下位函数：bit_shift_left、bit_shift_right和bit_shift_right_logical。 [#14151](https://github.com/StarRocks/starrocks/pull/14151)

### 改进

- 优化了内存释放逻辑，显著降低了查询中包含大量聚合查询时的峰值内存使用量。 [#16913](https://github.com/StarRocks/starrocks/pull/16913)
- 减少了排序的内存使用量。当查询涉及窗口函数或排序时，内存消耗减半。#16937 [  #](https://github.com/StarRocks/starrocks/pull/16937)17362[ ](https://github.com/StarRocks/starrocks/pull/17362)#17408[](https://github.com/StarRocks/starrocks/pull/17408)

### Bug修复

已修复以下错误：

- 无法刷新包含MAP和ARRAY数据的Apache Hive外部表。 [#17548](https://github.com/StarRocks/starrocks/pull/17548)
- Superset无法识别物化视图的列类型。 [#17686](https://github.com/StarRocks/starrocks/pull/17686)
- BI连接失败，因为无法解析SET GLOBAL/SESSION TRANSACTION。 [#17295](https://github.com/StarRocks/starrocks/pull/17295)
- 无法修改共置组中动态分区表的桶数，并返回错误信息。 [#17418](https://github.com/StarRocks/starrocks/pull/17418/)
- “准备”阶段失败导致的潜在问题。 [#17323](https://github.com/StarRocks/starrocks/pull/17323)

### 行为更改

- 将默认值 `enable_experimental_mv` 从 `false` 更改为 `true`，这意味着默认情况下启用了异步物化视图。
- 将 CHARACTER 添加到保留关键字列表中。 [#17488](https://github.com/StarRocks/starrocks/pull/17488)

## 2.5.1

发布日期：2023年2月5日

### 改进

- 基于外部目录创建的异步物化视图支持查询重写。 #[11116](https://github.com/StarRocks/starrocks/issues/11116) [#15791](https://github.com/StarRocks/starrocks/issues/15791)
- 支持用户自定义CBO统计信息自动采集的采集周期，避免自动全量采集导致的集群性能抖动。 [#14996](https://github.com/StarRocks/starrocks/pull/14996)
- 添加了Thrift服务器队列。在INSERT INTO SELECT期间无法立即处理的请求可以在Thrift服务器队列中挂起，从而防止请求被拒绝。 [#14571](https://github.com/StarRocks/starrocks/pull/14571)
- 弃用了FE参数 `default_storage_medium`。如果用户创建表时未明确指定 `storage_medium`，系统会根据BE磁盘类型自动推断表的存储介质。有关详细信息，请参阅[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_VIEW.md)中的`storage_medium`说明。 [#14394](https://github.com/StarRocks/starrocks/pull/14394)

### Bug修复

已修复以下错误：

- 由SET PASSWORD导致的空指针异常（NPE）。 [#15247](https://github.com/StarRocks/starrocks/pull/15247)
- 无法解析具有空键的JSON数据。 [#16852](https://github.com/StarRocks/starrocks/pull/16852)
- 无效类型的数据可以成功转换为ARRAY数据。 [#16866](https://github.com/StarRocks/starrocks/pull/16866)
- 异常发生时，嵌套循环联接无法中断。  [#16875](https://github.com/StarRocks/starrocks/pull/16875)

### 行为更改

- 弃用了FE参数 `default_storage_medium`。表的存储介质由系统自动推断。 [#14394](https://github.com/StarRocks/starrocks/pull/14394)

## 2.5.0

发布日期：2023年1月22日

### 新功能

- 支持使用[Hudi目录](../data_source/catalog/hudi_catalog.md)和[Hudi外部表](../data_source/External_table.md#deprecated-hudi-external-table)查询Merge On Read表。 [#6780](https://github.com/StarRocks/starrocks/pull/6780)
- 支持使用[Hive目录](../data_source/catalog/hive_catalog.md)、Hudi目录和[Iceberg目录](../data_source/catalog/iceberg_catalog.md)查询STRUCT和MAP数据。 [#10677](https://github.com/StarRocks/starrocks/issues/10677)
- 提供[数据缓存](../data_source/data_cache.md)，提升HDFS等外部存储系统热数据的访问性能。 [#11597](https://github.com/StarRocks/starrocks/pull/11579)
- 支持创建[Delta Lake目录](../data_source/catalog/deltalake_catalog.md)，允许直接查询Delta Lake中的数据。 [#11972](https://github.com/StarRocks/starrocks/issues/11972)
- Hive、Hudi和Iceberg目录与AWS Glue兼容。 [#12249](https://github.com/StarRocks/starrocks/issues/12249)
- 支持创建[文件外部表](../data_source/file_external_table.md)，允许从HDFS和对象存储直接查询Parquet和ORC文件。 [#13064](https://github.com/StarRocks/starrocks/pull/13064)
- 支持基于Hive、Hudi、Iceberg目录和物化视图创建物化视图。有关详细信息，请参阅[物化视图](../using_starrocks/Materialized_view.md)。 [#11116](https://github.com/StarRocks/starrocks/issues/11116) [#11873](https://github.com/StarRocks/starrocks/pull/11873)
- 支持对使用主键表的表进行条件更新。有关详细信息，请参阅[通过加载更改数据](../loading/Load_to_Primary_Key_tables.md)。 [#12159](https://github.com/StarRocks/starrocks/pull/12159)
- 支持[查询缓存](../using_starrocks/query_cache.md)，存储查询的中间计算结果，提高QPS，降低高并发、简单查询的平均时延。 [#9194](https://github.com/StarRocks/starrocks/pull/9194)
- 支持指定Broker Load作业的优先级。有关更多信息，请参阅[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) [#11029](https://github.com/StarRocks/starrocks/pull/11029)
- 支持指定 StarRocks 原生表数据加载的副本数。有关更多信息，请参阅 [CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)。 [#11253](https://github.com/StarRocks/starrocks/pull/11253)
- 支持[查询队列](../administration/query_queues.md)。 [#12594](https://github.com/StarRocks/starrocks/pull/12594)
- 支持隔离数据加载占用的计算资源，从而限制数据加载任务的资源消耗。有关详细信息，请参阅 [资源组](../administration/resource_group.md)。 [#12606](https://github.com/StarRocks/starrocks/pull/12606)
- 支持指定以下数据压缩算法用于 StarRocks 原生表：LZ4、Zstd、Snappy 和 Zlib。有关详细信息，请参阅[数据压缩](../table_design/data_compression.md)。 [#10097](https://github.com/StarRocks/starrocks/pull/10097) [#12020](https://github.com/StarRocks/starrocks/pull/12020)
- 支持[用户定义的变量](../reference/user_defined_variables.md)。 [#10011](https://github.com/StarRocks/starrocks/pull/10011)
- 支持[lambda表达式](../sql-reference/sql-functions/Lambda_expression.md)和以下高阶函数：[array_map](../sql-reference/sql-functions/array-functions/array_map.md)、[array_sum](../sql-reference/sql-functions/array-functions/array_sum.md) 和 [array_sortby](../sql-reference/sql-functions/array-functions/array_sortby.md)。 [#9461](https://github.com/StarRocks/starrocks/pull/9461) [#9806](https://github.com/StarRocks/starrocks/pull/9806) [#10323](https://github.com/StarRocks/starrocks/pull/10323) [#14034](https://github.com/StarRocks/starrocks/pull/14034)
- 提供 QUALIFY 子句，用于筛选[窗口函数的结果](../sql-reference/sql-functions/Window_function.md)。 [#13239](https://github.com/StarRocks/starrocks/pull/13239)
- 支持在创建表时将 uuid() 和 uuid_numeric() 函数返回的结果用作列的默认值。有关更多信息，请参阅 [CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)。 [#11155](https://github.com/StarRocks/starrocks/pull/11155)
- 支持以下函数：[map_size](../sql-reference/sql-functions/map-functions/map_size.md)、[map_keys](../sql-reference/sql-functions/map-functions/map_keys.md)、[map_values](../sql-reference/sql-functions/map-functions/map_values.md)、[max_by](../sql-reference/sql-functions/aggregate-functions/max_by.md)、[sub_bitmap](../sql-reference/sql-functions/bitmap-functions/sub_bitmap.md)、[bitmap_to_base64](../sql-reference/sql-functions/bitmap-functions/bitmap_to_base64.md)、[host_name](../sql-reference/sql-functions/utility-functions/host_name.md) 和 [date_slice](../sql-reference/sql-functions/date-time-functions/date_slice.md)。 [#11299](https://github.com/StarRocks/starrocks/pull/11299) [#11323](https://github.com/StarRocks/starrocks/pull/11323) [#12243](https://github.com/StarRocks/starrocks/pull/12243) [#11776](https://github.com/StarRocks/starrocks/pull/11776) [#12634](https://github.com/StarRocks/starrocks/pull/12634) [#14225](https://github.com/StarRocks/starrocks/pull/14225)

### 改进

- 优化了使用 Hive 目录、Hudi 目录和 Iceberg 目录查询外部数据时的元数据访问性能。 [#11349](https://github.com/StarRocks/starrocks/issues/11349)
- 支持使用 Elasticsearch 外部表查询 ARRAY 数据。 [#9693](https://github.com/StarRocks/starrocks/pull/9693)
- 优化物化视图的以下几个方面：
  - 异步物化视图支持基于 SPJG 类型的物化视图自动、透明地重写查询。有关详细信息，请参阅 [实例化视图](../using_starrocks/Materialized_view.md#rewrite-and-accelerate-queries-with-the-asynchronous-materialized-view)。 [#13193](https://github.com/StarRocks/starrocks/issues/13193)
  - 异步实例化视图支持多种异步刷新机制。有关详细信息，请参阅[实例化视图](../using_starrocks/Materialized_view.md#manually-refresh-an-asynchronous-materialized-view)。 [#12712](https://github.com/StarRocks/starrocks/pull/12712) [#13171](https://github.com/StarRocks/starrocks/pull/13171) [#13229](https://github.com/StarRocks/starrocks/pull/13229) [#12926](https://github.com/StarRocks/starrocks/pull/12926)
  - 提高了刷新物化视图的效率。 [#13167](https://github.com/StarRocks/starrocks/issues/13167)
- 优化数据加载的以下几个方面：
  - 通过支持“单主复制”模式，优化了多副本场景下的加载性能。数据加载可将性能提升一倍。有关“单主复制”的详细信息，请参见 `replicated_storage` [CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)。 [#10138](https://github.com/StarRocks/starrocks/pull/10138)
  - 当只配置了一个 HDFS 集群或一个 Kerberos 用户时，Broker Load 和 Spark Load 不再需要依赖 Broker 进行数据加载。但是，如果您有多个 HDFS 集群或多个 Kerberos 用户，您仍然需要部署一个代理。有关更多信息，请参阅[从 HDFS 或云存储加载数据](../loading/BrokerLoad.md)和[使用 Apache Spark™ 批量加载](../loading/SparkLoad.md)。 [#9049](https://github.com/starrocks/starrocks/pull/9049) [#9228](https://github.com/StarRocks/starrocks/pull/9228)
  - 优化 Broker Load 加载大量小型 ORC 文件时的性能。 [#11380](https://github.com/StarRocks/starrocks/pull/11380)
  - 减少了将数据加载到主键表中的内存使用量。
- 优化了 `information_schema` 数据库和其中的 `tables` and `columns` 表。添加一个新表 `table_config`。有关详细信息，请参阅 [信息架构](../reference/overview-pages/information_schema.md)。 [#10033](https://github.com/StarRocks/starrocks/pull/10033)
- 优化数据备份和恢复：
  - 支持一次备份和恢复数据库中的多个表的数据。具体操作，请参见 [备份和恢复数据](../administration/Backup_and_restore.md)。 [#11619](https://github.com/StarRocks/starrocks/issues/11619)
  - 支持备份和恢复主键表中的数据。有关详细信息，请参阅备份和还原。 [#11885](https://github.com/StarRocks/starrocks/pull/11885)
- 优化以下功能：
  - 新增 [time_slice](../sql-reference/sql-functions/date-time-functions/time_slice.md) 函数可选参数，用于判断返回时间间隔的开始还是结束。 [#11216](https://github.com/StarRocks/starrocks/pull/11216)
  - 为 `INCREASE`window_funnel[ 函数添加了新模式 ](../sql-reference/sql-functions/aggregate-functions/window_funnel.md) ，以避免计算重复的时间戳。 [#10134](https://github.com/StarRocks/starrocks/pull/10134)
  - 支持在 unnest[ 函数 ](../sql-reference/sql-functions/array-functions/unnest.md) 中指定多个参数。[#12484](https://github.com/StarRocks/starrocks/pull/12484)
  - lead（） 和 lag（） 函数支持查询 HLL 和 BITMAP 数据。有关详细信息，请参阅 [窗口函数](../sql-reference/sql-functions/Window_function.md)。 [#12108](https://github.com/StarRocks/starrocks/pull/12108)
  - 以下 ARRAY 函数支持查询 JSON 数据：[array_agg、](../sql-reference/sql-functions/array-functions/array_agg.md)array_sort[、array_concat](../sql-reference/sql-functions/array-functions/array_sort.md)[、](../sql-reference/sql-functions/array-functions/array_concat.md)array_slice[和](../sql-reference/sql-functions/array-functions/array_slice.md)反向[。 ](../sql-reference/sql-functions/array-functions/reverse.md) [#13155](https://github.com/StarRocks/starrocks/pull/13155)
  - 优化部分功能使用。、 `current_date` `current_timestamp` `current_time` `localtimestamp` `localtime` 、`()`、 `select current_date;` [#14319](https://github.com/StarRocks/starrocks/pull/14319)
- 从 FE 日志中删除了一些冗余信息。 [#15374](https://github.com/StarRocks/starrocks/pull/15374)

### Bug 修复

修复了以下错误：

- 当第一个参数为空时，append_trailing_char_if_absent（） 函数可能会返回不正确的结果。 [#13762](https://github.com/StarRocks/starrocks/pull/13762)
- 使用RECOVER语句恢复表后，该表将不存在。 [#13921](https://github.com/StarRocks/starrocks/pull/13921)
- SHOW CREATE MATERIALIZED VIEW 语句返回的结果不包含创建实例化视图时查询语句中指定的数据库和目录。 [#12833](https://github.com/StarRocks/starrocks/pull/12833)
- 无法取消状态  `waiting_stable`中的架构更改作业。[#12530](https://github.com/StarRocks/starrocks/pull/12530)
-  `SHOW PROC '/statistic';` 在 Leader FE 和非 Leader FE 上运行该命令会返回不同的结果。 [#12491](https://github.com/StarRocks/starrocks/issues/12491)
- ORDER BY 子句在 SHOW CREATE TABLE 返回的结果中的位置不正确。 [#13809](https://github.com/StarRocks/starrocks/pull/13809)
- 用户使用 Hive Catalog 查询 Hive 数据时，如果 FE 生成的执行计划不包含分区 ID，则 BE 无法查询到 Hive 分区数据。 [#15486](https://github.com/StarRocks/starrocks/pull/15486)。

### 行为改变

- 将参数的默认值更改为 `AWS_EC2_METADATA_DISABLED` `False`，表示获取 Amazon EC2 的元数据以访问 AWS 资源。
- 将会话变量 `is_report_success` 重命名为 `enable_profile`，可使用 SHOW VARIABLES 语句进行查询。
- 添加了四个保留关键字：`CURRENT_DATE`、`CURRENT_TIME`、`LOCALTIME` 和 `LOCALTIMESTAMP`。 [# 14319](https://github.com/StarRocks/starrocks/pull/14319)
- 表名和数据库名的最大长度可达 1023 个字符。 [# 14929](https://github.com/StarRocks/starrocks/pull/14929) [# 15020](https://github.com/StarRocks/starrocks/pull/15020)
- BE配置项 `enable_event_based_compaction_framework` 和 `enable_size_tiered_compaction_strategy` 默认设置为 `true`，当存在大量Tablet或单个Tablet数据量较大时，可显著降低压缩开销。

### 升级说明

- 您可以将集群从 2.0.x、2.1.x、2.2.x、2.3.x 或 2.4.x 升级到 2.5.0。但是，如果需要执行回滚，建议仅回滚到 2.4.x。