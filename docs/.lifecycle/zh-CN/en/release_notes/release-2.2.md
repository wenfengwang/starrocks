---
displayed_sidebar: "中文"
---

# StarRocks版本 2.2

发布日期：2023年4月6日

## 改进

- 优化了bitmap_contains()函数，减少其内存消耗，在某些场景下提高了性能。[#20616](https://github.com/StarRocks/starrocks/issues/20616)
- 优化了压缩框架，减少其CPU资源消耗。[#11746](https://github.com/StarRocks/starrocks/issues/11746)

## Bug修复

已修复以下bug：

- 当流式加载作业中请求的URL不正确时，负责的前端挂起，无法处理HTTP请求。 [#18468](https://github.com/StarRocks/starrocks/issues/18468)
- 当负责的前端收集统计信息时，可能会消耗异常大量的内存，导致OOM。 [#16331](https://github.com/StarRocks/starrocks/issues/16331)
- 如果某些查询中未正确处理内存释放，则后端会崩溃。[#11395](https://github.com/StarRocks/starrocks/issues/11395)
- 执行TRUNCATE TABLE命令后，可能会发生空指针异常，负责的前端无法重新启动。[#16773](https://github.com/StarRocks/starrocks/issues/16773)

## 2.2.10

发布日期：2022年12月2日

### 改进

- 优化了常规加载作业返回的错误消息。[#12203](https://github.com/StarRocks/starrocks/pull/12203)

- 支持逻辑运算符`&&`。[#11819](https://github.com/StarRocks/starrocks/issues/11819)

- 如果后端崩溃，立即取消查询，防止由过期查询导致的系统卡死问题。[#12954](https://github.com/StarRocks/starrocks/pull/12954)

- 优化了前端启动脚本。现在在前端启动时检查Java版本。[#14094](https://github.com/StarRocks/starrocks/pull/14094)

- 支持从主键表中删除大量数据。[#4772](https://github.com/StarRocks/starrocks/issues/4772)

### Bug修复

已修复以下bug：

- 当用户从多个表（联合）创建视图时，如果UNION操作的最左侧子句使用NULL常量，后端会崩溃。([#13792](https://github.com/StarRocks/starrocks/pull/13792))

- 如果要查询的Parquet文件与Hive表模式的列类型不一致，则后端会崩溃。[#8848](https://github.com/StarRocks/starrocks/issues/8848)

- 当查询包含大量OR运算符时，规划器需要执行过多的递归计算，导致查询超时。[#12788](https://github.com/StarRocks/starrocks/pull/12788)

- 当子查询包含LIMIT子句时，查询结果不正确。[#12466](https://github.com/StarRocks/starrocks/pull/12466)

- 如果SELECT子句中混合了双引号和单引号，创建VIEW语句将失败。[#13102](https://github.com/StarRocks/starrocks/pull/13102)

## 2.2.9

发布日期：2022年11月15日

### 改进

- 添加了会话变量`hive_partition_stats_sample_size`以控制收集统计信息时要从Hive分区中获取的数量。过多的分区将导致获取Hive元数据时出错。[#12700](https://github.com/StarRocks/starrocks/pull/12700)

- Elasticsearch外部表支持自定义时区。[#12662](https://github.com/StarRocks/starrocks/pull/12662)

### Bug修复

已修复以下bug：

- 如果在外部表的元数据同步期间发生错误，DECOMMISSION操作将会卡住。[#12369](https://github.com/StarRocks/starrocks/pull/12368)

- 如果新添加的列被删除，则压缩会导致后端崩溃。[#12907](https://github.com/StarRocks/starrocks/pull/12907)

- SHOW CREATE VIEW不会显示在创建视图时添加的注释。[#4163](https://github.com/StarRocks/starrocks/issues/4163)

- Java UDF中的内存泄漏可能会导致OOM。[#12418](https://github.com/StarRocks/starrocks/pull/12418)

- Follower FEs中存储的节点存活状态在某些场景下不准确，因为状态依赖于`heartbeatRetryTimes`。为了解决此问题，向`HeartbeatResponse`添加了`aliveStatus`属性以指示节点的存活状态。[#12481](https://github.com/StarRocks/starrocks/pull/12481)

### 行为变更

将StarRocks对Hive STRING列的查询长度从64KB扩展到了1MB。如果STRING列超过1MB，将在查询期间处理为null列。[#12986](https://github.com/StarRocks/starrocks/pull/12986)

## 2.2.8

发布日期：2022年10月17日

### Bug修复

已修复以下bug：

- 如果表达式在初始化阶段遇到错误，后端可能会崩溃。[#11395](https://github.com/StarRocks/starrocks/pull/11395)

- 如果加载无效的JSON数据，后端可能会崩溃。[#10804](https://github.com/StarRocks/starrocks/issues/10804)

- 当启用管道引擎时，并行写入遇到错误。[#11451](https://github.com/StarRocks/starrocks/issues/11451)

- 使用ORDER BY NULL LIMIT子句时，后端会崩溃。[#11648](https://github.com/StarRocks/starrocks/issues/11648)

- 如果要查询的Parquet文件与Hive表模式的列类型不一致，则后端会崩溃。[#11839](https://github.com/StarRocks/starrocks/issues/11839)

## 2.2.7

发布日期：2022年9月23日

### Bug修复

已修复以下bug：

- 用户将JSON数据加载到StarRocks中时可能会丢失数据。[#11054](https://github.com/StarRocks/starrocks/issues/11054)

- SHOW FULL TABLES的输出不正确。[#11126](https://github.com/StarRocks/starrocks/issues/11126)

- 在之前的版本中，要访问视图中的数据，用户必须在基表和视图上都拥有权限。在当前版本中，只需要在视图上具有权限。[#11290](https://github.com/StarRocks/starrocks/pull/11290)

- 嵌套有EXISTS或IN的复杂查询的结果不正确。[#11415](https://github.com/StarRocks/starrocks/pull/11415)

- 如果对应Hive表的模式发生更改，刷新外部表将失败。[#11406](https://github.com/StarRocks/starrocks/pull/11406)

- 如果非主导FE重放位图索引创建操作，则可能会出现错误。[#11261](https://github.com/StarRocks/starrocks/pull/11261)

## 2.2.6

发布日期：2022年9月14日

### Bug修复

已修复以下bug：

- 当子查询中包含LIMIT时，`order by... limit...offset`的结果不正确。[#9698](https://github.com/StarRocks/starrocks/issues/9698)

- 如果在数据量很大的表上执行部分更新，后端会崩溃。[#9809](https://github.com/StarRocks/starrocks/issues/9809)

- 如果要压缩的BITMAP数据大小超过2GB，则压缩会导致后端崩溃。[#11159](https://github.com/StarRocks/starrocks/pull/11159)

- 如果pattern长度超过16KB，则like()和regexp()函数无法工作。[#10364](https://github.com/StarRocks/starrocks/issues/10364)

### 行为变更

在输出中修改了表示数组中JSON值格式的方式。返回的JSON值不再使用转义字符。[#10790](https://github.com/StarRocks/starrocks/issues/10790)

## 2.2.5

发布日期：2022年8月18日

### 改进

- 当启用管道引擎时，改进了系统性能。[#9580](https://github.com/StarRocks/starrocks/pull/9580)

- 改进了索引元数据的内存统计准确性。[#9837](https://github.com/StarRocks/starrocks/pull/9837)

### Bug修复

已修复以下bug：

- 在常规加载期间，后端可能会在查询Kafka分区偏移量(`get_partition_offset`)时卡住。[#9937](https://github.com/StarRocks/starrocks/pull/9937)

- 当多个 Broker Load 线程尝试加载同一个 HDFS 文件时会发生错误。[#9507](https://github.com/StarRocks/starrocks/pull/9507)

## 2.2.4

发布日期：2022年8月3日

### 改进

- 支持将 Hive 表上的模式更改同步到相应的外部表。[#9010](https://github.com/StarRocks/starrocks/pull/9010)

- 支持通过 Broker Load 加载 Parquet 文件中的 ARRAY 数据。[#9131](https://github.com/StarRocks/starrocks/pull/9131)

### Bug 修复

修复以下 Bug：

- Broker Load 无法处理带有多个 keytab 文件的 Kerberos 登录。[#8820](https://github.com/StarRocks/starrocks/pull/8820) [#8837](https://github.com/StarRocks/starrocks/pull/8837)

- 如果 **stop_be.sh** 在执行后立即退出，监督程序可能无法重启服务。[#9175](https://github.com/StarRocks/starrocks/pull/9175)

- 不正确的 Join Reorder 优先级导致错误“无法解析列”。[#9063](https://github.com/StarRocks/starrocks/pull/9063) [#9487](https://github.com/StarRocks/starrocks/pull/9487)

## 2.2.3

发布日期：2022年7月24日

### Bug 修复

修复以下 Bug：

- 用户删除资源组时发生错误。[#8036](https://github.com/StarRocks/starrocks/pull/8036)

- 当线程数量不足时，Thrift 服务器可能会退出。[#7974](https://github.com/StarRocks/starrocks/pull/7974)

- 在某些场景中，CBO 中的 join reorder 不返回结果。[#7099](https://github.com/StarRocks/starrocks/pull/7099) [#7831](https://github.com/StarRocks/starrocks/pull/7831) [#6866](https://github.com/StarRocks/starrocks/pull/6866)

## 2.2.2

发布日期：2022年6月29日

### 改进

- UDF 可以跨数据库使用。[#6865](https://github.com/StarRocks/starrocks/pull/6865) [#7211](https://github.com/StarRocks/starrocks/pull/7211)

- 优化了内部处理（例如模式更改）的并发控制。这减少了对 FE 元数据管理的压力。此外，在需要高并发加载大量数据的场景中，减少了加载作业可能堆积或减慢的可能性。[#6838](https://github.com/StarRocks/starrocks/pull/6838)

### Bug 修复

修复以下 Bug：

- 使用 CTAS 创建的副本数（`replication_num`）不正确。[#7036](https://github.com/StarRocks/starrocks/pull/7036)

- 在执行 ALTER ROUTINE LOAD 后可能会丢失元数据。[#7068](https://github.com/StarRocks/starrocks/pull/7068)

- 运行时过滤器无法下推。[#7206](https://github.com/StarRocks/starrocks/pull/7206) [#7258](https://github.com/StarRocks/starrocks/pull/7258)

- 可能导致内存泄漏的 Pipeline 问题。[#7295](https://github.com/StarRocks/starrocks/pull/7295)

- Aborting Routine Load 作业时可能发生死锁。[#6849](https://github.com/StarRocks/starrocks/pull/6849)

- 一些 profile 统计信息不准确。[#7074](https://github.com/StarRocks/starrocks/pull/7074) [#6789](https://github.com/StarRocks/starrocks/pull/6789)

- get_json_string 函数对 JSON 数组的处理不正确。[#7671](https://github.com/StarRocks/starrocks/pull/7671)

## 2.2.1

发布日期：2022年6月2日

### 改进

- 通过重构部分热点代码和减小锁粒度来优化数据加载性能和减少长尾延迟。[#6641](https://github.com/StarRocks/starrocks/pull/6641)

- 将每个查询在 BE 上部署的机器的 CPU 和内存使用信息添加到 FE 审计日志中。[#6208](https://github.com/StarRocks/starrocks/pull/6208) [#6209](https://github.com/StarRocks/starrocks/pull/6209)

- 支持在主键表和唯一键表中使用 JSON 数据类型。[#6544](https://github.com/StarRocks/starrocks/pull/6544)

- 通过减少锁粒度和去重 BE 报告请求，减轻 FE 负载。针对大量 BE 部署的情况下优化了报告性能，并解决了大集群中 Routine Load 任务卡住的问题。[#6293](https://github.com/StarRocks/starrocks/pull/6293)

### Bug 修复

修复以下 Bug：

- 当 StarRocks 解析 `SHOW FULL TABLES FROM DatabaseName` 语句中指定的转义字符时发生错误。[#6559](https://github.com/StarRocks/starrocks/issues/6559)

- FE 磁盘空间使用急剧上升（通过回滚 BDBJE 版本修复此 Bug）。[#6708](https://github.com/StarRocks/starrocks/pull/6708)

- 因在启用列扫描后的数据返回中找不到相关字段，导致 BE 出现故障（`enable_docvalue_scan=true`）。[#6600](https://github.com/StarRocks/starrocks/pull/6600)

## 2.2.0

发布日期：2022年5月22日

### 新特性

- [预览] 发布资源组管理功能。该功能允许 StarRocks 在处理来自不同租户的复杂查询和简单查询时，隔离资源并有效使用 CPU 和内存。

- [预览] 实现了基于 Java 的用户自定义函数（UDF）框架。这个框架支持使用符合 Java 语法编译的 UDF 来扩展 StarRocks 的功能。

- [预览] 主键表支持实时数据更新场景（如订单更新和多流连接）时，只更新特定列的数据。

- [预览] 支持 JSON 数据类型和 JSON 函数。

- 外部表可以用于查询 Apache Hudi 中的数据。这进一步改善了用户使用 StarRocks 进行数据湖分析的体验。更多信息，请参见[外部表](../data_source/External_table.md)。

- 添加如下函数：
  - ARRAY 函数：[array_agg](../sql-reference/sql-functions/array-functions/array_agg.md)，array_sort，array_distinct，array_join，reverse，array_slice，array_concat，array_difference，arrays_overlap 和 array_intersect
  - BITMAP 函数：bitmap_max 和 bitmap_min
  - 其他函数：retention 和 square

### 改进

- 重构了基于成本的优化器（CBO）的解析器和分析器，优化了代码结构，支持了 INSERT with Common Table Expression (CTE) 等语法。这些改进是为了提高复杂查询的性能，例如涉及 CTE 复用的查询。

- 优化了存储在 AWS Simple Storage Service (S3)、Alibaba Cloud Object Storage Service (OSS) 和 Tencent Cloud Object Storage (COS) 等云对象存储服务的 Apache Hive™ 外部表查询的性能。经优化后，基于对象存储的查询性能可与基于 HDFS 的查询相媲美。另外，支持 ORC 文件的延迟物化，并加速了小文件查询。更多信息，请参见[Apache Hive™ 外部表](../data_source/External_table.md)。

- 当使用外部表运行来自 Apache Hive™ 的查询时，StarRocks 自动通过消费 Hive metastore 事件（例如数据变更和分区变更）来增量更新缓存的元数据。StarRocks 还支持从 Apache Hive™ 查询 DECIMAL 和 ARRAY 类型的数据。更多信息，请参见[Apache Hive™ 外部表](../data_source/External_table.md)。

- UNION ALL 操作符的性能比以前快了 2 到 25 倍。

- 发布了一个支持自适应并行性和提供优化配置文件的 Pipeline 引擎，以提高高并发情况下简单查询的性能。

- 多个字符可以组合并用作 CSV 文件导入时的单个行分隔符。

### Bug 修复

- 如果数据加载或提交更改到基于主键表的表中，会发生死锁。[#4998](https://github.com/StarRocks/starrocks/pull/4998)

- 前端（FEs），包括运行 Oracle Berkeley DB Java 版本（BDB JE）的前端，不稳定。[#4428](https://github.com/StarRocks/starrocks/pull/4428)，[#4666](https://github.com/StarRocks/starrocks/pull/4666)，[#2](https://github.com/StarRocks/bdb-je/pull/2)

- 如果对大量数据调用 SUM 函数，返回的结果会遇到算术溢出的问题。[#3944](https://github.com/StarRocks/starrocks/pull/3944)

- ROUND 和 TRUNCATE 函数返回结果的精度不令人满意。[#4256](https://github.com/StarRocks/starrocks/pull/4256)

- 通过合成查询枪手（SQLancer）检测到一些漏洞。更多信息，请参阅 [SQLancer 相关问题](https://github.com/StarRocks/starrocks/issues?q=is:issue++label:sqlancer++milestone:2.2)。

### 其他

Flink-connector-starrocks 支持 Apache Flink® v1.14。

### 升级说明

- 如果您使用的是 StarRocks 版本高于 2.0.4 或 StarRocks 版本 2.1.x 高于 2.1.6，您可以在升级之前禁用平板克隆特性（`ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "0");` 和 `ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "0");`）。升级后，您可以启用此特性（`ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "2000");` 和 `ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "100");`）。

- 要回滚到升级前使用的前一个版本，请在每个 FE 的 **fe.conf** 文件中添加 `ignore_unknown_log_id` 参数，并将参数设置为 `true`。需要该参数是因为在 StarRocks v2.2.0 中添加了新类型的日志。如果不添加该参数，您将无法回滚到前一个版本。我们建议您在创建检查点后，在每个 FE 的 **fe.conf** 文件中将 `ignore_unknown_log_id` 参数设置为 `false`。然后，重启 FEs 以将 FEs 恢复到之前的配置。