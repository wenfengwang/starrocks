---
displayed_sidebar: English
---

# StarRocks版本2.2

发布日期：2023年4月6日

## 改进

- 优化了`bitmap_contains()`函数，减少其内存消耗，并在某些场景下提高了性能。[#20616](https://github.com/StarRocks/starrocks/issues/20616)
- 优化了Compaction框架，以减少其CPU资源消耗。[#11746](https://github.com/StarRocks/starrocks/issues/11746)

## Bug修复

修复了以下错误：

- 如果Stream Load作业中请求的URL不正确，负责的FE会挂起，无法处理HTTP请求。[#18468](https://github.com/StarRocks/starrocks/issues/18468)
- 负责的FE在收集统计信息时，可能会消耗异常大量的内存，导致OOM。[#16331](https://github.com/StarRocks/starrocks/issues/16331)
- 如果某些查询中未正确处理内存释放，BEs会崩溃。[#11395](https://github.com/StarRocks/starrocks/issues/11395)
- 执行`TRUNCATE TABLE`命令后，可能发生NullPointerException，导致负责的FE无法重启。[#16773](https://github.com/StarRocks/starrocks/issues/16773)

## 2.2.10

发布日期：2022年12月2日

### 改进

- 优化了Routine Load作业返回的错误消息。[#12203](https://github.com/StarRocks/starrocks/pull/12203)

- 支持逻辑运算符`&&`。[#11819](https://github.com/StarRocks/starrocks/issues/11819)

- 当BE崩溃时，查询会立即取消，防止因过期查询导致的系统卡顿问题。[#12954](https://github.com/StarRocks/starrocks/pull/12954)

- 优化了FE启动脚本，现在会在FE启动期间检查Java版本。[#14094](https://github.com/StarRocks/starrocks/pull/14094)

- 支持从主键表中删除大量数据。[#4772](https://github.com/StarRocks/starrocks/issues/4772)

### Bug修复

修复了以下错误：

- 当用户从多个表（UNION）创建视图时，如果UNION操作的最左侧子查询使用NULL常量，BEs会崩溃。([#13792](https://github.com/StarRocks/starrocks/pull/13792))

- 如果查询的Parquet文件与Hive表架构的列类型不一致，BEs会崩溃。[#8848](https://github.com/StarRocks/starrocks/issues/8848)

- 当查询包含大量OR运算符时，规划器需要执行过多的递归计算，导致查询超时。[#12788](https://github.com/StarRocks/starrocks/pull/12788)

- 当子查询包含LIMIT子句时，查询结果不正确。[#12466](https://github.com/StarRocks/starrocks/pull/12466)

- 当SELECT子句中的双引号与单引号混用时，CREATE VIEW语句失败。[#13102](https://github.com/StarRocks/starrocks/pull/13102)

## 2.2.9

发布日期：2022年11月15日

### 改进

- 新增会话变量`hive_partition_stats_sample_size`，用于控制收集统计信息的Hive分区数量。分区数量过多会导致获取Hive元数据时出错。[#12700](https://github.com/StarRocks/starrocks/pull/12700)

- Elasticsearch外部表支持自定义时区。[#12662](https://github.com/StarRocks/starrocks/pull/12662)

### Bug修复

修复了以下错误：

- 如果在外部表的元数据同步过程中发生错误，DECOMMISSION操作会卡住。[#12369](https://github.com/StarRocks/starrocks/pull/12368)

- 如果新添加的列被删除，Compaction会崩溃。[#12907](https://github.com/StarRocks/starrocks/pull/12907)

- SHOW CREATE VIEW不显示创建视图时添加的注释。[#4163](https://github.com/StarRocks/starrocks/issues/4163)

- Java UDF中的内存泄漏可能导致OOM。[#12418](https://github.com/StarRocks/starrocks/pull/12418)

- 在某些场景下，Follower FE存储的节点存活状态不准确，因为该状态依赖于`heartbeatRetryTimes`。为解决此问题，向`HeartbeatResponse`中添加了`aliveStatus`属性，以指示节点存活状态。[#12481](https://github.com/StarRocks/starrocks/pull/12481)

### 行为变更

将StarRocks可以查询的Hive STRING列的长度从64KB扩展到1MB。如果STRING列超过1MB，在查询期间将被处理为null列。[#12986](https://github.com/StarRocks/starrocks/pull/12986)

## 2.2.8

发布日期：2022年10月17日

### Bug修复

修复了以下错误：

- 如果表达式在初始化阶段遇到错误，BEs可能会崩溃。[#11395](https://github.com/StarRocks/starrocks/pull/11395)

- 如果加载的JSON数据无效，BEs可能会崩溃。[#10804](https://github.com/StarRocks/starrocks/issues/10804)

- 启用pipeline engine时，如果并行写入遇到错误，会出现问题。[#11451](https://github.com/StarRocks/starrocks/issues/11451)

- 使用ORDER BY NULL LIMIT子句时，BEs会崩溃。[#11648](https://github.com/StarRocks/starrocks/issues/11648)

- 如果要查询的Parquet文件的列类型与Hive表架构不一致，BEs会崩溃。[#11839](https://github.com/StarRocks/starrocks/issues/11839)

## 2.2.7

发布日期：2022年9月23日

### Bug修复

修复了以下错误：

- 当用户将JSON数据加载到StarRocks时，可能会丢失数据。[#11054](https://github.com/StarRocks/starrocks/issues/11054)

- SHOW FULL TABLES的输出不正确。[#11126](https://github.com/StarRocks/starrocks/issues/11126)

- 在以前的版本中，要访问视图中的数据，用户必须同时拥有基础表和视图的权限。在当前版本中，用户只需要对视图拥有权限。[#11290](https://github.com/StarRocks/starrocks/pull/11290)

- 嵌套了EXISTS或IN的复杂查询结果不正确。[#11415](https://github.com/StarRocks/starrocks/pull/11415)

- 如果相应的Hive表架构发生变化，REFRESH EXTERNAL TABLE操作会失败。[#11406](https://github.com/StarRocks/starrocks/pull/11406)

- 当非领导者FE重放位图索引创建操作时，可能会出现错误。[#11261](https://github.com/StarRocks/starrocks/issues/11261)

## 2.2.6

发布日期：2022年9月14日

### Bug修复

修复了以下错误：

- 当子查询包含LIMIT时，`order by... limit...offset`的结果不正确。[#9698](https://github.com/StarRocks/starrocks/issues/9698)

- 如果对数据量大的表执行部分更新，BE会崩溃。[#9809](https://github.com/StarRocks/starrocks/issues/9809)

- 如果要压缩的BITMAP数据大小超过2GB，Compaction会导致BE崩溃。[#11159](https://github.com/StarRocks/starrocks/pull/11159)

- 如果模式长度超过16KB，like()和regexp()函数不起作用。[#10364](https://github.com/StarRocks/starrocks/issues/10364)

### 行为变更

修改了在输出中表示数组中JSON值的格式。返回的JSON值中不再使用转义字符。[#10790](https://github.com/StarRocks/starrocks/issues/10790)

## 2.2.5

发布日期：2022年8月18日

### 改进

- 启用pipeline engine时，提升了系统性能。[#9580](https://github.com/StarRocks/starrocks/pull/9580)

- 提高了索引元数据内存统计的准确性。[#9837](https://github.com/StarRocks/starrocks/pull/9837)

### Bug修复

修复了以下错误：

- BEs在Routine Load期间查询Kafka分区偏移量（`get_partition_offset`）时可能会卡住。[#9937](https://github.com/StarRocks/starrocks/pull/9937)

- 当多个Broker Load线程尝试加载同一HDFS文件时，会发生错误。[#9507](https://github.com/StarRocks/starrocks/pull/9507)

## 2.2.4

发布日期：2022年8月3日

### 改进

- 支持将Hive表的schema更改同步到相应的外部表。[#9010](https://github.com/StarRocks/starrocks/pull/9010)

- 支持通过Broker Load加载Parquet文件中的ARRAY数据。[#9131](https://github.com/StarRocks/starrocks/pull/9131)

### Bug修复

修复了以下错误：

```
- Broker Load 无法处理具有多个 keytab 文件的 Kerberos 登录。[#8820](https://github.com/StarRocks/starrocks/pull/8820) [#8837](https://github.com/StarRocks/starrocks/pull/8837)

- 如果 **stop_be.sh** 执行后立即退出，Supervisor 可能无法重启服务。[#9175](https://github.com/StarRocks/starrocks/pull/9175)

- 不正确的 Join Reorder 优先级导致错误“无法解析列”。[#9063](https://github.com/StarRocks/starrocks/pull/9063) [#9487](https://github.com/StarRocks/starrocks/pull/9487)

## 2.2.3

发布日期：2022年7月24日

### Bug 修复

修复了以下错误：

- 用户删除资源组时出现错误。[#8036](https://github.com/StarRocks/starrocks/pull/8036)

- 当线程数不足时，Thrift 服务器退出。[#7974](https://github.com/StarRocks/starrocks/pull/7974)

- 在某些情况下，CBO 中的 join reorder 不返回任何结果。[#7099](https://github.com/StarRocks/starrocks/pull/7099) [#7831](https://github.com/StarRocks/starrocks/pull/7831) [#6866](https://github.com/StarRocks/starrocks/pull/6866)

## 2.2.2

发布日期：2022年6月29日

### 改进

- UDF 可跨数据库使用。[#6865](https://github.com/StarRocks/starrocks/pull/6865) [#7211](https://github.com/StarRocks/starrocks/pull/7211)

- 优化了内部处理如 schema change 的并发控制。这减轻了 FE 元数据管理的压力，并在需要高并发加载大量数据的场景中，减少了加载作业堆积或变慢的可能性。[#6838](https://github.com/StarRocks/starrocks/pull/6838)

### Bug 修复

修复了以下错误：

- 使用 CTAS 创建的副本数 (`replication_num`) 不正确。[#7036](https://github.com/StarRocks/starrocks/pull/7036)

- 执行 ALTER ROUTINE LOAD 后可能丢失元数据。[#7068](https://github.com/StarRocks/starrocks/pull/7068)

- Runtime filters 未能下推。[#7206](https://github.com/StarRocks/starrocks/pull/7206) [#7258](https://github.com/StarRocks/starrocks/pull/7258)

- Pipeline 问题可能导致内存泄漏。[#7295](https://github.com/StarRocks/starrocks/pull/7295)

- 当 Routine Load 作业被中止时，可能发生死锁。[#6849](https://github.com/StarRocks/starrocks/pull/6849)

- 一些 profile 统计信息不准确。[#7074](https://github.com/StarRocks/starrocks/pull/7074) [#6789](https://github.com/StarRocks/starrocks/pull/6789)

- `get_json_string` 函数处理 JSON 数组不正确。[#7671](https://github.com/StarRocks/starrocks/pull/7671)

## 2.2.1

发布日期：2022年6月2日

### 改进

- 通过重构热点代码和降低锁粒度，优化了数据加载性能并减少了长尾延迟。[#6641](https://github.com/StarRocks/starrocks/pull/6641)

- 在 FE 审计日志中添加了每次查询时部署 BE 的机器的 CPU 和内存使用信息。[#6208](https://github.com/StarRocks/starrocks/pull/6208) [#6209](https://github.com/StarRocks/starrocks/pull/6209)

- 支持在主键表和唯一键表中使用 JSON 数据类型。[#6544](https://github.com/StarRocks/starrocks/pull/6544)

- 通过降低锁粒度和去重 BE 报告请求，减轻了 FE 负载。优化了大量 BE 部署时的报告性能，并解决了大型集群中 Routine Load 任务卡住的问题。[#6293](https://github.com/StarRocks/starrocks/pull/6293)

### Bug 修复

修复了以下错误：

- 当 StarRocks 解析 `SHOW FULL TABLES FROM DatabaseName` 语句中指定的转义字符时出现错误。[#6559](https://github.com/StarRocks/starrocks/issues/6559)

- FE 磁盘空间使用率急剧上升（通过回滚 BDBJE 版本修复此错误）。[#6708](https://github.com/StarRocks/starrocks/pull/6708)

- 启用列扫描 (`enable_docvalue_scan=true`) 后，因无法在返回的数据中找到相关字段，导致 BE 故障。[#6600](https://github.com/StarRocks/starrocks/pull/6600)

## 2.2.0

发布日期：2022年5月22日

### 新特性

- [预览] 发布资源组管理功能。此功能允许 StarRocks 在同一集群中处理不同租户的复杂查询和简单查询时，隔离并高效使用 CPU 和内存资源。

- [预览] 实现了基于 Java 的用户定义函数（UDF）框架。该框架支持符合 Java 语法编译的 UDF，以扩展 StarRocks 的功能。

- [预览] 主键表在订单更新、多流 Join 等实时数据更新场景下，支持仅对特定列进行数据更新。

- [预览] 支持 JSON 数据类型和 JSON 函数。

- 外部表可以用于查询 Apache Hudi 的数据。这进一步改善了用户使用 StarRocks 进行数据湖分析的体验。更多信息，请参见 [外部表](../data_source/External_table.md)。

- 新增以下函数：
  - ARRAY 函数：[array_agg](../sql-reference/sql-functions/array-functions/array_agg.md)、array_sort、array_distinct、array_join、reverse、array_slice、array_concat、array_difference、arrays_overlap 和 array_intersect
  - BITMAP 函数：bitmap_max 和 bitmap_min
  - 其他函数：retention 和 square

### 改进

- 重构了基于成本的优化器（CBO）的解析器和分析器，优化了代码结构，并支持了 INSERT 语句中的公共表表达式（CTE）等语法。这些改进旨在提高复杂查询的性能，例如涉及 CTE 重用的查询。

- 优化了存储在 AWS S3、阿里云 OSS 和腾讯云 COS 等云对象存储服务中的 Apache Hive™ 外部表的查询性能。优化后，基于对象存储的查询性能与基于 HDFS 的查询性能相当。此外，支持 ORC 文件的后期物化，并加速了小文件的查询。更多信息，请参见 [Apache Hive™ 外部表](../data_source/External_table.md)。

- 当使用外部表运行 Apache Hive™ 的查询时，StarRocks 通过消费 Hive 元存储的事件（如数据变更和分区变更）自动进行缓存元数据的增量更新。StarRocks 还支持对 Apache Hive™ 中 DECIMAL 和 ARRAY 类型数据的查询。更多信息，请参见 [Apache Hive™ 外部表](../data_source/External_table.md)。

- 对 UNION ALL 运算符进行了优化，运行速度比以前快 2 到 25 倍。

- 发布了支持自适应并行性并提供优化的 profile 的管道引擎，提高了高并发场景下简单查询的性能。

- 可以将多个字符组合使用作为 CSV 文件导入时的单行分隔符。

### Bug 修复

- 如果将数据加载或更改提交到基于主键表的表中，可能会发生死锁。[#4998](https://github.com/StarRocks/starrocks/pull/4998)

- 前端（FE），包括运行 Oracle Berkeley DB Java Edition（BDB JE）的 FE，不稳定。[#4428](https://github.com/StarRocks/starrocks/pull/4428)，[#4666](https://github.com/StarRocks/starrocks/pull/4666)，[#2](https://github.com/StarRocks/bdb-je/pull/2)

- 如果对大量数据调用 SUM 函数，则该函数返回的结果可能会发生算术溢出。[#3944](https://github.com/StarRocks/starrocks/pull/3944)

- ROUND 和 TRUNCATE 函数返回的结果精度不满意。[#4256](https://github.com/StarRocks/starrocks/pull/4256)

- SQLancer 检测到一些错误。更多信息，请参见 [SQLancer 相关问题](https://github.com/StarRocks/starrocks/issues?q=is:issue++label:sqlancer++milestone:2.2)。

### 其他

Flink-connector-starrocks 支持 Apache Flink® v1.14。

### 升级注意事项
- 如果您使用的是 StarRocks 2.0.4 之后的版本或者是 2.1.x 系列中 2.1.6 之后的版本，在升级之前您可以禁用平板克隆功能（`ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "0");` 和 `ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "0");`）。升级之后，您可以重新启用此功能（`ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "2000");` 和 `ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "100");`）。
