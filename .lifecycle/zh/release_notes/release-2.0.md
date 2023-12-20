---
displayed_sidebar: English
---

# StarRocks版本2.0

## 2.0.9

发布日期：2022年8月6日

### 错误修复

修复了以下错误：

- 对于Broker Load作业，如果broker负载过重，内部心跳可能会超时，导致数据丢失。[#8282](https://github.com/StarRocks/starrocks/issues/8282)
- 对于Broker Load作业，如果目标StarRocks表缺少`COLUMNS FROM PATH AS`参数指定的列，则BE会停止运行。[#5346](https://github.com/StarRocks/starrocks/issues/5346)
- 某些查询被转发到Leader FE，导致`/api/query_detail`操作返回关于SQL语句（例如SHOW FRONTENDS）的错误执行信息。[#9185](https://github.com/StarRocks/starrocks/issues/9185)
- 当创建多个Broker Load作业来加载相同的HDFS数据文件时，如果一个作业遇到异常，其他作业可能也无法正确读取数据，进而导致失败。[#9506](https://github.com/StarRocks/starrocks/issues/9506)

## 2.0.8

发布日期：2022年7月15日

### 错误修复

修复了以下错误：

- 频繁切换Leader FE节点可能导致所有加载作业挂起并失败。[#7350](https://github.com/StarRocks/starrocks/issues/7350)
- 当MemTable的内存使用估计超过4GB时，BE可能会崩溃，因为在数据倾斜的加载期间，某些字段可能占用大量内存资源。[#7161](https://github.com/StarRocks/starrocks/issues/7161)
- 重启FE后，由于大小写解析错误，物化视图的模式发生变化。[#7362](https://github.com/StarRocks/starrocks/issues/7362)
- 当你使用Routine Load将JSON数据从Kafka加载到StarRocks时，如果JSON数据中存在空行，空行之后的数据将会丢失。[#8534](https://github.com/StarRocks/starrocks/issues/8534)

## 2.0.7

发布日期：2022年6月13日

### 错误修复

修复了以下错误：

- 如果表的压缩过程中某一列的重复值数量超过0x40000000，压缩会被暂停。[#6513](https://github.com/StarRocks/starrocks/issues/6513)
- FE重启后，由于BDB JE v7.3.8存在问题，导致I/O高和磁盘使用量异常增加，并且没有恢复正常的迹象。在回滚到BDB JE v7.3.7后FE恢复正常。[#6634](https://github.com/StarRocks/starrocks/issues/6634)

## 2.0.6

发布日期：2022年5月25日

### 错误修复

修复了以下错误：

- 一些图形用户界面（GUI）工具会自动配置`set_sql_limit`变量，导致SQL语句ORDER BY LIMIT被忽略，因此查询返回的行数不正确。[#5966](https://github.com/StarRocks/starrocks/issues/5966)
- 如果一个Colocation Group（CG）包含大量表格，并且数据经常被加载到这些表格中，CG可能无法保持稳定状态。在这种情况下，JOIN语句不支持Colocate Join操作。StarRocks已经优化，以便在数据加载期间稍等片刻，从而最大化数据加载到的tablet副本的完整性。
- 如果由于负载过重或网络延迟等原因导致一些副本加载失败，就会触发对这些副本的克隆操作。在这种情况下，可能会发生死锁，导致虽然进程的加载较低，但大量请求超时。[#5646](https://github.com/StarRocks/starrocks/issues/5646) [#6290](https://github.com/StarRocks/starrocks/issues/6290)
- 在使用主键表的表结构发生变化后，加载数据时可能会出现“duplicate key xxx”错误。[#5878](https://github.com/StarRocks/starrocks/issues/5878)
- 如果执行`DROP SCHEMA`语句删除数据库，那么数据库将被强制删除且无法恢复。[#6201](https://github.com/StarRocks/starrocks/issues/6201)

## 2.0.5

发布日期：2022年5月13日

升级建议：此版本修复了一些与存储数据或数据查询正确性相关的关键错误。我们建议您尽早升级您的StarRocks集群。

### 错误修复

修复了以下错误：

- 【关键错误】Data may be lost as a result of BE failures. This bug is fixed by introducing a mechanism that is used to publish a specific version to multiple BEs at a time. [#3140](https://github.com/StarRocks/starrocks/issues/3140)

- [关键错误] 如果在特定数据摄取阶段迁移Tablet，数据将继续写入存储Tablet的原始磁盘。因此，数据将丢失，查询无法正常运行。[#5160](https://github.com/StarRocks/starrocks/issues/5160)

- 【关键错误】在执行多次DELETE操作后运行查询时，如果对查询进行了低基数列优化，则可能获得不正确的查询结果。[#5712](https://github.com/StarRocks/starrocks/issues/5712)

- 【关键错误】如果查询包含用于组合DOUBLE值列和VARCHAR值列的JOIN子句，则查询结果可能不正确。[#5809](https://github.com/StarRocks/starrocks/pull/5809)

- 在某些情况下，当您将数据加载到StarRocks集群时，FE在副本生效之前将某些特定版本的副本标记为有效。此时，如果您查询特定版本的数据，StarRocks可能找不到数据并报错。[#5153](https://github.com/StarRocks/starrocks/issues/5153)

- 如果一个参数在`SPLIT`函数中设置为`NULL`，你的StarRocks集群的BE可能会停止运行。[#4092](https://github.com/StarRocks/starrocks/issues/4092)

- 当您的集群从Apache Doris 0.13升级到StarRocks 1.19.x并运行一段时间后，进一步升级到StarRocks 2.0.1可能会失败。[#5309](https://github.com/StarRocks/starrocks/issues/5309)

## 2.0.4

发布日期：2022年4月18日

### 错误修复

修复了以下错误：

- 删除列、添加新分区和克隆Tablet后，新旧Tablet中列的唯一ID可能不一致，这可能导致BE停止工作，因为系统使用共享的Tablet模式。[#4514](https://github.com/StarRocks/starrocks/issues/4514)
- 当数据加载到StarRocks外部表时，如果配置的FE目标StarRocks集群不是Leader，它将导致FE停止工作。[#4573](https://github.com/StarRocks/starrocks/issues/4573)
- 当一个Duplicate Key表执行模式变更并同时创建[物化视图](https://github.com/StarRocks/starrocks/issues/4839)时，查询结果可能不正确。#4839
- 解决了BE故障可能导致数据丢失的问题（通过使用批量发布版本解决）。[#3140](https://github.com/StarRocks/starrocks/issues/3140)

## 2.0.3

发布日期：2022年3月14日

### 错误修复

修复了以下错误：

- 当BE节点处于假死状态时，查询失败。
- 当单表连接没有合适的执行计划时，查询失败。[#3854](https://github.com/StarRocks/starrocks/issues/3854)
- 当一个FE节点收集信息来构建全局字典以进行低基数优化时，可能会出现死锁问题。[#3839](https://github.com/StarRocks/starrocks/issues/3839)

## 2.0.2

发布日期：2022年3月2日

### 改进

- 优化了内存使用。用户可以通过设置label_keep_max_num参数来控制一段时间内保留的最大加载作业数量，以避免因FE频繁加载数据时内存占用过高而导致的全量垃圾回收（Full GC）。

### 错误修复

修复了以下错误：

- 当列解码器遇到异常时，BE节点失败。
- 加载JSON数据时，如果在命令中指定了jsonpaths，则自动__op映射不会生效。
- 使用Broker Load加载数据期间，如果源数据发生变化，则BE节点失败。
- 在创建物化视图后，某些SQL语句报错。
- 如果SQL子句包含既支持低基数优化全局字典的谓词，也包含不支持的谓词，则查询可能失败。

## 2.0.1

发布日期：2022年1月21日

### 改进

- 当StarRocks使用外部表查询[Hive数据](https://github.com/StarRocks/starrocks/pull/2829)时，可以读取Hive的implicit_cast操作。#2829
- 使用读/写锁来解决高CPU占用率的问题，当StarRocks CBO收集统计信息以支持高并发查询时。[#2901](https://github.com/StarRocks/starrocks/pull/2901)
- 优化了CBO统计数据收集和UNION运算符。

### 错误修复

- 修复了副本全局字典不一致导致的查询错误。[#2700](https://github.com/StarRocks/starrocks/pull/2700) [#2765](https://github.com/StarRocks/starrocks/pull/2765)
- 修复了数据加载时参数 `exec_mem_limit` 不生效的错误。[#2693](https://github.com/StarRocks/starrocks/pull/2693)
    > 参数exec_mem_limit指定了数据加载时每个BE节点的内存限制。
- 修复了向主键表导入数据时出现的OOM错误。[#2743](https://github.com/StarRocks/starrocks/pull/2743) [#2777](https://github.com/StarRocks/starrocks/pull/2777)
- 修复了StarRocks使用外部表查询大型MySQL表时BE节点停止响应的错误。[#2881](https://github.com/StarRocks/starrocks/pull/2881)

### 行为变更

StarRocks可以使用外部表访问Hive及其基于AWS S3的外部表。然而，用于访问S3数据的jar文件体积较大，StarRocks的二进制包中不包含该jar文件。如果您想使用这个jar文件，可以从[Hive_s3_lib](https://releases.starrocks.io/resources/hive_s3_jar.tar.gz)下载。

## 2.0.0

发布日期：2022年1月5日

### 新特性

- 外部表
  - [实验功能]支持S3上的Hive外部表
  - 为外部表支持[DecimalV3](https://github.com/StarRocks/starrocks/pull/425) #425
- 实现复杂表达式下推到存储层进行计算，以提升性能
- 正式发布Primary Key功能，支持Stream Load、Broker Load、Routine Load，并提供基于Flink-cdc的MySQL数据秒级同步工具

### 改进

- 算术运算符优化
  - 优化了低基数字典的性能 [#791](https://github.com/StarRocks/starrocks/pull/791)
  - 优化了单表`int`类型的扫描性能 [#273](https://github.com/StarRocks/starrocks/issues/273)
  - 优化了高基数`count(distinct int)`的性能 [#139](https://github.com/StarRocks/starrocks/pull/139) [#250](https://github.com/StarRocks/starrocks/pull/250)  [#544](https://github.com/StarRocks/starrocks/pull/544)[#570](https://github.com/StarRocks/starrocks/pull/570)
  - 在实现层面优化了Group by int / limit / case when / not equal等操作
- 内存管理优化
  - 重构了内存统计和控制框架，精确统计内存使用，彻底解决了OOM问题
  - 优化了元数据内存使用
  - 解决了执行线程中大量内存释放长时间卡住的问题
  - 添加了进程优雅退出机制，并支持内存泄露检查 [#1093](https://github.com/StarRocks/starrocks/pull/1093)

### 错误修复

- 修复了Hive外部表在获取大量元数据时超时的问题。
- 修复了物化视图创建时错误提示不清晰的问题。
- 修复了矢量化引擎中like实现的问题 [#722](https://github.com/StarRocks/starrocks/pull/722)
- 修复了`alter table`中谓词解析错误的问题[#725](https://github.com/StarRocks/starrocks/pull/725)
- 修复了curdate函数无法正确格式化日期的问题。
