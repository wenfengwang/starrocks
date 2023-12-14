---
displayed_sidebar: "Chinese"
---

# StarRocks 版本 2.5

## 2.5.16

发布日期：2023年12月1日

### Bug 修复

修复了以下问题：

- 在某些情况下，全局 Runtime Filter 可能导致 BEs 崩溃。[#35776](https://github.com/StarRocks/starrocks/pull/35776)

## 2.5.15

发布日期：2023年11月29日

### 改进

- 添加了慢请求日志以跟踪慢请求。[#33908](https://github.com/StarRocks/starrocks/pull/33908)
- 在存在大量文件时，优化了使用 Spark Load 读取 Parquet 和 ORC 文件的性能。[#34787](https://github.com/StarRocks/starrocks/pull/34787)
- 优化了一些与 Bitmap 相关的操作的性能，包括：
  - 优化了嵌套循环连接。[#340804](https://github.com/StarRocks/starrocks/pull/34804) [#35003](https://github.com/StarRocks/starrocks/pull/35003)
  - 优化了 `bitmap_xor` 函数。[#34069](https://github.com/StarRocks/starrocks/pull/34069)
  - 支持写时复制以优化 Bitmap 性能并减少内存消耗。[#34047](https://github.com/StarRocks/starrocks/pull/34047)

### 兼容性变更

#### 参数

- FE 动态参数 `enable_new_publish_mechanism` 更改为静态参数。修改参数设置后，必须重新启动 FE。[#35338](https://github.com/StarRocks/starrocks/pull/35338)

### Bug 修复

- 如果在 Broker Load 作业中指定了筛选条件，则在数据加载过程中，BEs 可能会崩溃。[#29832](https://github.com/StarRocks/starrocks/pull/29832)
- 在重放复制操作失败时，可能导致 FEs 崩溃。[#32295](https://github.com/StarRocks/starrocks/pull/32295)
- 将 FE 参数 `recover_with_empty_tablet` 设置为 `true` 可能导致 FEs 崩溃。[#33071](https://github.com/StarRocks/starrocks/pull/33071)
- 查询时返回错误 "get_applied_rowsets failed, tablet updates is in error state: tablet:18849 actual row size changed after compaction"。[#33246](https://github.com/StarRocks/starrocks/pull/33246)
- 包含窗口函数的查询可能导致 BEs 崩溃。[#33671](https://github.com/StarRocks/starrocks/pull/33671)
- 运行 `show proc '/statistic'` 可能导致死锁。[#34237](https://github.com/StarRocks/starrocks/pull/34237/files)
- 如果大量数据加载到启用了持久性索引的主键表中，可能会抛出错误。[#34566](https://github.com/StarRocks/starrocks/pull/34566)
- 从 v2.4 或更早的版本升级到更高版本后，压实分数可能出现意外上升。[#34618](https://github.com/StarRocks/starrocks/pull/34618)
- 如果使用数据库驱动程序 MariaDB ODBC 查询 `INFORMATION_SCHEMA`，`schemata` 视图返回的 `CATALOG_NAME` 列仅包含 `null` 值。[#34627](https://github.com/StarRocks/starrocks/pull/34627)
- 在执行流加载作业处于 **PREPARD** 状态时执行模式更改，会导致作业要加载的部分源数据丢失。[#34381](https://github.com/StarRocks/starrocks/pull/34381)
- 在 HDFS 存储路径末尾包含两个或更多斜杠(`/`)，将导致从 HDFS 备份和还原数据失败。[#34601](https://github.com/StarRocks/starrocks/pull/34601)
- 运行加载任务或查询可能会导致 FEs 挂起。[#34569](https://github.com/StarRocks/starrocks/pull/34569)

## 2.5.14

发布日期：2023年11月14日

### 改进

- 系统数据库 `INFORMATION_SCHEMA` 中的 `COLUMNS` 表可以显示 ARRAY、MAP 和 STRUCT 列。[#33431](https://github.com/StarRocks/starrocks/pull/33431)

### 兼容性更改

#### 系统变量

- 添加了会话变量 `cbo_decimal_cast_string_strict`，用于控制 CBO 如何将 DECIMAL 类型转换为 STRING 类型的数据。如果将此变量设置为 `true`，则 v2.5.x 和更高版本中内置的逻辑将起作用，并且系统执行严格转换（即，系统截断生成的字符串，并根据比例长度填充 0）。如果将此变量设置为 `false`，则 v2.5.x 之前版本中内置的逻辑将起作用，并且系统会处理所有有效数字以生成字符串。默认值为 `true`。[#34208](https://github.com/StarRocks/starrocks/pull/34208)
- 添加了会话变量 `cbo_eq_base_type`，用于指定在 DECIMAL 类型数据和 STRING 类型数据之间进行数据比较时使用的数据类型。默认值为 `VARCHAR`，也可使用 DECIMAL。[#34208](https://github.com/StarRocks/starrocks/pull/34208)

### Bug 修复

修复了以下问题：

- 如果 ON 条件与子查询嵌套，则报告错误 `java.lang.IllegalStateException: null`。[#30876](https://github.com/StarRocks/starrocks/pull/30876)
- 如果在成功执行 `INSERT INTO SELECT ... LIMIT` 后立即运行 COUNT(*)，则不同副本之间的 COUNT(*) 结果不一致。[#24435](https://github.com/StarRocks/starrocks/pull/24435)
- 如果在 cast() 函数中指定的目标数据类型与原始数据类型相同，则对特定数据类型会导致 BE 崩溃。[#31465](https://github.com/StarRocks/starrocks/pull/31465)
- 如果在通过 Broker Load 加载数据时使用特定路径格式，则报告错误 `msg:Fail to parse columnsFromPath, expected: [rec_dt]`。[#32721](https://github.com/StarRocks/starrocks/issues/32721)
- 在升级到 3.x 时，如果某些列类型也已升级（例如，Decimal 升级为 Decimal v3），则在具有特定特性的表上执行 Compaction 时，BEs 会崩溃。[#31626](https://github.com/StarRocks/starrocks/pull/31626)
- 使用 Flink Connector 加载数据时，如果存在高度并发的加载作业，并且 HTTP 和扫描线程数都达到上限，则加载作业会意外中止。[#32251](https://github.com/StarRocks/starrocks/pull/32251)
- 在调用 libcurl 时，BEs 崩溃。[#31667](https://github.com/StarRocks/starrocks/pull/31667)
- 向主键表添加 BITMAP 列失败，并显示以下错误：`Analyze columnDef error: No aggregate function specified for 'userid'`。[#31763](https://github.com/StarRocks/starrocks/pull/31763)
- 启用了持久索引的主键表中长时间频繁加载数据可能导致 BEs 崩溃。[#33220](https://github.com/StarRocks/starrocks/pull/33220)
- 启用查询缓存后，查询结果不正确。[#32778](https://github.com/StarRocks/starrocks/pull/32778)
- 在创建主键表时指定可空排序键会导致压实失败。[#29225](https://github.com/StarRocks/starrocks/pull/29225)
- 对于复杂的连接查询，偶尔会出现错误 "StarRocks planner use long time 10000 ms in logical phase"。[#34177](https://github.com/StarRocks/starrocks/pull/34177)

## 2.5.13

发布日期：2023年9月28日

### 改进

- 窗口函数 COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD 和 STDDEV_SAMP 现在支持 ORDER BY 子句和 WINDOW 子句。[#30786](https://github.com/StarRocks/starrocks/pull/30786)
- 如果查询 DECIMAL 类型数据时发生十进制溢出，则返回错误而不是 NULL。[#30419](https://github.com/StarRocks/starrocks/pull/30419)
- 现在执行具有无效注释的 SQL 命令返回与 MySQL 一致的结果。[#30210](https://github.com/StarRocks/starrocks/pull/30210)
- 删除已删除的与小区对应的 rowsets，减少了 BE 启动期间的内存使用。[#30625](https://github.com/StarRocks/starrocks/pull/30625)

### 修复的错误

已解决以下问题：

- Spark Connector或Flink Connector从StarRocks读取数据时出现“Set cancelled by MemoryScratchSinkOperator”错误。[#30702](https://github.com/StarRocks/starrocks/pull/30702) [#30751](https://github.com/StarRocks/starrocks/pull/30751)
- 使用包含聚合函数的ORDER BY子句进行查询时出现“java.lang.IllegalStateException: null”错误。[#30108](https://github.com/StarRocks/starrocks/pull/30108)
- 在存在非活动物化视图时，FE无法重新启动。[#30015](https://github.com/StarRocks/starrocks/pull/30015)
- 对重复分区执行INSERT OVERWRITE操作会损坏元数据，导致FE重新启动失败。[#27545](https://github.com/StarRocks/starrocks/pull/27545)
- 用户在修改主键表中不存在的列时会出现“java.lang.NullPointerException: null”错误。[#30366](https://github.com/StarRocks/starrocks/pull/30366)
- 在用户将数据加载到分区化StarRocks外部表时会出现“从TNetworkAddress获取TableMeta失败”错误。[#30124](https://github.com/StarRocks/starrocks/pull/30124)
- 在某些情况下，用户通过CloudCanal加载数据时会出现错误。[#30799](https://github.com/StarRocks/starrocks/pull/30799)
- 当用户通过Flink Connector加载数据或执行DELETE和INSERT操作时，会出现“当前数据库xxx上运行的txns为200，大于限制200”错误。[#18393](https://github.com/StarRocks/starrocks/pull/18393)
- 使用包含聚合函数的HAVING子句的异步物化视图无法正确重写查询。[#29976](https://github.com/StarRocks/starrocks/pull/29976)

## 2.5.12

发布日期：2023年9月4日

### 改进

- 在审计日志中保留SQL中的注释。[#29747](https://github.com/StarRocks/starrocks/pull/29747)
- 将INSERT INTO SELECT的CPU和内存统计信息添加到审计日志中。[#29901](https://github.com/StarRocks/starrocks/pull/29901)

### 修复的错误

已解决以下问题：

- 使用Broker Load加载数据时，某些字段的NOT NULL属性可能导致BE崩溃或出现“msg:mismatched row count”错误。[#29832](https://github.com/StarRocks/starrocks/pull/29832)
- 对ORC格式文件的查询失败，因为未合并Apache ORC的bug修复ORC-1304（[apache/orc#1299](https://github.com/apache/orc/pull/1299)）。[#29804](https://github.com/StarRocks/starrocks/pull/29804)
- 在BE重新启动后，恢复主键表会导致元数据不一致。[#30135](https://github.com/StarRocks/starrocks/pull/30135)

## 2.5.11

发布日期：2023年8月28日

### 改进

- 支持所有复合谓词和WHERE子句中所有表达式的隐式转换。可以通过使用[会话变量](https://docs.starrocks.io/zh-cn/3.1/reference/System_variable) `enable_strict_type`启用或禁用隐式转换。默认值为`false`。[#21870](https://github.com/StarRocks/starrocks/pull/21870)
- 在创建Iceberg Catalog时，如果用户未指定`hive.metastore.uri`，优化了错误提示。错误提示更准确。[#16543](https://github.com/StarRocks/starrocks/issues/16543)
- 在错误消息“xxx too many versions xxx”中添加更多提示。[#28397](https://github.com/StarRocks/starrocks/pull/28397)
- 动态分区进一步支持`year`作为分区单元。[#28386](https://github.com/StarRocks/starrocks/pull/28386)

### 修复的错误

已解决以下问题:

- 当数据加载到具有多个副本的表中时，如果表的某些分区为空，则会写入大量无效的日志记录。[#28824](https://github.com/StarRocks/starrocks/issues/28824)
- 如果WHERE条件中的字段为BITMAP或HLL字段，则DELETE操作会失败。[#28592](https://github.com/StarRocks/starrocks/pull/28592)
- 通过同步调用（SYNC MODE）手动刷新异步物化视图会导致`information_schema.task_runs`表中出现多个INSERT OVERWRITE记录。[#28060](https://github.com/StarRocks/starrocks/pull/28060)
- 如果在ERROR状态的分片上触发CLONE操作，磁盘使用量会增加。[#28488](https://github.com/StarRocks/starrocks/pull/28488)

... (remaining content omitted)
- 当使用CAST将字符串转换为数组时，如果输入包含常量，则结果可能不正确。[#19793](https://github.com/StarRocks/starrocks/pull/19793)
- 如果SHOW TABLET包含ORDER BY和LIMIT，则返回的结果可能不正确。[#23375](https://github.com/StarRocks/starrocks/pull/23375)
- 外连接和反连接为物化视图重写错误。[#28028](https://github.com/StarRocks/starrocks/pull/28028)
- FE中表级扫描统计信息不准确，导致表查询和加载的指标不准确。[#27779](https://github.com/StarRocks/starrocks/pull/27779)
- 如果事件监听器配置在HMS上用于增量更新Hive元数据，则在FE日志中报告“使用当前长链接访问元存储时发生异常。消息: Failed to get next notification based on last event id: 707602”。[#21056](https://github.com/StarRocks/starrocks/pull/21056)
- 如果对分区表修改了排序键，则查询结果可能不稳定。[#27850](https://github.com/StarRocks/starrocks/pull/27850)
- 如果使用Spark Load加载数据，则DATE、DATETIME或DECIMAL列作为分桶列时，数据可能分布到错误的桶中。[#27005](https://github.com/StarRocks/starrocks/pull/27005)
- 在某些情况下，regex_replace函数可能导致BE崩溃。[#27117](https://github.com/StarRocks/starrocks/pull/27117)
- 如果sub_bitmap函数的输入不是BITMAP值，则BE会崩溃。[#27982](https://github.com/StarRocks/starrocks/pull/27982)
- 如果启用连接重新排序，则对于查询返回“未知错误”。[#27472](https://github.com/StarRocks/starrocks/pull/27472)
- 平均行大小的估算不准确，导致主键部分更新占用过多内存。[#27485](https://github.com/StarRocks/starrocks/pull/27485)
- 如果启用低基数优化，则一些INSERT作业返回“[42000][1064]字典解码失败，字典无法覆盖所有键:0”。[#26463](https://github.com/StarRocks/starrocks/pull/26463)
- 如果用户在Broker Load作业中指定"hadoop.security.authentication"="simple"以从HDFS加载数据，则作业将失败。[#27774](https://github.com/StarRocks/starrocks/pull/27774)
- 修改物化视图的刷新模式导致主FE和从FE之间的元数据不一致。[#28082](https://github.com/StarRocks/starrocks/pull/28082) [#28097](https://github.com/StarRocks/starrocks/pull/28097)
- 当使用SHOW CREATE CATALOG和SHOW RESOURCES查询特定信息时，密码不会被隐藏。[#28059](https://github.com/StarRocks/starrocks/pull/28059)
- 受阻的LabelCleaner线程导致FE内存泄漏。[#28311](https://github.com/StarRocks/starrocks/pull/28311)

## 2.5.9

发布日期: 2023年7月19日

### 新功能

- 包含与物化视图不同类型的连接的查询可以被重写。[#25099](https://github.com/StarRocks/starrocks/pull/25099)

### 改进

- 当前StarRocks集群的目的地集群是当前StarRocks集群的StarRocks外部表无法创建。[#25441](https://github.com/StarRocks/starrocks/pull/25441)
- 如果查询字段未包含在物化视图的输出列中，但包含在物化视图的谓词中，则仍然可以重写查询。[#23028](https://github.com/StarRocks/starrocks/issues/23028)
- 在数据库"Information_schema"的表"tables_config"中添加了一个新字段`table_id`。您可以将`tables_config`与`be_tablets`在列`table_id`上进行连接，以查询一个tablet所属的数据库和表的名称。[#24061](https://github.com/StarRocks/starrocks/pull/24061)

### Bug修复

修复了以下问题:

- 重复键表的Count Distinct结果不正确。[#24222](https://github.com/StarRocks/starrocks/pull/24222)
- 如果连接键是大BINARY列，则BE可能会崩溃。[#25084](https://github.com/StarRocks/starrocks/pull/25084)
- 如果要插入的结构体中的CHAR数据长度超过了结构体列中定义的最大CHAR长度，则INSERT操作将挂起。[#25942](https://github.com/StarRocks/starrocks/pull/25942)
- coalesce()的结果不正确。[#26250](https://github.com/StarRocks/starrocks/pull/26250)
- 恢复数据后，tablet的版本号在BE和FE之间不一致。[#26518](https://github.com/StarRocks/starrocks/pull/26518/files)
- 对于恢复的表，无法自动创建分区。[#26813](https://github.com/StarRocks/starrocks/pull/26813)

## 2.5.8

发布日期: 2023年6月30日

### 改进

- 在向非分区表添加分区时，优化了报告的错误消息。[#25266](https://github.com/StarRocks/starrocks/pull/25266)
- 优化了表的[自动tablet分布策略](../table_design/Data_distribution.md#determine-the-number-of-buckets)。[#24543](https://github.com/StarRocks/starrocks/pull/24543)
- 优化了CREATE TABLE语句中的默认注释。[#24803](https://github.com/StarRocks/starrocks/pull/24803)
- 优化了异步物化视图的手动刷新。支持使用REFRESH MATERIALIZED VIEW WITH SYNC MODE语法同步调用物化视图刷新任务。[#25910](https://github.com/StarRocks/starrocks/pull/25910)

### Bug修复

修复了以下问题:

- 如果物化视图是基于Union结果构建的，异步物化视图的COUNT结果可能不准确。[#24460](https://github.com/StarRocks/starrocks/issues/24460)
- 当用户尝试强制重置root密码时，报告"未知错误"。[#25492](https://github.com/StarRocks/starrocks/pull/25492)
- 在少于三个活动的BE的集群上执行INSERT OVERWRITE时，显示不准确的错误消息。[#25314](https://github.com/StarRocks/starrocks/pull/25314)

## 2.5.7

发布日期: 2023年6月14日

### 新功能

- 可以使用`ALTER MATERIALIZED VIEW <mv_name> ACTIVE`手动激活未激活的物化视图。您可以使用此SQL命令激活其基表被删除然后重新创建的物化视图。有关详细信息，请参见[ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md)。[#24001](https://github.com/StarRocks/starrocks/pull/24001)
- StarRocks在创建表或添加分区时可以自动设置适当数量的tablet，无需手动操作。有关详细信息，请参见[确定tablet数](../table_design/Data_distribution.md#determine-the-number-of-buckets)。[#10614](https://github.com/StarRocks/starrocks/pull/10614)

### 改进

- 优化了用于外部表查询的Scan节点的I/O并发性，减少了内存使用量，并提高了从外部表加载数据的稳定性。[#23617](https://github.com/StarRocks/starrocks/pull/23617) [#23624](https://github.com/StarRocks/starrocks/pull/23624) [#23626](https://github.com/StarRocks/starrocks/pull/23626)
- 优化了Broker Load作业的错误消息。错误消息包含重试信息和错误文件的名称。[#18038](https://github.com/StarRocks/starrocks/pull/18038) [#21982](https://github.com/StarRocks/starrocks/pull/21982)
- 当CREATE TABLE超时时，优化了返回的错误消息，并添加了参数调整提示。[#24510](https://github.com/StarRocks/starrocks/pull/24510)
- 当ALTER TABLE因表状态不是Normal而失败时，优化了返回的错误消息。[#24381](https://github.com/StarRocks/starrocks/pull/24381)
- 忽略了CREATE TABLE语句中的全角空格。[#23885](https://github.com/StarRocks/starrocks/pull/23885)
- 优化了经纪人访问超时，以增加经纪负载作业的成功率。 [#22699](https://github.com/StarRocks/starrocks/pull/22699)
- 对于主键表，SHOW TABLET返回的`VersionCount`字段包含处于Pending状态的Rowset。 [#23847](https://github.com/StarRocks/starrocks/pull/23847)
- 优化了持久化索引策略。 [#22140](https://github.com/StarRocks/starrocks/pull/22140)

### Bug修复

已修复以下问题：

- 当用户将Parquet数据加载到StarRocks时，由于类型转换导致DATETIME值溢出，从而导致数据错误。 [#22356](https://github.com/StarRocks/starrocks/pull/22356)
- 动态分区禁用后，Bucket信息丢失。 [#22595](https://github.com/StarRocks/starrocks/pull/22595)
- 在CREATE TABLE语句中使用不受支持的属性导致空指针异常（NPE）。 [#23859](https://github.com/StarRocks/starrocks/pull/23859)
- 在`information_schema`中的表权限过滤变得无效。结果，用户可以查看他们没有权限的表。 [#23804](https://github.com/StarRocks/starrocks/pull/23804)
- SHOW TABLE STATUS返回的信息不完整。 [#24279](https://github.com/StarRocks/starrocks/issues/24279)
- 如果数据加载与模式更改同时发生，则有时会出现模式更改挂起。 [#23456](https://github.com/StarRocks/starrocks/pull/23456)
- RocksDB WAL刷新会阻止brpc工作器处理b线程，这会中断将高频数据加载到主键表的过程。 [#22489](https://github.com/StarRocks/starrocks/pull/22489)
- 在StarRocks中成功创建不支持的TIME类型列。 [#23474](https://github.com/StarRocks/starrocks/pull/23474)
- Materialized view Union重写失败。 [#22922](https://github.com/StarRocks/starrocks/pull/22922)

## 2.5.6

发布日期：2023年5月19日

### 改进

- 当INSERT INTO ... SELECT由于`thrift_server_max_worker_thread`值较小而过期时，优化了报告的错误消息。 [#21964](https://github.com/StarRocks/starrocks/pull/21964)
- 使用CTAS创建的表默认具有三个副本，与普通表的默认副本数一致。 [#22854](https://github.com/StarRocks/starrocks/pull/22854)

### Bug修复

- 截断分区失败，因为TRUNCATE操作对分区名称区分大小写。 [#21809](https://github.com/StarRocks/starrocks/pull/21809)
- 由于为物化视图创建临时分区失败，导致BE下架失败。 [#22745](https://github.com/StarRocks/starrocks/pull/22745)
- 无法将需要ARRAY值的动态FE参数设置为空数组。 [#22225](https://github.com/StarRocks/starrocks/pull/22225)
- 可能由于指定了`partition_refresh_number`属性，物化视图无法完全刷新。 [#21619](https://github.com/StarRocks/starrocks/pull/21619)
- SHOW CREATE TABLE屏蔽了云凭据信息，这导致内存中存在不正确的凭据信息。 [#21311](https://github.com/StarRocks/starrocks/pull/21311)
- 对通过外部表查询的一些ORC文件上的谓词无效。 [#21901](https://github.com/StarRocks/starrocks/pull/21901)
- 最小-最大过滤器无法正确处理列名中的小写和大写字母。 [#22626](https://github.com/StarRocks/starrocks/pull/22626)
- 后期物化导致查询复杂数据类型（STRUCT或MAP）出错。  [#22862](https://github.com/StarRocks/starrocks/pull/22862)
- 恢复主键表时出现的问题。 [#23384](https://github.com/StarRocks/starrocks/pull/23384)

## 2.5.5

发布日期：2023年4月28日

### 新功能

添加了用于监控主键表的分片状态的度量标准：

- 添加FE度量标准`err_state_metric`。
- 向`SHOW PROC '/statistic/'`的输出中添加了`ErrorStateTabletNum`列，以显示**err_state**分片的数量。
- 向`SHOW PROC '/statistic/<db_id>/'`的输出中添加了`ErrorStateTablets`列，以显示**err_state**分片的ID。

有关更多信息，请参见[SHOW PROC](../sql-reference/sql-statements/Administration/SHOW_PROC.md)。

### 改进

- 当添加多个BE时，优化了磁盘平衡速度。 [# 19418](https://github.com/StarRocks/starrocks/pull/19418)
- 优化了`storage_medium`的推断。当BE同时使用SSD和HDD作为存储设备，如果指定了`storage_cooldown_time`属性，StarRocks将`storage_medium`设置为`SSD`。否则，StarRocks将`storage_medium`设置为`HDD`。 [#18649](https://github.com/StarRocks/starrocks/pull/18649)
- 通过禁止从值列收集统计信息，优化了Unique Key表的性能。 [#19563](https://github.com/StarRocks/starrocks/pull/19563)

### Bug修复

- 对于Colocation表，可以使用类似`ADMIN SET REPLICA STATUS PROPERTIES ("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");`的语句手动将副本状态指定为`bad`。如果BE的数量少于或等于副本的数量，则无法修复损坏的副本。 [# 17876](https://github.com/StarRocks/starrocks/issues/17876)
- 启动BE后，其进程存在，但BE端口无法启用。 [# 19347](https://github.com/StarRocks/starrocks/pull/19347)
- 查询的结果可能是错误的，其子查询与窗口函数嵌套。 [# 19725](https://github.com/StarRocks/starrocks/issues/19725)
- 当MV首次刷新时，`auto_refresh_partitions_limit`不起作用。结果，所有分区都会被刷新。 [# 19759](https://github.com/StarRocks/starrocks/issues/19759)
- 查询CSV Hive外部表时，数组数据嵌套了MAP和STRUCT等复杂数据时，会出现错误。 [# 20233](https://github.com/StarRocks/starrocks/pull/20233)
- 使用Spark连接器的查询超时。 [# 20264](https://github.com/StarRocks/starrocks/pull/20264)
- 如果两个副本表中的一个副本损坏，则该表无法恢复。 [# 20681](https://github.com/StarRocks/starrocks/pull/20681)
- 由MV查询重写失败导致查询失败。 [# 19549](https://github.com/StarRocks/starrocks/issues/19549)
- 由于数据库锁定，度量接口过期。 [# 20790](https://github.com/StarRocks/starrocks/pull/20790)
- 对Broadcast Join返回错误的结果。 [# 20952](https://github.com/StarRocks/starrocks/issues/20952)
- 在CREATE TABLE中使用不受支持的数据类型会返回NPE。 [# 20999](https://github.com/StarRocks/starrocks/issues/20999)
- 使用Query Cache功能时，使用window_funnel()会出现问题。 [# 21474](https://github.com/StarRocks/starrocks/issues/21474)
- CTE重写后，优化计划选择花费了意外长的时间。 [# 16515](https://github.com/StarRocks/starrocks/pull/16515)

## 2.5.4

发布日期：2023年4月4日

### 改进

- 优化了查询计划期间对物化视图的查询重写性能。查询计划所需的时间减少约70％。 [#19579](https://github.com/StarRocks/starrocks/pull/19579)
- 优化了类型推断逻辑。如果查询中包含常量`0`（例如`SELECT sum(CASE WHEN k1 = 1 THEN v1 ELSE 0 END) FROM test;`）的查询如`SELECT sum(CASE WHEN XXX);`，则会自动启用预聚合以加速查询。 [#19474](https://github.com/StarRocks/starrocks/pull/19474)
- 支持使用`SHOW CREATE VIEW`查看物化视图的创建语句。 [#19999](https://github.com/StarRocks/starrocks/pull/19999)
- 支持在BE节点之间的单个bRPC请求中传输大小为2GB或更大的数据包。[#20283](https://github.com/StarRocks/starrocks/pull/20283) [#20230](https://github.com/StarRocks/starrocks/pull/20230)
- 支持使用[SHOW CREATE CATALOG](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)查询外部目录的创建语句。

### Bug修复

已修复以下bug：

- 在对物化视图进行重写查询后，低基数优化的全局字典不会生效。[#19615](https://github.com/StarRocks/starrocks/pull/19615)
- 如果对物化视图的查询无法被重写，该查询会失败。[#19774](https://github.com/StarRocks/starrocks/pull/19774)
- 如果基于主键或唯一键表创建物化视图，对该物化视图的查询无法进行重写。[#19600](https://github.com/StarRocks/starrocks/pull/19600)
- 物化视图的列名区分大小写。但是，当创建表时，表可以成功创建，即使在表创建语句的`PROPERTIES`中列名不正确，此外，对创建在该表上的物化视图的查询重写也会失败而不会出现错误消息。[#19780](https://github.com/StarRocks/starrocks/pull/19780)
- 对物化视图的查询重写后，查询计划可能包含基于分区列、无效谓词，影响查询性能。[#19784](https://github.com/StarRocks/starrocks/pull/19784)
- 当数据加载到新创建的分区时，对物化视图的查询可能无法重写。[#20323](https://github.com/StarRocks/starrocks/pull/20323)
- 在创建物化视图时配置`"storage_medium" = "SSD"`会导致物化视图的刷新失败。[#19539](https://github.com/StarRocks/starrocks/pull/19539) [#19626](https://github.com/StarRocks/starrocks/pull/19626)
- 主键表上可能发生并发压缩。[#19692](https://github.com/StarRocks/starrocks/pull/19692)
- 大量DELETE操作后可能无法立即进行压缩。[#19623](https://github.com/StarRocks/starrocks/pull/19623)
- 如果语句的表达式包含多个低基数列，则可能无法正确重写该表达式。因此，低基数优化的全局字典不会生效。[#20161](https://github.com/StarRocks/starrocks/pull/20161)

## 2.5.3

发布日期：2023年3月10日

### 改进

- 优化了对物化视图（MVs）的查询重写。
  - 支持重写带有外连接和交叉连接的查询。[#18629](https://github.com/StarRocks/starrocks/pull/18629)
  - 优化了MV的数据扫描逻辑，进一步加快了重写查询的速度。[#18629](https://github.com/StarRocks/starrocks/pull/18629)
  - 加强了对单表聚合查询的重写能力。[#18629](https://github.com/StarRocks/starrocks/pull/18629)
  - 加强了在视图增量方案中的重写能力，即查询表是MV的基表子集时。[#18800](https://github.com/StarRocks/starrocks/pull/18800)
- 优化了使用窗口函数RANK()作为过滤器或排序键时的性能和内存使用。[#17553](https://github.com/StarRocks/starrocks/issues/17553)

### Bug修复

已修复以下bug：

- 数组数据中`[]`空字面量引起的错误。[#18563](https://github.com/StarRocks/starrocks/pull/18563)
- 在一些复杂的查询场景中，对低基数优化字典的误用。现在，在应用字典之前添加了字典映射检查。[#17318](https://github.com/StarRocks/starrocks/pull/17318)
- 在单个BE环境中，本地Shuffle导致GROUP BY生成重复结果。[#17845](https://github.com/StarRocks/starrocks/pull/17845)
- 对非分区MV误用与分区相关的PROPERTIES可能导致MV刷新失败。用户在创建MV时现在会执行分区PROPERTIES检查。[#18741](https://github.com/StarRocks/starrocks/pull/18741)
- 解析Parquet重复列时发生的错误。[#17626](https://github.com/StarRocks/starrocks/pull/17626) [#17788](https://github.com/StarRocks/starrocks/pull/17788) [#18051](https://github.com/StarRocks/starrocks/pull/18051)
- 获取的列的可空信息不正确。解决方案：使用CTAS创建主键表时，只有主键列是不可为空的；非主键列是可为空的。[#16431](https://github.com/StarRocks/starrocks/pull/16431)
- 由于删除主键表中的数据而引起的一些问题。[#18768](https://github.com/StarRocks/starrocks/pull/18768)

## 2.5.2

发布日期：2023年2月21日

### 新特性

- 支持使用实例配置和假定角色凭据方法访问AWS S3和AWS Glue。[#15958](https://github.com/StarRocks/starrocks/pull/15958)
- 支持以下比特函数：bit_shift_left、bit_shift_right和bit_shift_right_logical。[#14151](https://github.com/StarRocks/starrocks/pull/14151)

### 改进

- 优化了内存释放逻辑，在查询中包含大量聚合查询时，极大降低了内存峰值使用情况。[#16913](https://github.com/StarRocks/starrocks/pull/16913)
- 减少了排序的内存使用。当查询涉及窗口函数或排序时，内存消耗减少了一半。[#16937](https://github.com/StarRocks/starrocks/pull/16937) [#17362](https://github.com/StarRocks/starrocks/pull/17362) [#17408](https://github.com/StarRocks/starrocks/pull/17408)

### Bug修复

已修复以下bug：

- 包含MAP和ARRAY数据的Apache Hive外部表无法刷新。[#17548](https://github.com/StarRocks/starrocks/pull/17548)
- Superset无法识别物化视图的列类型。[#17686](https://github.com/StarRocks/starrocks/pull/17686)
- BI连接失败，因为无法解析SET GLOBAL/SESSION TRANSACTION。[#17295](https://github.com/StarRocks/starrocks/pull/17295)
- 动态分区表的桶数在Colocate Group中无法修改并返回错误消息。[#17418](https://github.com/StarRocks/starrocks/pull/17418/)
- 在准备阶段失败可能引起的潜在问题。[#17323](https://github.com/StarRocks/starrocks/pull/17323)

### 行为变更

- 将`enable_experimental_mv`的默认值从`false`更改为`true`，这意味着异步物化视图默认已启用。
- 将CHARACTER添加到保留关键字列表。[#17488](https://github.com/StarRocks/starrocks/pull/17488)

## 2.5.1

发布日期：2023年2月5日

### 改进

- 基于外部目录创建的异步物化视图支持查询重写。[#11116](https://github.com/StarRocks/starrocks/issues/11116) [#15791](https://github.com/StarRocks/starrocks/issues/15791)
- 允许用户为自动CBO统计信息的收集指定收集周期，以防止因自动全量收集而引起的集群性能波动。[#14996](https://github.com/StarRocks/starrocks/pull/14996)
- 增加了Thrift服务器队列。在INSERT INTO SELECT期间无法立即处理的请求可以挂起在Thrift服务器队列中，防止请求被拒绝。[#14571](https://github.com/StarRocks/starrocks/pull/14571)
- 废弃了FE参数`default_storage_medium`。如果用户在创建表时未显式指定`storage_medium`，系统会根据BE磁盘类型自动推断表的存储介质。有关更多信息，请参阅[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_VIEW.md)中`storage_medium`的描述。[#14394](https://github.com/StarRocks/starrocks/pull/14394)

### Bug修复

已修复以下bug：

- 由于SET PASSWORD引起的空指针异常（NPE）。[#15247](https://github.com/StarRocks/starrocks/pull/15247)
- 无效键json数据无法解析。[#16852](https://github.com/StarRocks/starrocks/pull/16852)
- 成功将无效类型数据转换为ARRAY数据。[#16866](https://github.com/StarRocks/starrocks/pull/16866)
- 当异常发生时，无法中断嵌套循环连接。[#16875](https://github.com/StarRocks/starrocks/pull/16875)

### 行为改变

- 弃用FE参数`default_storage_medium`。表的存储介质会自动由系统推断。[#14394](https://github.com/StarRocks/starrocks/pull/14394)

## 2.5.0

发布日期：2023年1月22日

### 新特性

- 支持使用[Hudi目录](../data_source/catalog/hudi_catalog.md)和[Hudi外部表](../data_source/External_table.md#deprecated-hudi-external-table)查询Merge On Read表。[#6780](https://github.com/StarRocks/starrocks/pull/6780)
- 支持使用[Hive目录](../data_source/catalog/hive_catalog.md)、Hudi目录和[Iceberg目录](../data_source/catalog/iceberg_catalog.md)查询STRUCT和MAP数据。[#10677](https://github.com/StarRocks/starrocks/issues/10677)
- 提供[Data Cache](../data_source/data_cache.md)以提高存储在外部存储系统（如HDFS）中的热数据访问性能。[#11597](https://github.com/StarRocks/starrocks/pull/11579)
- 支持创建[Delta Lake目录](../data_source/catalog/deltalake_catalog.md)，允许直接查询Delta Lake中的数据。[#11972](https://github.com/StarRocks/starrocks/issues/11972)
- Hive、Hudi和Iceberg目录与AWS Glue兼容。[#12249](https://github.com/StarRocks/starrocks/issues/12249)
- 支持创建[file外部表](../data_source/file_external_table.md)，允许直接查询存储在HDFS和对象存储中的Parquet和ORC文件。[#13064](https://github.com/StarRocks/starrocks/pull/13064)
- 支持基于Hive、Hudi、Iceberg目录和物化视图创建物化视图。更多信息，请参阅[物化视图](../using_starrocks/Materialized_view.md)。[#11116](https://github.com/StarRocks/starrocks/issues/11116) [#11873](https://github.com/StarRocks/starrocks/pull/11873)
- 支持对使用主键表的表进行条件更新。更多信息，请参阅[通过加载更改数据](../loading/Load_to_Primary_Key_tables.md)。[#12159](https://github.com/StarRocks/starrocks/pull/12159)
- 支持[查询缓存](../using_starrocks/query_cache.md)，存储查询的中间计算结果，提高高并发、简单查询的QPS，并减少平均延迟。[#9194](https://github.com/StarRocks/starrocks/pull/9194)
- 支持指定Broker Load作业的优先级。更多信息，请参阅[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。[#11029](https://github.com/StarRocks/starrocks/pull/11029)
- 支持指定StarRocks本地表数据加载的副本数。更多信息，请参阅[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)。[#11253](https://github.com/StarRocks/starrocks/pull/11253)
- 支持[查询队列](../administration/query_queues.md)。[#12594](https://github.com/StarRocks/starrocks/pull/12594)
- 支持隔离数据加载占用的计算资源，限制数据加载任务的资源消耗。更多信息，请参阅[资源组](../administration/resource_group.md)。[#12606](https://github.com/StarRocks/starrocks/pull/12606)
- 支持为StarRocks本地表指定以下数据压缩算法：LZ4、Zstd、Snappy和Zlib。更多信息，请参阅[数据压缩](../table_design/data_compression.md)。[#10097](https://github.com/StarRocks/starrocks/pull/10097) [#12020](https://github.com/StarRocks/starrocks/pull/12020)
- 支持[用户定义变量](../reference/user_defined_variables.md)。[#10011](https://github.com/StarRocks/starrocks/pull/10011)
- 支持[lambda表达式](../sql-reference/sql-functions/Lambda_expression.md)和以下高阶函数：[array_map](../sql-reference/sql-functions/array-functions/array_map.md)、[array_sum](../sql-reference/sql-functions/array-functions/array_sum.md)和[array_sortby](../sql-reference/sql-functions/array-functions/array_sortby.md)。[#9461](https://github.com/StarRocks/starrocks/pull/9461) [#9806](https://github.com/StarRocks/starrocks/pull/9806) [#10323](https://github.com/StarRocks/starrocks/pull/10323) [#14034](https://github.com/StarRocks/starrocks/pull/14034)
- 提供QUALIFY子句，用于过滤[窗口函数](../sql-reference/sql-functions/Window_function.md)的结果。[#13239](https://github.com/StarRocks/starrocks/pull/13239)
- 支持使用uuid()和uuid_numeric()函数返回的结果作为创建表时列的默认值。更多信息，请参阅[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)。[#11155](https://github.com/StarRocks/starrocks/pull/11155)
- 支持以下函数：[map_size](../sql-reference/sql-functions/map-functions/map_size.md)、[map_keys](../sql-reference/sql-functions/map-functions/map_keys.md)、[map_values](../sql-reference/sql-functions/map-functions/map_values.md)、[max_by](../sql-reference/sql-functions/aggregate-functions/max_by.md)、[sub_bitmap](../sql-reference/sql-functions/bitmap-functions/sub_bitmap.md)、[bitmap_to_base64](../sql-reference/sql-functions/bitmap-functions/bitmap_to_base64.md)、[host_name](../sql-reference/sql-functions/utility-functions/host_name.md)和[date_slice](../sql-reference/sql-functions/date-time-functions/date_slice.md)。[#11299](https://github.com/StarRocks/starrocks/pull/11299) [#11323](https://github.com/StarRocks/starrocks/pull/11323) [#12243](https://github.com/StarRocks/starrocks/pull/12243) [#11776](https://github.com/StarRocks/starrocks/pull/11776) [#12634](https://github.com/StarRocks/starrocks/pull/12634) [#14225](https://github.com/StarRocks/starrocks/pull/14225)

### 改进

- 优化了通过[Hive目录](../data_source/catalog/hive_catalog.md)、[Hudi目录](../data_source/catalog/hudi_catalog.md)和[Iceberg目录](../data_source/catalog/iceberg_catalog.md)查询外部数据时的元数据访问性能。[#11349](https://github.com/StarRocks/starrocks/issues/11349)
- 支持使用[Elasticsearch外部表](../data_source/External_table.md#deprecated-elasticsearch-external-table)查询ARRAY数据。[#9693](https://github.com/StarRocks/starrocks/pull/9693)
- 优化了物化视图的以下方面：
  - 异步物化视图支持基于SPJG类型的自动和透明查询重写。更多信息，请参阅[物化视图](../using_starrocks/Materialized_view.md#rewrite-and-accelerate-queries-with-the-asynchronous-materialized-view)。[#13193](https://github.com/StarRocks/starrocks/issues/13193)
  - 异步物化视图支持多种异步刷新机制。更多信息，请参阅[物化视图](../using_starrocks/Materialized_view.md#manually-refresh-an-asynchronous-materialized-view)。[#12712](https://github.com/StarRocks/starrocks/pull/12712) [#13171](https://github.com/StarRocks/starrocks/pull/13171) [#13229](https://github.com/StarRocks/starrocks/pull/13229) [#12926](https://github.com/StarRocks/starrocks/pull/12926)
  - 优化了刷新物化视图的效率。[#13167](https://github.com/StarRocks/starrocks/issues/13167)
- 优化了数据加载的以下方面:
- 通过支持“单主复制”模式，优化了多副本场景下的加载性能。数据加载获得了一倍的性能提升。有关“单主复制”的更多信息，请参见 [CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) 中的 `replicated_storage`。[#10138](https://github.com/StarRocks/starrocks/pull/10138)
- 当只配置一个HDFS集群或一个Kerberos用户时，Broker Load和Spark Load不再需要依赖代理进行数据加载。但是，如果您有多个HDFS集群或多个Kerberos用户，则仍需部署代理。有关更多信息，请参见[从HDFS或云存储加载数据](../loading/BrokerLoad.md) 和 [使用Apache Spark™进行批量加载](../loading/SparkLoad.md)。[#9049](https://github.com/starrocks/starrocks/pull/9049) [#9228](https://github.com/StarRocks/starrocks/pull/9228)
- 当加载大量小的ORC文件时，优化了Broker Load的性能。[#11380](https://github.com/StarRocks/starrocks/pull/11380)
- 在将数据加载到主键表时，减少了内存使用。
- 优化了`information_schema`数据库以及其中的`tables`和`columns`表。增加了一个名为`table_config`的新表。有关更多信息，请参见 [Information Schema](../reference/overview-pages/information_schema.md)。[#10033](https://github.com/StarRocks/starrocks/pull/10033)
- 优化了数据备份和恢复：
  - 支持一次性备份和恢复数据库中多张表的数据。有关更多信息，请参见[备份和恢复数据](../administration/Backup_and_restore.md)。[#11619](https://github.com/StarRocks/starrocks/issues/11619)
  - 支持备份和恢复主键表的数据。有关更多信息，请参见备份和恢复。[#11885](https://github.com/StarRocks/starrocks/pull/11885)
- 优化了以下函数：
  - 为[time_slice](../sql-reference/sql-functions/date-time-functions/time_slice.md)函数增加了一个可选参数，用于确定返回时间间隔的开始或结束。[#11216](https://github.com/StarRocks/starrocks/pull/11216)
  - 为[window_funnel](../sql-reference/sql-functions/aggregate-functions/window_funnel.md)函数新增了一个名为 `INCREASE` 的模式，以避免计算重复的时间戳。[#10134](https://github.com/StarRocks/starrocks/pull/10134)
  - 支持在[unnest](../sql-reference/sql-functions/array-functions/unnest.md)函数中指定多个参数。[#12484](https://github.com/StarRocks/starrocks/pull/12484)
  - lead()和lag()函数支持查询HLL和BITMAP数据。有关更多信息，请参见[Window函数](../sql-reference/sql-functions/Window_function.md)。[#12108](https://github.com/StarRocks/starrocks/pull/12108)
  - 以下ARRAY函数支持查询JSON数据：[array_agg](../sql-reference/sql-functions/array-functions/array_agg.md)，[array_sort](../sql-reference/sql-functions/array-functions/array_sort.md)，[array_concat](../sql-reference/sql-functions/array-functions/array_concat.md)，[array_slice](../sql-reference/sql-functions/array-functions/array_slice.md)和[reverse](../sql-reference/sql-functions/array-functions/reverse.md)。[#13155](https://github.com/StarRocks/starrocks/pull/13155)
  - 优化了某些函数的使用。`current_date`、`current_timestamp`、`current_time`、`localtimestamp`和`localtime`函数可以在不使用`()`的情况下执行，例如，您可以直接运行`select current_date;`。[# 14319](https://github.com/StarRocks/starrocks/pull/14319)
- 从FE日志中删除了一些多余的信息。[# 15374](https://github.com/StarRocks/starrocks/pull/15374)

### Bug修复

已修复以下错误：

- 当第一个参数为空时，append_trailing_char_if_absent()函数可能返回不正确的结果。[#13762](https://github.com/StarRocks/starrocks/pull/13762)
- 使用RECOVER语句还原表后，该表不存在。[#13921](https://github.com/StarRocks/starrocks/pull/13921)
- 当创建物化视图时，SHOW CREATE MATERIALIZED VIEW语句返回的结果不包含查询语句中指定的数据库和目录。[#12833](https://github.com/StarRocks/starrocks/pull/12833)
- 处于`waiting_stable`状态的模式更改作业无法取消。[#12530](https://github.com/StarRocks/starrocks/pull/12530)
- 在Leader FE和非Leader FE上运行`SHOW PROC '/statistic';`命令返回的结果不同。[#12491](https://github.com/StarRocks/starrocks/issues/12491)
- SHOW CREATE TABLE返回的结果中ORDER BY子句的位置不正确。[# 13809](https://github.com/StarRocks/starrocks/pull/13809)
- 当用户使用Hive Catalog查询Hive数据时，如果FE生成的执行计划不包含分区ID，则BE无法查询Hive分区数据。[# 15486](https://github.com/StarRocks/starrocks/pull/15486)。

### 行为更改

- 将参数`AWS_EC2_METADATA_DISABLED`的默认值更改为`False`，这意味着获取Amazon EC2的元数据以访问AWS资源。
- 将会话变量`is_report_success`重命名为`enable_profile`，可以使用SHOW VARIABLES语句进行查询。
- 增加了四个保留关键字：`CURRENT_DATE`、`CURRENT_TIME`、`LOCALTIME`和`LOCALTIMESTAMP`。[# 14319](https://github.com/StarRocks/starrocks/pull/14319)
- 表名和数据库名的最大长度可以达到1023个字符。[# 14929](https://github.com/StarRocks/starrocks/pull/14929) [# 15020](https://github.com/StarRocks/starrocks/pull/15020)
- 将BE配置项`enable_event_based_compaction_framework`和 `enable_size_tiered_compaction_strategy`设置为默认值`true`，这在存在大量tablet或单个tablet具有大数据量时显著减少了压实开销。

### 升级说明

- 您可以将集群从2.0.x、2.1.x、2.2.x、2.3.x或2.4.x升级至2.5.0。但是，如果需要执行回滚操作，建议仅回滚到2.4.x版本。