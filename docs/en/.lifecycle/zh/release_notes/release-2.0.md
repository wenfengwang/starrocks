---
displayed_sidebar: English
---

# StarRocks 2.0 版本

## 2.0.9

发布日期：2022年8月6日

### Bug 修复

已修复以下问题：

- 对于 Broker Load 作业，如果代理负载过重，内部心跳可能会超时，导致数据丢失。[#8282](https://github.com/StarRocks/starrocks/issues/8282)
- 对于 Broker Load 作业，如果目标 StarRocks 表中没有 `COLUMNS FROM PATH AS` 参数指定的列，BE 将停止运行。[#5346](https://github.com/StarRocks/starrocks/issues/5346)
- 一些查询被转发到 Leader FE，导致 `/api/query_detail` 操作返回有关 SQL 语句（如 SHOW FRONTENDS）的不正确执行信息。[#9185](https://github.com/StarRocks/starrocks/issues/9185)
- 当创建多个 Broker Load 作业来加载相同的 HDFS 数据文件时，如果一个作业遇到异常，其他作业可能无法正确读取数据，因此失败。[#9506](https://github.com/StarRocks/starrocks/issues/9506)

## 2.0.8

发布日期：2022年7月15日

### Bug 修复

已修复以下问题：

- 反复切换 Leader FE 节点可能导致所有加载作业挂起并失败。[#7350](https://github.com/StarRocks/starrocks/issues/7350)
- 当 MemTable 的内存使用估计超过 4GB 时，BE 会崩溃，因为在数据倾斜期间，某些字段可能会占用大量内存资源。[#7161](https://github.com/StarRocks/starrocks/issues/7161)
- 重启 FE 后，由于大小写解析不正确，物化视图的模式发生了变化。[#7362](https://github.com/StarRocks/starrocks/issues/7362)
- 当使用 Routine Load 将 Kafka 中的 JSON 数据加载到 StarRocks 中时，如果 JSON 数据中有空白行，则空白行后的数据将丢失。[#8534](https://github.com/StarRocks/starrocks/issues/8534)

## 2.0.7

发布日期：2022年6月13日

### Bug 修复

已修复以下问题：

- 如果正在压缩的表的列中的重复值数超过 0x40000000，则将暂停压缩。[#6513](https://github.com/StarRocks/starrocks/issues/6513)
- FE 重启后，由于 BDB JE v7.3.8 中存在一些问题，会遇到高 I/O 和异常增加的磁盘使用率，并且没有恢复正常的迹象。FE 回滚到 BDB JE v7.3.7 后恢复正常。[#6634](https://github.com/StarRocks/starrocks/issues/6634)

## 2.0.6

发布日期：2022年5月25日

### Bug 修复

已修复以下问题：

- 一些图形用户界面（GUI）工具会自动配置 `set_sql_limit` 变量。因此，将忽略 SQL 语句中的 ORDER BY LIMIT，导致查询返回的行数不正确。[#5966](https://github.com/StarRocks/starrocks/issues/5966)
- 如果 colocation group（CG）包含大量表，并且数据频繁加载到表中，则 CG 可能无法保持 `stable` 状态。在这种情况下，JOIN 语句不支持 Colocate Join 操作。StarRocks 已经优化，在数据加载过程中等待的时间会稍长一些，以最大限度地提高加载数据的平板电脑副本的完整性。
- 如果由于负载过重、网络延迟过高等原因导致少数副本加载失败，则会触发克隆。在这种情况下，可能会出现死锁，这可能会导致进程负载较低但大量请求超时的情况。[#5646](https://github.com/StarRocks/starrocks/issues/5646) [#6290](https://github.com/StarRocks/starrocks/issues/6290)
- 更改使用主键表的表的架构后，当数据加载到该表中时，可能会出现“重复键 xxx”错误。[#5878](https://github.com/StarRocks/starrocks/issues/5878)
- 如果对数据库执行 DROP SCHEMA 语句，则该数据库将被强制删除，无法恢复。[#6201](https://github.com/StarRocks/starrocks/issues/6201)

## 2.0.5

发布日期：2022年5月13日

升级建议：此版本修复了一些与存储数据或数据查询的正确性相关的关键错误。建议您尽早升级 StarRocks 集群。

### Bug 修复

已修复以下问题：

- [严重错误] 由于 BE 故障，数据可能会丢失。通过引入一种机制来修复此 bug，该机制用于一次将特定版本发布到多个 BE。[#3140](https://github.com/StarRocks/starrocks/issues/3140)
- [严重错误] 如果平板电脑在特定的数据引入阶段进行迁移，数据将继续写入存储平板电脑的原始磁盘。因此，数据会丢失，查询无法正常运行。[#5160](https://github.com/StarRocks/starrocks/issues/5160)
- [严重错误] 在执行多个 DELETE 操作后运行查询时，如果对查询执行低基数列的优化，可能会获得不正确的查询结果。[#5712](https://github.com/StarRocks/starrocks/issues/5712)
- [严重错误] 如果查询中包含一个 JOIN 子句，该子句用于将具有 DOUBLE 值的列和具有 VARCHAR 值的列组合在一起，则查询结果可能不正确。[#5809](https://github.com/StarRocks/starrocks/pull/5809)
- 在某些情况下，当您将数据加载到 StarRocks 集群时，部分特定版本的副本会在副本生效前被 FE 标记为有效。此时，查询特定版本的数据时，StarRocks 无法找到数据并报错。[#5153](https://github.com/StarRocks/starrocks/issues/5153)
- 如果函数中的参数 `SPLIT` 设置为 `NULL`，StarRocks 集群的 BE 可能会停止运行。[#4092](https://github.com/StarRocks/starrocks/issues/4092)  
- 当您的集群从Apache Doris 0.13升级到StarRocks 1.19.x并持续运行一段时间后，进一步升级到StarRocks 2.0.1可能会失败。 [#5309](https://github.com/StarRocks/starrocks/issues/5309)

## 2.0.4

发布日期：2022年4月18日

### Bug修复

已修复以下问题：

- 在删除列、添加新分区和克隆平板电脑后，新旧平板电脑中列的唯一ID可能不同，这可能会导致BE停止工作，因为系统使用共享平板电脑架构。 [#4514](https://github.com/StarRocks/starrocks/issues/4514)
- 当数据加载到StarRocks外部表时，如果目标StarRocks集群的FE不是Leader，会导致FE停止工作。 [#4573](https://github.com/StarRocks/starrocks/issues/4573)
- 当Duplicate Key表执行架构更改并同时创建物化视图时，查询结果可能不正确。 [#4839](https://github.com/StarRocks/starrocks/issues/4839)
- 由于BE故障可能导致数据丢失的问题（通过使用批量发布版本解决）。 [#3140](https://github.com/StarRocks/starrocks/issues/3140)

## 2.0.3

发布日期：2022年3月14日

### Bug修复

已修复以下问题：

- 当BE节点处于挂起动画状态时，查询失败。
- 当单个平板电脑表联接没有适当的执行计划时，查询将失败。 [#3854](https://github.com/StarRocks/starrocks/issues/3854)
- 当FE节点收集信息构建全局字典以进行低基数优化时，可能会出现死锁问题。 [#3839](https://github.com/StarRocks/starrocks/issues/3839)

## 2.0.2

发布日期：2022年3月2日

### 改进

- 优化了内存使用。用户可以指定label_keep_max_num参数来控制一段时间内要保留的最大加载作业数。这可以防止由于频繁数据加载导致FE内存使用过高而引起的GC满量。

### Bug修复

已修复以下问题：

- 当列解码器遇到异常时，BE节点将失败。
- 在用于加载JSON数据的命令中指定jsonpaths时，自动__op映射不会生效。
- 由于在使用Broker Load加载数据期间源数据发生变化，BE节点失败。
- 在创建物化视图后，某些SQL语句报错。
- 如果SQL子句包含支持全局字典进行低基数优化的谓词和不支持全局字典的谓词，则查询可能会失败。

## 2.0.1

发布日期：2022年1月21日

### 改进

- 当StarRocks使用外部表查询Hive数据时，可以读取Hive的implicit_cast操作。 [#2829](https://github.com/StarRocks/starrocks/pull/2829)
- 使用读写锁来修复StarRocks CBO收集统计信息以支持高并发查询时CPU使用率过高的问题。 [#2901](https://github.com/StarRocks/starrocks/pull/2901)
- 优化了CBO统计和UNION算子。

### Bug修复

- 修复了由于副本全局字典不一致导致的查询错误。 [#2700](https://github.com/StarRocks/starrocks/pull/2700) [#2765](https://github.com/StarRocks/starrocks/pull/2765)
- 修复了数据加载时参数`exec_mem_limit`不生效的问题。 [#2693](https://github.com/StarRocks/starrocks/pull/2693)
  > 参数`exec_mem_limit`指定数据加载期间每个BE节点的内存限制。
- 修正了导入主键表时出现的OOM错误。 [#2743](https://github.com/StarRocks/starrocks/pull/2743) [#2777](https://github.com/StarRocks/starrocks/pull/2777)
- 修复了StarRocks使用外部表查询大型MySQL表时BE节点停止响应的问题。 [#2881](https://github.com/StarRocks/starrocks/pull/2881)

### 行为变更

StarRocks现在可以使用外部表访问Hive及其基于AWS S3的外部表。但是，用于访问S3数据的jar文件过大，StarRocks的二进制包中不包含此jar文件。如果要使用此jar文件，可以从[Hive_s3_lib](https://releases.starrocks.io/resources/hive_s3_jar.tar.gz)下载。

## 2.0.0

发布日期：2022年1月5日

### 新功能

- 外部表
  - [实验功能]支持S3上的Hive外部表
  - DecimalV3支持外部表 [#425](https://github.com/StarRocks/starrocks/pull/425)
- 实现了将复杂表达式下推到存储层进行计算，从而获得性能提升
- Primary Key正式发布，支持Stream Load、Broker Load、Routine Load，并且基于Flink-cdc为MySQL数据提供了二级同步工具

### 改进

- 优化了算术运算符
  - 优化了低基数字典的性能 [#791](https://github.com/StarRocks/starrocks/pull/791)
  - 优化了单表中int的扫描性能 [#273](https://github.com/StarRocks/starrocks/issues/273)
  - 优化了高基数`count(distinct int)`的性能  #[139](https://github.com/StarRocks/starrocks/pull/139) [ #250](https://github.com/StarRocks/starrocks/pull/250)#544#[570](https://github.com/StarRocks/starrocks/pull/544)[  ](https://github.com/StarRocks/starrocks/pull/570)
  - 在实现层面优化了`Group by int` / `limit` / `case when` / `not equal`的性能
- 优化了内存管理
  - 重构了内存统计和控制框架，精确统计内存使用情况，彻底解决了OOM问题
  - 优化了元数据的内存使用
  - 解决了长时间卡在执行线程中的大内存释放问题
  - 增加了进程平滑退出机制，并支持内存泄漏检查 [#1093](https://github.com/StarRocks/starrocks/pull/1093)

### Bug修复
- 修复了 Hive 外部表在获取大量元数据时超时的问题。
- 修复了物化视图创建时错误消息不清晰的问题。
- 修复了矢量化引擎中 like 的实现问题 [#722](https://github.com/StarRocks/starrocks/pull/722)
- 修复了在 `alter table` 中解析谓词时的错误问题 [#725](https://github.com/StarRocks/starrocks/pull/725)
- 修复了 `curdate` 函数无法格式化日期的问题