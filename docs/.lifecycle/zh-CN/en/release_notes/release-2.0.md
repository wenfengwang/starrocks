---
displayed_sidebar: "Chinese"
---

# StarRocks版本2.0

## 2.0.9

发布日期：2022年8月6日

### Bug修复

已修复以下bug：

- 对于Broker Load作业，如果broker负载过重，内部心跳可能超时，导致数据丢失。[#8282](https://github.com/StarRocks/starrocks/issues/8282)
- 对于Broker Load作业，如果目标StarRocks表未指定`COLUMNS FROM PATH AS`参数指定的列，BE会停止运行。[#5346](https://github.com/StarRocks/starrocks/issues/5346)
- 一些查询被转发到Leader FE，导致`/api/query_detail`操作返回关于SQL语句（例如SHOW FRONTENDS）的不正确的执行信息。[#9185](https://github.com/StarRocks/starrocks/issues/9185)
- 当创建多个Broker Load作业以加载相同的HDFS数据文件时，如果一个作业遇到异常，其他作业可能无法正确读取数据，因此也会失败。[#9506](https://github.com/StarRocks/starrocks/issues/9506)

## 2.0.8

发布日期：2022年7月15日

### Bug修复

已修复以下bug：

- 反复切换Leader FE节点可能导致所有加载作业挂起和失败。[#7350](https://github.com/StarRocks/starrocks/issues/7350)
- 当MemTable的内存使用量估计超过4GB时，BE会崩溃，因为在加载过程中发生数据倾斜时，某些字段可能占用大量内存资源。[#7161](https://github.com/StarRocks/starrocks/issues/7161)
- 重新启动FE后，由于对大写和小写字母解析不正确，导致物化视图的架构发生变化。[#7362](https://github.com/StarRocks/starrocks/issues/7362)
- 当您使用例行加载将来自Kafka的JSON数据加载到StarRocks时，如果JSON数据中存在空行，则空行后的数据将丢失。[#8534](https://github.com/StarRocks/starrocks/issues/8534)

## 2.0.7

发布日期：2022年6月13日

### Bug修复

已修复以下bug：

- 如果表在进行压缩时某列中的重复值数量超过0x40000000，压缩将被挂起。[#6513](https://github.com/StarRocks/starrocks/issues/6513)
- FE重新启动后，由于BDB JE v7.3.8中存在一些问题，并且未显示恢复正常迹象，导致I/O开销高和磁盘使用量异常增长。FE回滚至BDB JE v7.3.7后，FE会恢复正常。[#6634](https://github.com/StarRocks/starrocks/issues/6634)

## 2.0.6

发布日期：2022年5月25日

### Bug修复

已修复以下bug：

- 一些图形用户界面（GUI）工具会自动配置`set_sql_limit`变量。因此，SQL语句ORDER BY LIMIT被忽略，导致查询返回不正确的行数。[#5966](https://github.com/StarRocks/starrocks/issues/5966)
- 如果一个Colocation Group（CG）包含大量表，并且频繁向表加载数据，CG可能无法保持“stable”状态。在这种情况下，JOIN语句不支持Colocate Join操作。StarRocks已经优化，等待更长时间的数据加载。这样，可以最大化加载数据的tablet副本的完整性。
- 如果部分副本由于负载过重或网络延迟过高等原因无法加载，会触发这些副本的克隆。在这种情况下，可能会发生死锁，导致进程的负载很低，但大量请求超时。[#5646](https://github.com/StarRocks/starrocks/issues/5646) [#6290](https://github.com/StarRocks/starrocks/issues/6290)
- 当使用Primary Key表的表架构发生更改后，向该表加载数据可能会出现“duplicate key xxx”错误。[#5878](https://github.com/StarRocks/starrocks/issues/5878)
- 如果在数据库上执行DROP SCHEMA语句，数据库将被强制删除且无法恢复。[#6201](https://github.com/StarRocks/starrocks/issues/6201)

## 2.0.5

发布日期：2022年5月13日

升级建议：在此版本中修复了与存储数据的正确性或数据查询有关的一些关键bug。建议尽快升级StarRocks集群。

### Bug修复

已修复以下bug：

- [关键bug] BE故障可能导致数据丢失。通过引入一种机制来同时发布特定版本到多个BE来修复此bug。[#3140](https://github.com/StarRocks/starrocks/issues/3140)

- [关键bug] 如果在特定的数据摄入阶段迁移了tablet，数据会继续写入存储了tablet的原始磁盘。结果，数据丢失，无法正常运行查询。[#5160](https://github.com/StarRocks/starrocks/issues/5160)

- [关键bug] 在执行多次DELETE操作后运行查询时，如果为查询执行低基数列的优化，可能会获取不正确的查询结果。[#5712](https://github.com/StarRocks/starrocks/issues/5712)

- [关键bug] 如果查询包含用于将DOUBLE值列和VARCHAR值列结合的JOIN子句，则查询结果可能不正确。[#5809](https://github.com/StarRocks/starrocks/pull/5809)

- 在某些情况下，当您将数据加载到StarRocks集群中时，FE在复制生效之前就将特定版本的某些副本标记为有效。此时，如果查询特定版本的数据，则StarRocks无法找到数据并报告错误。[#5153](https://github.com/StarRocks/starrocks/issues/5153)

- 如果`SPLIT`函数中的参数设置为`NULL`，则StarRocks集群的BE将停止运行。[#4092](https://github.com/StarRocks/starrocks/issues/4092)

- 将集群从Apache Doris 0.13升级到StarRocks 1.19.x并运行一段时间后，进一步升级至StarRocks 2.0.1可能会失败。[#5309](https://github.com/StarRocks/starrocks/issues/5309)

## 2.0.4

发布日期：2022年4月18日

### Bug修复

已修复以下bug：

- 删除列、添加新分区和克隆tablet后，旧和新tablet中列的唯一ID可能不相同，这可能导致BE停止工作，因为系统使用共享tablet架构。[#4514](https://github.com/StarRocks/starrocks/issues/4514)
- 将数据加载到StarRocks外部表时，如果目标StarRocks集群的配置FE不是Leader，则会导致FE停止工作。[#4573](https://github.com/StarRocks/starrocks/issues/4573)
- 当Duplicate Key表执行架构更改并同时创建物化视图时，查询结果可能不正确。[#4839](https://github.com/StarRocks/starrocks/issues/4839)
- 由于BE故障可能导致数据丢失的问题（使用批量发布版本解决）。[#3140](https://github.com/StarRocks/starrocks/issues/3140)

## 2.0.3

发布日期：2022年3月14日

### Bug修复

已修复以下bug：

- 当BE节点处于挂起状态时，查询失败。
- 当单tablet表联接没有适当的执行计划时，查询失败。[#3854](https://github.com/StarRocks/starrocks/issues/3854)
- FE节点在收集信息以构建低基数优化的全局字典时可能会发生死锁问题。[#3839](https://github.com/StarRocks/starrocks/issues/3839)

## 2.0.2

发布日期：2022年3月2日

### 改进

- 优化了内存使用。用户可以指定label_keep_max_num参数来控制一段时间内保留的加载作业的最大数量。这可以防止由于频繁数据加载导致FE内存使用过高而引起的full GC。

### Bug修复

已修复以下bug：

- 当列解码器遇到异常时，BE节点会失败。
- 当在用于加载JSON数据的命令中指定jsonpaths时，自动op映射不会生效。
- 在使用Broker Load进行数据加载时，BE节点因源数据发生变化而失败。
- 在创建了物化视图后，一些SQL语句报错。
- 如果一个SQL子句包含支持全局字典进行低基数优化的谓词和一个不支持的谓词，可能会导致查询失败。

## 2.0.1

发布日期：2022年1月21日

### 改进

- 当StarRocks使用外部表查询Hive数据时，可以读取Hive的implicit_cast操作。[#2829](https://github.com/StarRocks/starrocks/pull/2829)
- 使用读/写锁来解决StarRocks CBO在支持高并发查询时收集统计信息导致CPU使用率过高的问题。[#2901](https://github.com/StarRocks/starrocks/pull/2901)
- 优化CBO统计信息收集和UNION运算符。

### Bug修复

- 修复因副本的全局字典不一致导致的查询错误。[#2700](https://github.com/StarRocks/starrocks/pull/2700) [#2765](https://github.com/StarRocks/starrocks/pull/2765)
- 修复数据加载期间参数`exec_mem_limit`不生效的错误。[#2693](https://github.com/StarRocks/starrocks/pull/2693)
  > 参数`exec_mem_limit`指定数据加载期间每个BE节点的内存限制。
- 修复将数据导入主键表时发生的OOM错误。[#2743](https://github.com/StarRocks/starrocks/pull/2743) [#2777](https://github.com/StarRocks/starrocks/pull/2777)
- 修复当StarRocks使用外部表查询大型MySQL表时BE节点停止响应的错误。[#2881](https://github.com/StarRocks/starrocks/pull/2881)

### 行为变更

StarRocks现在可以使用外部表访问Hive及其基于AWS S3的外部表。然而，用于访问S3数据的jar文件过大，StarRocks的二进制包中不包含此jar文件。如果要使用此jar文件，可以从[Hive_s3_lib](https://releases.starrocks.io/resources/hive_s3_jar.tar.gz)下载。

## 2.0.0

发布日期：2022年1月5日

### 新特性

- 外部表
  - [实验性功能]支持Hive外部表在S3上的使用
  - DecimalV3支持外部表[#425](https://github.com/StarRocks/starrocks/pull/425)
- 实现将复杂表达式下推到存储层进行计算，从而获得性能提升
- 正式发布主键，支持Stream Load、Broker Load、Routine Load，并且基于Flink-cdc为MySQL数据提供二级同步工具

### 改进

- 算术运算符优化
  - 优化低基数的字典性能[#791](https://github.com/StarRocks/starrocks/pull/791)
  - 优化单表int的扫描性能[#273](https://github.com/StarRocks/starrocks/issues/273)
  - 优化`count(distinct int)`的性能，处理高基数[#139](https://github.com/StarRocks/starrocks/pull/139) [#250](https://github.com/StarRocks/starrocks/pull/250)  [#544](https://github.com/StarRocks/starrocks/pull/544) [#570](https://github.com/StarRocks/starrocks/pull/570)
  - 在实现层优化`Group by int` / `limit` / `case when` / `not equal`
- 内存管理优化
  - 重构内存统计和控制框架，准确计算内存使用量，完全解决OOM问题
  - 优化元数据内存使用
  - 解决长时间执行线程中大内存释放卡住的问题
  - 添加进程优雅退出机制，支持内存泄漏检查[#1093](https://github.com/StarRocks/starrocks/pull/1093)

### Bug修复

- 修复Hive外部表在大量情况下获取元数据超时的问题。
- 修复物化视图创建错误消息不明确的问题。
- 修复向量化引擎中的like实现[#722](https://github.com/StarRocks/starrocks/pull/722)
- 修复解析谓词在`alter table`中的错误[#725](https://github.com/StarRocks/starrocks/pull/725)
- 修复`curdate`函数无法格式化日期的问题