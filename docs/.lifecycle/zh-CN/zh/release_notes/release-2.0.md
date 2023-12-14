---
displayed_sidebar: "Chinese"
---

# StarRocks 版本 2.0

## 2.0.9

发布日期：2022年8月6日

### 问题修复

修复了如下问题：

- 使用 Broker Load 导入数据时，如果 Broker 进程压力大，可能导致内部心跳处理时出现问题，从而导致导入数据丢失。[#8282](https://github.com/StarRocks/starrocks/issues/8282)
- 使用 Broker Load 导入数据时，如果使用 `COLUMNS FROM PATH AS` 参数指定的列在 StarRocks 的表中不存在，会导致 BE 停止服务。[#5346](https://github.com/StarRocks/starrocks/issues/5346)
- 一些查询会被转发到 Leader FE 节点上，从而可能导致通过 `/api/query_detail` 接口获得的 SQL 语句执行信息不正确，比如 SHOW FRONTENDS 语句。[#9185](https://github.com/StarRocks/starrocks/issues/9185)
- 提交多个 Broker Load 作业同时导入相同 HDFS 文件的数据时，如果有一个作业出现异常，可能会导致其他作业也无法正常读取数据并且最终失败。[#9506](https://github.com/StarRocks/starrocks/issues/9506)

## 2.0.8

发布日期：2022年7月15日

### 问题修复

修复了如下问题：

- 反复切换 Leader FE 节点可能会导致所有导入作业挂起，无法进行导入。[#7350](https://github.com/StarRocks/starrocks/issues/7350)
- 导入的数据有倾斜时，某些列占用内存比较大，可能会导致 MemTable 的内存估算超过 4GB，从而导致 BE 停止工作。[#7161](https://github.com/StarRocks/starrocks/issues/7161)
- 物化视图处理大小写时有问题，导致重启 FE 后其 schema 发生变化。[#7362](https://github.com/StarRocks/starrocks/issues/7362)
- 使用 Routine Load 导入 Kafka 的 JSON 数据时，如果 JSON 数据中存在空行，那么空行后的数据会丢失。[#8534](https://github.com/StarRocks/starrocks/issues/8534)

## 2.0.7

发布日期：2022年6月13日

### 问题修复

修复了如下问题：

- 在进行表压缩 (Compaction) 时，如果某列的任意一个值重复出现的次数超过 0x40000000，会导致 Compaction 卡住。[#6513](https://github.com/StarRocks/starrocks/issues/6513)
- BDB JE v7.3.8 版本引入了一些问题，导致 FE 启动后磁盘 I\/O 很高、磁盘使用率持续异常增长、且没有恢复迹象，回退到 BDB JE v7.3.7 版本后 FE 恢复正常。[#6634](https://github.com/StarRocks/starrocks/issues/6634)

## 2.0.6

发布日期：2022年5月25日

### 问题修复

修复了如下问题：

- 某些图形化界面工具会自动设置 `set_sql_limit` 变量，导致 SQL 语句 ORDER BY LIMIT 被忽略，从而导致返回的数据行数不正确。[#5966](https://github.com/StarRocks/starrocks/issues/5966)
- 当一个 Colocation Group 中包含的表比较多、导入频率又比较高时，可能会导致该 Colocation Group 无法保持 `stable` 状态，从而导致 JOIN 语句无法使用 Colocate Join。现优化为导入数据时稍微多等一会，这样可以尽量保证导入的 Tablet 副本的完整性。
- 少数副本由于负载较高、网络延迟等原因导致导入失败，系统会触发副本克隆操作。在这种情况下，会有一定概率引发死锁，从而可能出现进程负载极低、却有大量请求超时的现象。[#5646](https://github.com/StarRocks/starrocks/issues/5646) [#6290](https://github.com/StarRocks/starrocks/issues/6290)
- 主键模型的表经过表结构变更以后，在数据导入时，可能会报 "duplicate key xxx" 错误。[#5878](https://github.com/StarRocks/starrocks/issues/5878)
- 执行 DROP SCHEMA 语句，会导致直接强制删除数据库，并且删除的数据库不可恢复。[#6201](https://github.com/StarRocks/starrocks/issues/6201)

## 2.0.5

发布日期：2022年5月13日

升级建议：本次修复了一些跟数据存储或数据查询正确性相关的关键 Bug，建议您及时升级。

### 问题修复

修复了如下问题：

- 【Critical Bug】通过改进为批量 publish version，解决 BE 可能因宕机而导致数据丢失的问题。[#3140](https://github.com/StarRocks/starrocks/issues/3140)
- 【Critical Bug】在数据写入中的一些特殊阶段，如果 Tablet 进行并完成迁移，数据会继续写入至原先 Tablet 对应的磁盘，导致数据丢失，进而导致查询错误。[#5160](https://github.com/StarRocks/starrocks/issues/5160)
- 【Critical Bug】在进行多个 DELETE 操作后，查询时，如果系统内部使用了低基数优化，则查询结果可能是错误的。[#5712](https://github.com/StarRocks/starrocks/issues/5712)
- 【Critical Bug】JOIN 查询的两个字段类型分别是 DOUBLE 和 VARCHAR 时，JOIN 查询结果可能错误。 [#5809](https://github.com/StarRocks/starrocks/pull/5809)
- 在数据导入中的某些特殊情形，可能一些副本的某些版本还未生效，却被 FE 标记为生效，导致查询时出现找不到对应版本数据的错误。[#5153](https://github.com/StarRocks/starrocks/issues/5153)
- `SPLIT` 函数使用 `NULL` 参数时，会导致 BE 停止服务。[#4092](https://github.com/StarRocks/starrocks/issues/4092)  
- 从 Apache Doris 0.13 升级到 StarRocks 1.19.x 并运行一段时间，再升级到 StarRocks 2.0.1，可能会升级失败。[#5309](https://github.com/StarRocks/starrocks/issues/5309)

## 2.0.4

发布日期： 2022年4月18日

### 问题修复

修复了如下问题：

- 在删列、新增分区、并克隆 Tablet 后，新旧 Tablet 的列 Unique ID 可能会不对应，由于系统使用共享的 Tablet Schema，可能导致 BE 停止服务。[#4514](https://github.com/StarRocks/starrocks/issues/4514)
- 向 StarRocks 外表导入数据时，如果设定的目标 StarRocks 集群的 FE 不是 Leader，则会导致 FE 停止服务。[#4573](https://github.com/StarRocks/starrocks/issues/4573)
- 明细模型的表同时执行表结构变更、创建物化视图时，可能导致数据查询错误。[#4839](https://github.com/StarRocks/starrocks/issues/4839)
- Improve the problem of possible data loss in BE due to downtime by optimizing for batch publish version. [#3140](https://github.com/StarRocks/starrocks/issues/3140)

## 2.0.3

Release Date: 14th March 2022

### Issue Fixes

- Fix the issue of BE hanging leading to query errors.
- Fix the issue of query failure for aggregation operations on a single tablet table due to inability to obtain a reasonable execution plan. [#3854](https://github.com/StarRocks/starrocks/issues/3854)
- Fix the potential deadlock issue when FE collects information to build low-cardinality global dictionaries. [#3839](https://github.com/StarRocks/starrocks/issues/3839)

## 2.0.2

Release Date: 2nd March 2022

### Feature Optimization

- Optimize FE memory usage. Control the maximum number of import tasks retained within a certain period of time by setting the parameter `label_keep_max_num` to avoid excessive FE memory usage and subsequent Full GC during high-frequency import tasks.

### Issue Fixes

- Fix the issue of BE node becoming unresponsive due to ColumnDecoder exceptions.
- Fix the automatic recognition issue of the `__op` field in imported JSON format data with jsonpaths set.
- Fix the issue of BE node becoming unresponsive due to source data changes during Broker Load data import process.
- Fix the issue of certain SQL statements reporting errors after materialized view creation.
- Fix the query failure issue when unsupported predicates are present in queries with low-cardinality global dictionaries.

## 2.0.1

Release Date: 21st January 2022

### Feature Optimization

- Optimize StarRocks' implicit data conversion functionality when reading Hive external tables. [#2829](https://github.com/StarRocks/starrocks/pull/2829)
- Optimize the lock contention issue during statistics collection by StarRocks CBO optimizer in high-concurrency query scenarios. [#2901](https://github.com/StarRocks/starrocks/pull/2901)
- Optimize CBO statistics work, UNION operator, and more.

### Issue Fixes

- Fix the query issue caused by inconsistent global dictionaries of replicas. [#2700](https://github.com/StarRocks/starrocks/pull/2700) [#2765](https://github.com/StarRocks/starrocks/pull/2765)
- Fix the ineffectiveness issue of setting the parameter `exec_mem_limit` before importing data into StarRocks. [#2693](https://github.com/StarRocks/starrocks/pull/2693)
  > The parameter `exec_mem_limit` is used to specify the memory limit for calculations on a single BE node during data import.
- Fix the OOM issue triggered when importing data into StarRocks primary key model. [#2743](https://github.com/StarRocks/starrocks/pull/2743) [#2777](https://github.com/StarRocks/starrocks/pull/2777)
- Fix the query hang issue when querying large-scale MySQL external tables with StarRocks. [#2881](https://github.com/StarRocks/starrocks/pull/2881)

### Behavior Change

- StarRocks now supports accessing Amazon S3 external tables created on Hive external tables. Due to the large jar package used to access Amazon S3 external tables, the StarRocks binary product package currently does not include this jar package. If needed, please click [Hive_s3_lib](https://releases.mirrorship.cn/resources/hive_s3_jar.tar.gz) to download.

## 2.0.0

Release Date: 5th January 2022

### New Features

- External Tables
  - [Experimental Feature] Support for Hive external tables on S3 [Reference Document](../data_source/External_table.md#deprecated-hive-external-table)
  - DecimalV3 supports external table queries [#425](https://github.com/StarRocks/starrocks/pull/425)
- Implemented storage layer complex expression pushdown calculation for performance improvement
- Broker Load supports Huawei OBS [#1182](https://github.com/StarRocks/starrocks/pull/1182)
- Support for national encryption standard SM3
- Adaptation for ARM-based domestic CPUs: verified on Kunpeng architecture
- Official release of the primary key model, which supports Stream Load, Broker Load, Routine Load, and provides a second-level synchronization tool for MySQL data based on Flink-cdc. [Reference Document](../table_design/table_types/primary_key_table.md)

### Feature Optimization

- Operator performance optimization
  - Low-cardinality dictionary performance optimization [#791](https://github.com/StarRocks/starrocks/pull/791)
  - Performance optimization for `int scan` on a single table [#273](https://github.com/StarRocks/starrocks/issues/273)
  - Performance optimization for `count(distinct int)` under high-cardinality scenarios [#139](https://github.com/StarRocks/starrocks/pull/139) [#250](https://github.com/StarRocks/starrocks/pull/250) [#544](https://github.com/StarRocks/starrocks/pull/544) [#570](https://github.com/StarRocks/starrocks/pull/570)
  - Execution layer optimization and improvement for `Group by 2 int` / `limit` / `case when` / `not equal`
- Memory management optimization
  - Refactored memory statistics/control framework for accurate memory usage statistics, completely solving OOM issues
  - Optimized metadata memory usage
  - Resolved the long-term thread blocking issue when releasing large memory
  - Process graceful exit mechanism, supports memory leak detection [#1093](https://github.com/StarRocks/starrocks/pull/1093)

### Issue Fixes

- Fix the issue of excessive metadata fetching timeouts for Hive external tables
- Fix the unclear error message issue when creating materialized views
- Fix the implementation of `like` in vectorized engine [#722](https://github.com/StarRocks/starrocks/pull/722)
- Fix parsing error of predicate `is` in `alter table` [#725](https://github.com/StarRocks/starrocks/pull/725)
- Fix the inability of the `curdate` function to format dates