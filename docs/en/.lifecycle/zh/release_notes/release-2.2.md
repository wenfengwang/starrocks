---
displayed_sidebar: English
---

# StarRocks 2.2 版本

发布日期：2023年4月6日

## 改进

- 优化了 bitmap_contains() 函数，减少了内存消耗，并提升了部分场景下的性能。[问题编号#20616](https://github.com/StarRocks/starrocks/issues/20616)
- 优化了 Compaction 框架，降低了 CPU 资源消耗。[问题编号#11746](https://github.com/StarRocks/starrocks/issues/11746)

## Bug 修复

已修复以下问题：

- 如果 Stream Load 作业中请求的 URL 不正确，则负责的 FE 会挂起，无法处理 HTTP 请求。[问题编号#18468](https://github.com/StarRocks/starrocks/issues/18468)
- 负责的 FE 在收集统计信息时，可能会消耗异常大量的内存，导致 OOM。[问题编号#16331](https://github.com/StarRocks/starrocks/issues/16331)
- 如果在某些查询中未正确处理内存释放，则 BE 会崩溃。[问题编号#11395](https://github.com/StarRocks/starrocks/issues/11395)
- 执行 TRUNCATE TABLE 命令后，可能会出现 NullPointerException，导致负责的 FE 无法重新启动。[问题编号#16773](https://github.com/StarRocks/starrocks/issues/16773)

## 2.2.10

发布日期：2022年12月2日

### 改进

- 优化了 Routine Load 作业返回的错误信息。[问题编号#12203](https://github.com/StarRocks/starrocks/pull/12203)
- 支持逻辑运算符 `&&`。[问题编号#11819](https://github.com/StarRocks/starrocks/issues/11819)
- 当 BE 崩溃时，查询会立即取消，防止过期查询导致系统卡顿的问题。[问题编号#12954](https://github.com/StarRocks/starrocks/pull/12954)
- 优化了 FE 启动脚本，现在在 FE 启动期间检查 Java 版本。[问题编号#14094](https://github.com/StarRocks/starrocks/pull/14094)
- 支持从主键表中删除大量数据。[问题编号#4772](https://github.com/StarRocks/starrocks/issues/4772)

### Bug 修复

已修复以下问题：

- 当用户从多个表（UNION）创建视图时，如果 UNION 操作的最左侧子项使用 NULL 常量，则 BE 会崩溃。([问题编号#13792](https://github.com/StarRocks/starrocks/pull/13792))
- 如果要查询的 Parquet 文件的列类型与 Hive 表架构不一致，则 BE 会崩溃。[问题编号#8848](https://github.com/StarRocks/starrocks/issues/8848)
- 当查询包含大量 OR 运算符时，规划器需要执行过多的递归计算，导致查询超时。[问题编号#12788](https://github.com/StarRocks/starrocks/pull/12788)
- 当子查询包含 LIMIT 子句时，查询结果不正确。[问题编号#12466](https://github.com/StarRocks/starrocks/pull/12466)
- 当 SELECT 子句中的双引号与单引号混合时，CREATE VIEW 语句将失败。[问题编号#13102](https://github.com/StarRocks/starrocks/pull/13102)

## 2.2.9

发布日期：2022年11月15日

### 改进

- 添加了 session 变量 `hive_partition_stats_sample_size` 以控制要从中收集统计信息的 Hive 分区数量。分区过多会导致 Hive 元数据获取错误。[问题编号#12700](https://github.com/StarRocks/starrocks/pull/12700)
- Elasticsearch 外部表支持自定义时区。[问题编号#12662](https://github.com/StarRocks/starrocks/pull/12662)

### Bug 修复

已修复以下问题：

- 如果在外部表的元数据同步过程中发生错误，则 DECOMMISSION 操作会卡住。[问题编号#12369](https://github.com/StarRocks/starrocks/pull/12368)
- 如果删除了新添加的列，则 Compaction 会崩溃。[问题编号#12907](https://github.com/StarRocks/starrocks/pull/12907)
- SHOW CREATE VIEW 不显示创建视图时添加的注释。[问题编号#4163](https://github.com/StarRocks/starrocks/issues/4163)
- Java UDF 中的内存泄漏可能会导致 OOM。[问题编号#12418](https://github.com/StarRocks/starrocks/pull/12418)
- 在某些情况下，存储在 Follower FE 中的节点活动状态并不准确，因为状态取决于 `heartbeatRetryTimes`。为解决此问题，添加了一个属性 `aliveStatus` 到 `HeartbeatResponse` 来指示节点的活动状态。[问题编号#12481](https://github.com/StarRocks/starrocks/pull/12481)

### 行为变更

将 StarRocks 可查询的 Hive STRING 列长度从 64 KB 扩展至 1 MB。如果 STRING 列超过 1 MB，则在查询过程中将作为 null 列处理。[问题编号#12986](https://github.com/StarRocks/starrocks/pull/12986)

## 2.2.8

发布日期：2022年10月17日

### Bug 修复

已修复以下问题：

- 如果表达式在初始化阶段遇到错误，则 BE 可能会崩溃。[问题编号#11395](https://github.com/StarRocks/starrocks/pull/11395)
- 如果加载了无效的 JSON 数据，则 BE 可能会崩溃。[问题编号#10804](https://github.com/StarRocks/starrocks/issues/10804)
- 并行写入在启用管道引擎时遇到错误。[问题编号#11451](https://github.com/StarRocks/starrocks/issues/11451)
- 使用 ORDER BY NULL LIMIT 子句时，BE 崩溃。[问题编号#11648](https://github.com/StarRocks/starrocks/issues/11648)
- 如果要查询的 Parquet 文件的列类型与 Hive 表架构不一致，则 BE 会崩溃。[问题编号#11839](https://github.com/StarRocks/starrocks/issues/11839)

## 2.2.7

发布日期：2022年9月23日

### Bug 修复

已修复以下问题：

- 当用户将 JSON 数据加载到 StarRocks 中时，数据可能会丢失。[问题编号#11054](https://github.com/StarRocks/starrocks/issues/11054)
- SHOW FULL TABLES 的输出不正确。[问题编号#11126](https://github.com/StarRocks/starrocks/issues/11126)
- 在以前的版本中，要访问视图中的数据，用户必须同时具有基表和视图的权限。在当前版本中，用户只需要对视图具有权限。[问题编号#11290](https://github.com/StarRocks/starrocks/pull/11290)
- 嵌套有 EXISTS 或 IN 的复杂查询的结果不正确。[问题编号#11415](https://github.com/StarRocks/starrocks/pull/11415)
- 如果更改了相应 Hive 表的架构，则 REFRESH EXTERNAL TABLE 将失败。[问题编号#11406](https://github.com/StarRocks/starrocks/pull/11406)
- 当非 leader FE 重放位图索引创建操作时，可能会发生错误。[问题编号#11261]

## 2.2.6

发布日期：2022年9月14日

### Bug 修复

已修复以下问题：

- 当子查询包含 LIMIT 时，`order by... limit...offset` 的结果不正确。[问题编号#9698](https://github.com/StarRocks/starrocks/issues/9698)
- 如果对数据量较大的表执行部分更新，则 BE 会崩溃。[问题编号#9809](https://github.com/StarRocks/starrocks/issues/9809)
- 如果要压缩的 BITMAP 数据大小超过 2 GB，则压缩会导致 BE 崩溃。[问题编号#11159](https://github.com/StarRocks/starrocks/pull/11159)
- 如果模式长度超过 16 KB，则 like() 和 regexp() 函数不起作用。[问题编号#10364](https://github.com/StarRocks/starrocks/issues/10364)

### 行为变更

修改了用于在输出中的数组中表示 JSON 值的格式。返回的 JSON 值中不再使用转义字符。[问题编号#10790](https://github.com/StarRocks/starrocks/issues/10790)

## 2.2.5

发布日期：2022年8月18日

### 改进

- 改进了启用流水线引擎时的系统性能。[问题编号#9580](https://github.com/StarRocks/starrocks/pull/9580)
- 提高了索引元数据的内存统计信息的准确性。[问题编号#9837](https://github.com/StarRocks/starrocks/pull/9837)

### Bug 修复

已修复以下问题：

- 在例程加载期间，`get_partition_offset`BE 可能会卡在查询 Kafka 分区偏移量时。[问题编号#9937](https://github.com/StarRocks/starrocks/pull/9937)
- 当多个 Broker Load 线程尝试装入同一个 HDFS 文件时，会发生错误。[问题编号#9507](https://github.com/StarRocks/starrocks/pull/9507)

## 2.2.4

发布日期：2022年8月3日

### 改进

- 支持将 Hive 表上的 schema 更改同步到对应的外部表。[问题编号#9010](https://github.com/StarRocks/starrocks/pull/9010)
- 支持通过 Broker Load 将 ARRAY 数据加载到 Parquet 文件中。[问题编号#9131](https://github.com/StarRocks/starrocks/pull/9131)

### Bug 修复

已修复以下问题：


- Broker Load 无法处理具有多个 keytab 文件的 Kerberos 登录。 [#8820](https://github.com/StarRocks/starrocks/pull/8820) [#8837](https://github.com/StarRocks/starrocks/pull/8837)

- 如果 **stop_be.sh** 在执行后立即退出，Supervisor 可能无法重新启动服务。 [#9175](https://github.com/StarRocks/starrocks/pull/9175)

- 不正确的联接重新排序优先级会导致错误 "无法解析列"。 [#9063](https://github.com/StarRocks/starrocks/pull/9063) [#9487](https://github.com/StarRocks/starrocks/pull/9487)

## 2.2.3

发布日期：2022年7月24日

### Bug 修复

已修复以下问题：

- 用户删除资源组时出现错误。 [#8036](https://github.com/StarRocks/starrocks/pull/8036)

- 当线程数不足时，Thrift 服务器会退出。 [#7974](https://github.com/StarRocks/starrocks/pull/7974)

- 在某些情况下，CBO 中的联接重新排序不会返回任何结果。 [#7099](https://github.com/StarRocks/starrocks/pull/7099) [#7831](https://github.com/StarRocks/starrocks/pull/7831) [#6866](https://github.com/StarRocks/starrocks/pull/6866)

## 2.2.2

发布日期：2022年6月29日

### 改进

- UDF 可以跨数据库使用。 [#6865](https://github.com/StarRocks/starrocks/pull/6865) [#7211](https://github.com/StarRocks/starrocks/pull/7211)

- 优化了内部处理，如架构更改的并发控制。这减轻了 FE 元数据管理的压力。此外，在需要高并发加载大量数据的情况下，负载作业堆积或变慢的可能性也会降低。 [#6838](https://github.com/StarRocks/starrocks/pull/6838)

### Bug 修复

已修复以下问题：

- 使用 CTAS 创建的 `replication_num` 副本数不正确。 [#7036](https://github.com/StarRocks/starrocks/pull/7036)

- 执行 ALTER ROUTINE LOAD 后，元数据可能会丢失。 [#7068](https://github.com/StarRocks/starrocks/pull/7068)

- 运行时筛选器无法下推。 [#7206](https://github.com/StarRocks/starrocks/pull/7206) [#7258](https://github.com/StarRocks/starrocks/pull/7258)

- 可能导致内存泄漏的管道问题。 [#7295](https://github.com/StarRocks/starrocks/pull/7295)

- 当例行加载作业中止时，可能会发生死锁。 [#6849](https://github.com/StarRocks/starrocks/pull/6849)

- 某些配置文件统计信息不准确。 [#7074](https://github.com/StarRocks/starrocks/pull/7074) [#6789](https://github.com/StarRocks/starrocks/pull/6789)

- get_json_string 函数不正确地处理 JSON 数组。 [#7671](https://github.com/StarRocks/starrocks/pull/7671)

## 2.2.1

发布日期：2022年6月2日

### 改进

- 通过重构部分热点代码和降低锁粒度，优化了数据加载性能，降低长尾延迟。 [#6641](https://github.com/StarRocks/starrocks/pull/6641)

- 在 FE 审计日志中新增了每次查询都部署了 BE 的机器的 CPU 和内存使用信息。 [#6208](https://github.com/StarRocks/starrocks/pull/6208) [#6209](https://github.com/StarRocks/starrocks/pull/6209)

- 主键表和唯一键表中支持的 JSON 数据类型。 [#6544](https://github.com/StarRocks/starrocks/pull/6544)

- 通过降低锁定粒度和删除 BE 报告请求的重复数据，减少了 FE 负载。优化部署大量 BE 时的报表性能，解决 Routine Load 任务卡在大型集群中的问题。 [#6293](https://github.com/StarRocks/starrocks/pull/6293)

### Bug 修复

已修复以下问题：

- StarRocks 解析语句中指定的转义字符时出现错误 `SHOW FULL TABLES FROM DatabaseName`。 [#6559](https://github.com/StarRocks/starrocks/issues/6559)

- FE 磁盘空间使用率急剧上升（通过回滚 BDBJE 版本修复此问题）。 [#6708](https://github.com/StarRocks/starrocks/pull/6708)

- BE 会出错，因为在启用列式扫描后返回的数据中找不到相关字段（`enable_docvalue_scan=true`）。 [#6600](https://github.com/StarRocks/starrocks/pull/6600)

## 2.2.0

发布日期：2022年5月22日

### 新功能

- [预览] 发布了资源组管理功能。该功能允许 StarRocks 在处理来自同一集群中不同租户的复杂查询和简单查询时，隔离并高效利用 CPU 和内存资源。

- [预览] 实现了基于 Java 的用户定义函数（UDF）框架。该框架支持按照 Java 语法编译的 UDF，以扩展 StarRocks 的能力。

- [预览] 在实时数据更新场景（如订单更新、多流联接等）中，当数据加载到主键表时，主键表仅支持对特定列的更新。

- [预览] 支持 JSON 数据类型和 JSON 函数。

- 外部表可用于从 Apache Hudi 查询数据。这进一步提升了用户对 StarRocks 的数据湖分析体验。有关详细信息，请参阅 [外部表](../data_source/External_table.md)。

- 新增以下功能：
  - ARRAY 功能：[array_agg](../sql-reference/sql-functions/array-functions/array_agg.md)、array_sort、array_distinct、array_join、reverse、array_slice、array_concat、array_difference、arrays_overlap、array_intersect
  - BITMAP 函数：bitmap_max 和 bitmap_min
  - 其他功能：retention 和 square

### 改进

- 重构了基于成本的优化器（CBO）的解析器和分析器，优化了代码结构，支持 INSERT 与通用表表达式（CTE）等语法。进行这些改进是为了提高复杂查询的性能，例如涉及重用 CTE 的查询。

- 优化了 AWS Simple Storage Service（S3）、阿里云 OSS、腾讯云对象存储（COS）等云对象存储服务中 Apache Hive™ 外部表的查询性能。优化后，基于对象存储的查询性能与基于 HDFS 的查询相当。此外，还支持 ORC 文件的后期物化，并加速对小文件的查询。有关详细信息，请参阅 [Apache Hive™ 外部表](../data_source/External_table.md)。

- 当 Apache Hive 使用外部表进行查询时，StarRocks 会通过消耗 Hive™ 元存储事件（如数据更改、分区更改等）自动对缓存的元数据进行增量更新。StarRocks 还支持对 Apache Hive™ 中 DECIMAL 和 ARRAY 类型的数据进行查询。有关详细信息，请参阅 [Apache Hive™ 外部表](../data_source/External_table.md)。

- UNION ALL 运算符经过优化，运行速度比以前快 2 到 25 倍。

- 发布支持自适应并行并提供优化配置文件的流水线引擎，提升高并发场景下简单查询的性能。

- 可以将多个字符组合在一起，并用作要导入的 CSV 文件的单行分隔符。

### Bug 修复

已修复以下问题：

- 如果加载数据或将更改提交到基于主键表的表中，则会发生死锁。 [#4998](https://github.com/StarRocks/starrocks/pull/4998)

- 前端（FE），包括运行 Oracle Berkeley DB Java 版（BDB JE）的 FE 不稳定。 [#4428](https://github.com/StarRocks/starrocks/pull/4428)、[#4666](https://github.com/StarRocks/starrocks/pull/4666)、[#2](https://github.com/StarRocks/bdb-je/pull/2)

- 如果对大量数据调用 SUM 函数返回的结果，则会遇到算术溢出。 [#3944](https://github.com/StarRocks/starrocks/pull/3944)

- ROUND 和 TRUNCATE 函数返回的结果的精度不令人满意。 [#4256](https://github.com/StarRocks/starrocks/pull/4256)

- Synthesized Query Lancer（SQLancer）检测到一些错误。有关详细信息，请参阅 [与 SQLancer 相关的问题](https://github.com/StarRocks/starrocks/issues?q=is:issue++label:sqlancer++milestone:2.2)。

### 其他

Flink-connector-starrocks 支持 Apache Flink® v1.14。

### 升级说明
- 如果您使用的是 StarRocks 2.0.4 以上版本或 StarRocks 2.1.x 2.1.6 以上版本，您可以在升级前禁用 tablet clone 功能（`ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "0");` 和 `ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "0");`）。升级后，您可以启用此功能（`ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "2000");` 和 `ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "100");`）。

- 要回滚到升级前使用的上一个版本，请在每个 FE 的 **fe.conf** 文件中添加 `ignore_unknown_log_id` 参数，并将该参数设置为 `true`。由于 StarRocks v2.2.0 中新增了新类型的日志，因此需要此参数。如果不添加该参数，将无法回滚到先前的版本。我们建议您在创建检查点后，将每个 FE 的 **fe.conf** 文件中的 `ignore_unknown_log_id` 参数设置为 `false`。然后，重新启动 FE 以恢复先前的配置。