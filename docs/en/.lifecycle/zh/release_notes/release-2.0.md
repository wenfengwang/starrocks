---
displayed_sidebar: English
---

# StarRocks 2.0 版本

## 2.0.9

发布日期：2022年8月6日

### Bug 修复

修复了以下错误：

- 对于 Broker Load 作业，如果 broker 负载过重，内部心跳可能会超时，从而导致数据丢失。[#8282](https://github.com/StarRocks/starrocks/issues/8282)
- 对于 Broker Load 作业，如果目标 StarRocks 表没有 `COLUMNS FROM PATH AS` 参数指定的列，则 BE 会停止运行。[#5346](https://github.com/StarRocks/starrocks/issues/5346)
- 某些查询被转发到 Leader FE，导致 `/api/query_detail` 操作返回关于 SQL 语句（例如 SHOW FRONTENDS）的错误执行信息。[#9185](https://github.com/StarRocks/starrocks/issues/9185)
- 当创建多个 Broker Load 作业以加载相同的 HDFS 数据文件时，如果一个作业遇到异常，其他作业也可能无法正确读取数据，因而失败。[#9506](https://github.com/StarRocks/starrocks/issues/9506)

## 2.0.8

发布日期：2022年7月15日

### Bug 修复

修复了以下错误：

- 反复切换 Leader FE 节点可能会导致所有加载作业挂起并失败。[#7350](https://github.com/StarRocks/starrocks/issues/7350)
- 当 MemTable 的内存使用估计超过 4GB 时，BE 会崩溃，因为在数据倾斜加载期间，某些字段可能占用大量内存资源。[#7161](https://github.com/StarRocks/starrocks/issues/7161)
- 重启 FE 后，由于大小写字母解析错误，物化视图的模式发生变化。[#7362](https://github.com/StarRocks/starrocks/issues/7362)
- 使用 Routine Load 从 Kafka 将 JSON 数据加载到 StarRocks 时，如果 JSON 数据中存在空行，空行之后的数据将丢失。[#8534](https://github.com/StarRocks/starrocks/issues/8534)

## 2.0.7

发布日期：2022年6月13日

### Bug 修复

修复了以下错误：

- 如果正在压缩的表中某列的重复值数量超过 0x40000000，压缩会被暂停。[#6513](https://github.com/StarRocks/starrocks/issues/6513)
- FE 重启后，由于 BDB JE v7.3.8 的问题，出现高 I/O 和异常增加的磁盘使用，且没有恢复正常的迹象。FE 回滚到 BDB JE v7.3.7 后恢复正常。[#6634](https://github.com/StarRocks/starrocks/issues/6634)

## 2.0.6

发布日期：2022年5月25日

### Bug 修复

修复了以下错误：

- 某些图形用户界面（GUI）工具会自动配置 `set_sql_limit` 变量。因此，SQL 语句 ORDER BY LIMIT 被忽略，导致查询返回的行数不正确。[#5966](https://github.com/StarRocks/starrocks/issues/5966)
- 如果 Colocation Group（CG）包含大量表，且数据频繁加载到这些表中，CG 可能无法保持 `stable` 状态。在这种情况下，JOIN 语句不支持 Colocate Join 操作。StarRocks 已优化，可以在数据加载期间等待更长时间，以最大化加载数据的 tablet 副本的完整性。
- 如果由于负载过重或网络延迟等原因导致一些副本加载失败，将触发这些副本的克隆。在这种情况下，可能会发生死锁，导致进程负载低但大量请求超时。[#5646](https://github.com/StarRocks/starrocks/issues/5646) [#6290](https://github.com/StarRocks/starrocks/issues/6290)
- 在对使用 Primary Key 表的 schema 进行更改后，向该表加载数据时，可能会出现 “duplicate key xxx” 错误。[#5878](https://github.com/StarRocks/starrocks/issues/5878)
- 如果对数据库执行 DROP SCHEMA 语句，数据库将被强制删除且无法恢复。[#6201](https://github.com/StarRocks/starrocks/issues/6201)

## 2.0.5

发布日期：2022年5月13日

升级建议：本版本修复了一些与存储数据或数据查询正确性相关的关键错误。我们建议您尽早升级您的 StarRocks 集群。

### Bug 修复

修复了以下错误：

- [关键错误] 由于 BE 故障可能导致数据丢失。此错误通过引入一种机制得到修复，该机制用于同时向多个 BE 发布特定版本。[#3140](https://github.com/StarRocks/starrocks/issues/3140)

- [关键错误] 如果在特定数据摄取阶段迁移 tablet，数据会继续写入存储 tablet 的原始磁盘。因此，数据丢失，查询无法正常运行。[#5160](https://github.com/StarRocks/starrocks/issues/5160)

- [关键错误] 在执行多次 DELETE 操作后运行查询时，如果对查询中的低基数列进行了优化，可能会得到不正确的查询结果。[#5712](https://github.com/StarRocks/starrocks/issues/5712)

- [关键错误] 如果查询包含一个用于组合 DOUBLE 类型列和 VARCHAR 类型列的 JOIN 子句，则查询结果可能不正确。[#5809](https://github.com/StarRocks/starrocks/pull/5809)

- 在某些情况下，当您将数据加载到 StarRocks 集群时，某些特定版本的副本在副本生效之前被 FE 标记为有效。此时，如果查询特定版本的数据，StarRocks 无法找到数据并报错。[#5153](https://github.com/StarRocks/starrocks/issues/5153)

- 如果 `SPLIT` 函数中的参数设置为 `NULL`，您的 StarRocks 集群的 BE 可能会停止运行。[#4092](https://github.com/StarRocks/starrocks/issues/4092)
```
- 当您的集群从 Apache Doris 0.13 升级到 StarRocks 1.19.x 并运行一段时间后，进一步升级到 StarRocks 2.0.1 可能会失败。[#5309](https://github.com/StarRocks/starrocks/issues/5309)

## 2.0.4

发布日期：2022年4月18日

### Bug 修复

修复了以下错误：

- 删除列、添加新分区和克隆 tablet 后，新旧 tablet 中列的唯一 ID 可能不相同，这可能会导致 BE 停止工作，因为系统使用了共享的 tablet schema。[#4514](https://github.com/StarRocks/starrocks/issues/4514)
- 当数据加载到 StarRocks 外部表时，如果目标 StarRocks 集群配置的 FE 不是 Leader，将会导致 FE 停止工作。[#4573](https://github.com/StarRocks/starrocks/issues/4573)
- 当 Duplicate Key 表执行 schema change 并同时创建物化视图时，查询结果可能不正确。[#4839](https://github.com/StarRocks/starrocks/issues/4839)
- 修复了 BE 失败可能导致数据丢失的问题（通过使用批量发布版本机制解决）。[#3140](https://github.com/StarRocks/starrocks/issues/3140)

## 2.0.3

发布日期：2022年3月14日

### Bug 修复

修复了以下错误：

- 当 BE 节点处于假死状态时查询失败。
- 当单表连接没有合适的执行计划时，查询会失败。[#3854](https://github.com/StarRocks/starrocks/issues/3854)
- 当 FE 节点收集信息以构建全局字典进行低基数优化时，可能会出现死锁问题。[#3839](https://github.com/StarRocks/starrocks/issues/3839)

## 2.0.2

发布日期：2022年3月2日

### 改进

- 内存使用得到优化。用户可以指定 `label_keep_max_num` 参数来控制一段时间内保留的最大加载作业数量。这样可以防止因 FE 内存占用过高而导致的 Full GC。

### Bug 修复

修复了以下错误：

- 当列解码器遇到异常时，BE 节点会失败。
- 在命令中指定 `jsonpaths` 时，自动 `__op` 映射不生效。
- 使用 Broker Load 加载数据期间，如果源数据发生变化，BE 节点可能会失败。
- 在创建物化视图后，某些 SQL 语句可能会报错。
- 如果 SQL 子句包含支持低基数优化的全局字典谓词和不支持的谓词，则查询可能会失败。

## 2.0.1

发布日期：2022年1月21日

### 改进

- 当 StarRocks 使用外部表查询 Hive 数据时，可以读取 Hive 的 `implicit_cast` 操作。[#2829](https://github.com/StarRocks/starrocks/pull/2829)
- 使用读/写锁来解决 StarRocks CBO 收集统计信息以支持高并发查询时的高 CPU 使用率问题。[#2901](https://github.com/StarRocks/starrocks/pull/2901)
- 对 CBO 统计数据收集和 UNION 运算符进行了优化。

### Bug 修复

- 修复了副本全局字典不一致导致的查询错误。[#2700](https://github.com/StarRocks/starrocks/pull/2700) [#2765](https://github.com/StarRocks/starrocks/pull/2765)
- 修复了数据加载时参数 `exec_mem_limit` 不生效的错误。[#2693](https://github.com/StarRocks/starrocks/pull/2693)
    > 参数 `exec_mem_limit` 指定数据加载时每个 BE 节点的内存限制。
- 修复了导入数据到 Primary Key 表时出现的 OOM 错误。[#2743](https://github.com/StarRocks/starrocks/pull/2743) [#2777](https://github.com/StarRocks/starrocks/pull/2777)
- 修复了 StarRocks 使用外部表查询大型 MySQL 表时 BE 节点停止响应的错误。[#2881](https://github.com/StarRocks/starrocks/pull/2881)

### 行为变更

StarRocks 可以使用外部表访问 Hive 及其基于 AWS S3 的外部表。但是，用于访问 S3 数据的 jar 文件太大，StarRocks 的二进制包中不包含该 jar 文件。如果您想使用这个 jar 文件，可以从 [Hive_s3_lib](https://releases.starrocks.io/resources/hive_s3_jar.tar.gz) 下载。

## 2.0.0

发布日期：2022年1月5日

### 新特性

- 外部表
  - [实验性功能] 支持 S3 上的 Hive 外部表
  - 对外部表支持 DecimalV3 类型 [#425](https://github.com/StarRocks/starrocks/pull/425)
- 实现了将复杂表达式下推到存储层进行计算，从而获得性能提升
- Primary Key 正式发布，支持 Stream Load、Broker Load、Routine Load，同时还提供了基于 Flink-cdc 的 MySQL 数据秒级同步工具

### 改进

- 算术运算符优化
  - 优化了低基数字典的性能 [#791](https://github.com/StarRocks/starrocks/pull/791)
  - 优化了 `int` 类型单表扫描的性能 [#273](https://github.com/StarRocks/starrocks/issues/273)
  - 优化了高基数 `count(distinct int)` 的性能 [#139](https://github.com/StarRocks/starrocks/pull/139) [#250](https://github.com/StarRocks/starrocks/pull/250) [#544](https://github.com/StarRocks/starrocks/pull/544) [#570](https://github.com/StarRocks/starrocks/pull/570)
  - 在实现层面优化了 `Group by int`、`limit`、`case when`、`not equal`

- 内存管理优化
  - 重构了内存统计及控制框架，精确统计内存使用，彻底解决了 OOM 问题
  - 优化了元数据内存使用
  - 解决了大量内存释放长时间卡在执行线程的问题
  - 添加了进程优雅退出机制并支持内存泄漏检查 [#1093](https://github.com/StarRocks/starrocks/pull/1093)

### 错误修复

- 修复了 Hive 外部表在大量获取元数据时超时的问题。
- 修复了物化视图创建错误提示不清晰的问题。
- 修复了向量化引擎中 `like` 实现的错误 [#722](https://github.com/StarRocks/starrocks/pull/722)
- 修复了 `alter table` 中解析谓词 `is` 的错误 [#725](https://github.com/StarRocks/starrocks/pull/725)
- 修复了 `curdate` 函数无法格式化日期的问题
- 修复了矢量化引擎中 `like` 实现的问题。[#722](https://github.com/StarRocks/starrocks/pull/722)
- 修复了在 `alter table` 语句中解析谓词时的错误。[#725](https://github.com/StarRocks/starrocks/pull/725)