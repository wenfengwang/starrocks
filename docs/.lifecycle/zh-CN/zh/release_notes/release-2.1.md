---
displayed_sidebar: "中文"
---

# StarRocks 版本 2.1

## 2.1.13

Release Date: September 6, 2022

### 功能优化

- 增加 BE 配置项 `enable_check_string_lengths` 以控制导入时数据长度检查，以解决 VARCHAR 类型数据越界导致的 Compaction 失败问题。[#10380](https://github.com/StarRocks/starrocks/issues/10380)
- 优化了查询中包含1000个以上的 OR 子句时的查询性能。[#9332](https://github.com/StarRocks/starrocks/pull/9332)

### 问题修复

修复了以下问题：

- 在查询聚合模型表中 ARRAY 类型的列时，如该列使用 REPLACE_IF_NOT_NULL 聚合函数，可能导致查询报错，并导致 BE 服务停止。[#10144](https://github.com/StarRocks/starrocks/issues/10144)
- 查询中嵌套 1 个以上的 IFNULL 函数时，查询结果不正确。[#5028](https://github.com/StarRocks/starrocks/issues/5028) [#10486](https://github.com/StarRocks/starrocks/pull/10486)
- Truncate 动态创建的分区后，其分桶数会从动态分区设置的分桶数变成默认分桶数。[#10435](https://github.com/StarRocks/starrocks/issues/10435)
- 在使用 Routine Load 导入的过程中，如出现 Kafka 服务下线的情况，StarRocks 集群可能会出现暂时性死锁，影响查询。[#8947](https://github.com/StarRocks/starrocks/issues/8947)
- 查询语句中同时有子查询和 ORDER BY 子句时会报错。[#10066](https://github.com/StarRocks/starrocks/pull/10066)

## 2.1.12

发布日期：2022年8月9日

### 功能优化

新增 `bdbje_cleaner_threads` 和 `bdbje_replay_cost_percent` 两个参数，以加速清理 BDB JE 中的元数据。 [#8371](https://github.com/StarRocks/starrocks/pull/8371)

### 问题修复

修复了以下问题：

- 一些查询会被转发到 Leader FE 节点上，从而可能导致通过 `/api/query_detail` 接口获得的 SQL 语句执行信息不正确，例如 SHOW FRONTENDS 语句。[#9185](https://github.com/StarRocks/starrocks/issues/9185)
- 停止 BE 后，当前进程未完全退出，导致重启 BE 失败。[#9175](https://github.com/StarRocks/starrocks/pull/9267)
- 提交多个 Broker Load 作业同时导入相同 HDFS 文件的数据时，如果有一个作业出现异常，可能会导致其他作业也无法正常读取数据并且最终失败。[#9506](https://github.com/StarRocks/starrocks/issues/9506)
- 表结构变更后，相关变量未重置，导致查询该表报错：`no delete vector found tablet`。[#9192](https://github.com/StarRocks/starrocks/issues/9192)

## 2.1.11

发布日期：2022年7月9日

### 问题修复

修复了以下问题：

- 主键模型 (Primary Key) 表在高频导入时，会卡住无法继续进行。[#7763](https://github.com/StarRocks/starrocks/issues/7763)
- 在低基数优化中，对聚合表达式的顺序处理有误，导致 `count distinct` 函数返回的一些结果错误。[#7659](https://github.com/StarRocks/starrocks/issues/7659)
- LIMIT 子句中的裁剪规则处理错误，导致 LIMIT 子句执行以后没有结果。[#7894](https://github.com/StarRocks/starrocks/pull/7894)
- 如果一个查询的 Join 条件列中有进行全局低基数字典优化，会导致查询结果错误。[#8302](https://github.com/StarRocks/starrocks/issues/8302)

## 2.1.10

发布日期：2022年6月24日

### 问题修复

修复了以下问题：

- 反复切换 Leader FE 节点可能会导致所有导入作业挂起，无法进行导入。[#7350](https://github.com/StarRocks/starrocks/issues/7350)
- 使用 `DESC` 查看表结构时，类型为 DECIMAL(18,2) 的字段会展示为 DECIMAL64(18,2) 类型。[#7309](https://github.com/StarRocks/starrocks/pull/7309)
- 导入的数据有倾斜时，某些列占用内存比较大，可能会导致 MemTable 的内存估算超过 4GB，从而导致 BE 停止工作。[#7161](https://github.com/StarRocks/starrocks/issues/7161)
- 当 Compaction 的输入行数较多时，max_rows_per_segment 的内部估算会产生溢出，最终导致创建了大量的小 Segment 文件。[#5610](https://github.com/StarRocks/starrocks/issues/5610)

## 2.1.8

发布日期：2022年6月9日

### 功能优化

- 优化表结构变更 (Schema Change) 等内部处理的并发控制，降低对 FE 元数据的压力，最终减少在高并发、大数据量导入场景下容易发生的导入积压、变慢的情况。[#6560](https://github.com/StarRocks/starrocks/pull/6560) [#6804](https://github.com/StarRocks/starrocks/pull/6804)
- 优化高频导入的性能。[#6532](https://github.com/StarRocks/starrocks/pull/6532) [#6533](https://github.com/StarRocks/starrocks/pull/6533)

### 问题修复

修复了以下问题：

- 对 Routine Load 任务进行 ALTER 操作后，由于 ALTER 操作没有记录全量的 LOAD 语句信息，导致这个导入任务的元数据在做完 Checkpoint 后丢失。[#6936](https://github.com/StarRocks/starrocks/issues/6936)
- 停止 Routine Load 任务可能导致死锁。[#6450](https://github.com/StarRocks/starrocks/issues/6450)
- BE 导入数据时默认按 UTC+8 时区导入。如果用户机器时区为 UTC，那么用户使用 Spark Load 方式导入的数据的 DateTime 列会多加 8 个小时。 [#6592](https://github.com/StarRocks/starrocks/issues/6592)
- GET_JSON_STRING 函数无法处理非 JSON string 类型的值。如果要提取的值是 JSON 对象或 JSON 数组、而不是 JSON string 类型，该函数会直接返回 NULL。当前优化为返回 JSON 格式的 STRING 类型的数据。[#6426](https://github.com/StarRocks/starrocks/issues/6426)
- 如果数据量很大，做表结构变更 (Schema Change) 时可能因为内存消耗过多而失败。现允许限制表结构变更中各阶段的内存使用限额。[#6705](https://github.com/StarRocks/starrocks/pull/6705)
- 在进行 Compaction 时，如果某列的任意一个值重复出现的次数超过 0x40000000，会导致 Compaction 卡住。[#6513](https://github.com/StarRocks/starrocks/issues/6513)
- BDB JE v7.3.8 introduced some issues, causing high disk I/O and abnormal growth in disk usage after FE startup, and no signs of recovery, reverting to BDB JE v7.3.7 version restored the FE to normal. [#6634] (https://github.com/StarRocks/starrocks/issues/6634)

## 2.1.7

Release date: May 26, 2022

### Feature Optimization

For window functions with Frame set as `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`, when a partition is very large during calculation, the system caches all the data in the partition and then performs the calculation, resulting in excessive memory consumption. Now, in this scenario, the partition data will no longer be cached. [5829](https://github.com/StarRocks/starrocks/issues/5829)

### Problem Fix

The following issues have been fixed:

- When importing data into a primary key model table, if the system fails to maintain strict sequential creation times for each data version (e.g. due to system time being adjusted backwards or due to related unknown bugs), it will cause processing errors and lead to BE service interruption. [#6046](https://github.com/StarRocks/starrocks/issues/6046)

- Some graphical interface tools automatically set the `set_sql_limit` variable, causing the SQL statement ORDER BY LIMIT to be ignored, resulting in incorrect number of returned data rows. [#5966](https://github.com/StarRocks/starrocks/issues/5966)

- Executing the DROP SCHEMA statement will directly force delete the database, and the deleted database cannot be recovered. [#6201](https://github.com/StarRocks/starrocks/issues/6201)

- When importing data in JSON format, if the JSON format contains errors (such as missing commas "," between multiple key-value pairs), it will cause BE service interruption. [#6098](https://github.com/StarRocks/starrocks/issues/6098)

- In high-concurrency import scenarios, the number of tasks writing to the BE disk may accumulate, leading to BE service interruption. [#3877](https://github.com/StarRocks/starrocks/issues/3877)

- Before making table structural changes, the system will first estimate the memory. If there are many STRING type fields in this table, the memory estimation result will be inaccurate. In this case, if the estimated memory exceeds the memory limit allowed for a single table structure change operation, it will cause an error in the originally executable table structural change operation. [#6322](https://github.com/StarRocks/starrocks/issues/6322)

- After a primary key model table undergoes table structure changes, when importing data, it may report "duplicate key xxx" error. [#5878](https://github.com/StarRocks/starrocks/issues/5878)

- During Shuffle Join, if low cardinality optimization is used, it may cause partition errors. [#4890](https://github.com/StarRocks/starrocks/issues/4890)

- When a Colocation Group contains a large number of tables and has a high import frequency, it may cause the Colocation Group to fail to maintain a `stable` state, thereby causing JOIN statements to be unable to use Colocate Join. Now, the optimization is to wait a little longer when importing data, so as to ensure the integrity of the imported Tablet replicas as much as possible.

## 2.1.6

Release date: May 10, 2022

### Problem Fix

The following issues have been fixed:

- After performing multiple DELETE operations, if the system internally uses low cardinality optimization, the query result may be incorrect. [#5712](https://github.com/StarRocks/starrocks/issues/5712)

- During certain special stages of data writing, if a Tablet migrates and completes the migration, the data will continue to be written to the original Tablet's corresponding disk, leading to data loss and subsequent query errors. [#5160](https://github.com/StarRocks/starrocks/issues/5160)

- When converting between DECIMAL and STRING types, it may cause precision errors. [#5608](https://github.com/StarRocks/starrocks/issues/5608)

- When multiplying DECIMAL and BIGINT types, it may cause the arithmetic result to overflow. Some adjustments and optimizations have been made this time. [#4211](https://github.com/StarRocks/starrocks/pull/4211)

## 2.1.5

Release date: April 27, 2022

### Problem Fix

The following issues have been fixed:

- Previously, when Decimal multiplication overflowed, it caused a calculation error. After the fix, when Decimal multiplication overflows, it returns NULL.
- When the statistical information has a large error, the priority of Colocate Join in the execution plan may be lower than that of Broadcast Join, causing the Colocate Join to not take effect during actual execution. [#4817](https://github.com/StarRocks/starrocks/pull/4817)
- When Joining more than 4 tables, erroneous planning of complex expressions may cause query errors.
- Under Shuffle Join, if the Shuffle column is a low cardinality column, it may cause BE service interruption. [#4890](https://github.com/StarRocks/starrocks/issues/4890)
- When the SPLIT function uses NULL parameters, it may cause BE service interruption. [#4092](https://github.com/StarRocks/starrocks/issues/4092)

## 2.1.4

Release date: April 8, 2022

### New Function

- Added the `UUID_NUMERIC` function, which returns a value of type LARGEINT. Compared to the `UUID` function, the performance is improved by nearly 2 orders of magnitude.

### Problem Fix

The following issues have been fixed:

- After deleting columns, adding partitions, and cloning Tablets, the Unique ID of the new and old Tablets may not correspond. As the system uses a shared Tablet Schema, it may cause BE service interruption. [#4514](https://github.com/StarRocks/starrocks/issues/4514)
- When importing data into external tables of StarRocks, if the FE of the target StarRocks cluster is not the Leader, it will cause FE service interruption. [#4573](https://github.com/StarRocks/starrocks/issues/4573)
- The execution results of the `CAST` function in StarRocks version 1.19 and 2.1 are inconsistent. [#4701](https://github.com/StarRocks/starrocks/pull/4701)
- Tables of detail models simultaneously undergoing table structural changes and creating materialized views may cause data query errors. [#4839](https://github.com/StarRocks/starrocks/issues/4839)

## 2.1.3

Release date: March 19, 2022

### Problem Fix

- By improving to batch publish version, the issue of possible data loss due to BE crash has been resolved. [#3140](https://github.com/StarRocks/starrocks/issues/3140)
- Fixed some queries that may cause memory overrun due to unreasonable execution plans.
- Fixed the inconsistency of checksum (checksum) of fragment replicas under different compaction processes. [#3438](https://github.com/StarRocks/starrocks/issues/3438)
- Fixed the issue where JOIN reorder projection was not handled correctly, leading to possible query errors. [#4056](https://github.com/StarRocks/starrocks/pull/4056)

## 2.1.2
发布日期： 2022年3月14日

### 问题修复

- 修复从 1.19 升级到 2.1 会因 `chunk_size` 不匹配导致 BE 崩溃的问题。[#3834](https://github.com/StarRocks/starrocks/issues/3834)
- 修复在从 2.0 升级到 2.1 的过程中有导入时，可能导致导入任务失败的问题。[#3828](https://github.com/StarRocks/starrocks/issues/3828)
- 修复对单 tablet 的表在做聚合操作时因无法得到合理的执行计划而导致查询失败的问题。[#3854](https://github.com/StarRocks/starrocks/issues/3854)
- 修复 FE 在低基数全局字典优化中收集信息时可能导致死锁的问题。[#3839](https://github.com/StarRocks/starrocks/issues/3839)
- 修复因死锁导致 BE 节点假死且查询失败的问题。
- 修复因 SHOW VARIABLES 命令出错而导致 BI 工具无法连接的问题。[#3708](https://github.com/StarRocks/starrocks/issues/3708)

## 2.1.0

发布日期： 2022年2月24日

### 新功能

- 【公测中】支持通过外表的方式查询 Apache Iceberg 数据湖中的数据，帮助您实现对数据湖的极速分析。TPC-H 测试集的结果显示，查询 Apache Iceberg 数据时，StarRocks 的查询速度是 Presto 的 **3 - 5** 倍。相关文档，请参见 [Apache Iceberg 外表](../data_source/External_table.md#deprecated-hive-外部表)。
- 【公测中】发布 Pipeline 执行引擎，可以自适应调节查询的并行度。您无需手动设置 session 级别的变量 parallel_fragment_exec_instance_num。并且，在部分高并发场景中，相较于历史版本，新版本性能提升两倍。
- 支持 CTAS（CREATE TABLE AS SELECT），基于查询结果创建表并且导入数据，从而简化建表和 ETL 操作。相关文档，请参见 [CREATE TABLE AS SELECT](../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)。
- 支持 SQL 指纹，针对慢查询中各类 SQL 语句计算出 SQL 指纹，方便您快速定位慢查询。相关文档，请参见 [SQL 指纹](../administration/Query_planning.md#查看-sql-指纹)。
- 新增函数 [ANY_VALUE](../sql-reference/sql-functions/aggregate-functions/any_value.md)，[ARRAY_REMOVE](../sql-reference/sql-functions/array-functions/array_remove.md)，哈希函数 [SHA2](../sql-reference/sql-functions/crytographic-functions/sha2.md)。

### 功能优化

- 优化 Compaction 的性能，支持导入 10000 列的数据。
- 优化 StarRocks 首次 Scan 和 Page Cache 的性能。通过降低随机 I/O ，提升 StarRocks 首次 Scan 的性能，如果首次 Scan 的磁盘为 SATA 盘，则性能提升尤为明显。另外，StarRocks 的 Page Cache 支持直接存放原始数据，无需经过 Bitshuffle 编码。因此读取 StarRocks 的 Page Cache 时无需额外解码，提高缓存命中率，进而大大提升查询效率。
- 支持主键模型（Primary Key Model）变更表结构（Schema Change），您可以执行 `ALTER TABLE` 增删和修改索引。
- 优化 JSON 导入性能，并去除了 JSON 导入中单个 JSON 文件不超过 100MB 大小的限制。
- 优化 Bitmap Index 性能。
- 优化通过外表方式读取 Hive 数据的性能，支持 Hive 的存储格式为 CSV。
- 支持建表语句的时间戳字段定义为 DEFAULT CURRENT_TIMESTAMP。
- 支持导入带有多个分隔符的 CSV 文件。
- flink-source-connector 支持 Flink 批量读取 StarRocks 数据，实现了直连并行读取 BE 节点、自动谓词下推等特性。相关文档，请参见 [Flink Connector](../unloading/Flink_connector.md)。

### 问题修复

- 修复在导入 JSON 格式数据中设置了 jsonpaths 后不能自动识别 __op 字段的问题。
- 修复 Broker Load 导入数据过程中因为源数据发生变化而导致 BE 节点挂掉的问题。
- 修复建立物化视图后，部分 SQL 语句报错的问题。
- 修复 Routine Load 中使用带引号的 jsonpaths 会报错的问题。
- 修复查询列数超过 200 列后，并发性能明显下降的问题。

### 其他

修改关闭 Colocation Group 的 HTTP Restful API。为了使语义更好理解，关闭 Colocation Group 的 API 修改为 `POST /api/colocate/group_unstable`（旧接口为 `DELETE /api/colocate/group_stable` ）。

> 如果需要重新开启 Colocation Group ，则可以使用 API `POST /api/colocate/group_stable`。