---
displayed_sidebar: English
---

# StarRocks 2.1 版本

## 2.1.13

发布日期：2022年9月6日

### 改进

- 新增了一个BE配置项 `enable_check_string_lengths` 用于检查加载数据的长度。该机制有助于防止因VARCHAR数据大小超出范围而导致的压缩失败。[#10380](https://github.com/StarRocks/starrocks/issues/10380)
- 优化了查询性能，当一个查询包含超过1000个OR运算符时。[#9332](https://github.com/StarRocks/starrocks/pull/9332)

### Bug 修复

修复了以下问题：

- 当您从一个表中使用Aggregate表查询ARRAY列（通过REPLACE_IF_NOT_NULL函数计算）时，可能会出现错误，并且BE可能会崩溃。[#10144](https://github.com/StarRocks/starrocks/issues/10144)
- 如果查询中嵌套了多个IFNULL()函数，则查询结果不正确。[#5028](https://github.com/StarRocks/starrocks/issues/5028) [#10486](https://github.com/StarRocks/starrocks/pull/10486)
- 在动态分区被截断后，分区中的Tablet数量会从动态分区配置的值变为默认值。[#10435](https://github.com/StarRocks/starrocks/issues/10435)
- 当您使用Routine Load将数据加载到StarRocks时，如果Kafka集群停止，可能会发生死锁，从而影响查询性能。[#8947](https://github.com/StarRocks/starrocks/issues/8947)
- 当一个查询同时包含子查询和ORDER BY子句时，会出现错误。[#10066](https://github.com/StarRocks/starrocks/pull/10066)

## 2.1.12

发布日期：2022年8月9日

### 改进

新增了两个参数 `bdbje_cleaner_threads` 和 `bdbje_replay_cost_percent`，以加快BDB JE中的元数据清理速度。[#8371](https://github.com/StarRocks/starrocks/pull/8371)

### Bug 修复

修复了以下问题：

- 一些查询被转发到Leader FE，导致`/api/query_detail`操作返回有关SQL语句（如SHOW FRONTENDS）的不正确执行信息。[#9185](https://github.com/StarRocks/starrocks/issues/9185)
- 当一个BE终止后，当前进程未完全终止，导致BE无法成功重启。[#9175](https://github.com/StarRocks/starrocks/pull/9267)
- 当创建多个Broker Load作业来加载相同的HDFS数据文件时，如果一个作业遇到异常，其他作业可能无法正确读取数据，从而导致失败。[#9506](https://github.com/StarRocks/starrocks/issues/9506)
- 当一个表的结构发生变化时，相关变量未重置，导致在查询表时出现错误（`no delete vector found tablet`）。[#9192](https://github.com/StarRocks/starrocks/issues/9192)

## 2.1.11

发布日期：2022年7月9日

### Bug 修复

修复了以下问题：

- 当数据频繁加载到主键表的表中时，可能会导致数据加载暂停。[#7763](https://github.com/StarRocks/starrocks/issues/7763)
- 在低基数优化期间，聚合表达式的处理顺序不正确，导致`count distinct`函数返回意外结果。[#7659](https://github.com/StarRocks/starrocks/issues/7659)
- LIMIT子句不会返回任何结果，因为无法正确处理子句中的修剪规则。[#7894](https://github.com/StarRocks/starrocks/pull/7894)
- 如果对定义为查询联接条件的列应用低基数优化的全局字典，则查询将返回意外结果。[#8302](https://github.com/StarRocks/starrocks/issues/8302)

## 2.1.10

发布日期：2022年6月24日

### Bug 修复

修复了以下问题：

- 重复切换Leader FE节点可能会导致所有加载作业挂起并失败。[#7350](https://github.com/StarRocks/starrocks/issues/7350)
- 使用SQL检查表模式时，DECIMAL（18,2）类型的字段显示为DECIMAL64（18,2）。[#7309](https://github.com/StarRocks/starrocks/pull/7309)
- 当MemTable的内存使用估计超过4GB时，由于在负载数据倾斜期间，某些字段可能会占用大量内存资源，BE会崩溃。[#7161](https://github.com/StarRocks/starrocks/issues/7161)
- 当压缩中有许多输入行时，由于max_rows_per_segment计算中的溢出，会创建大量小段文件。[#5610](https://github.com/StarRocks/starrocks/issues/5610)

## 2.1.8

发布日期：2022年6月9日

### 改进

- 优化了用于内部处理工作负载（如Schema更改）的并发控制机制，以减轻前端（FE）元数据的压力。因此，如果同时运行这些加载作业以加载大量数据，则加载作业不太可能堆积和减慢速度。[#6560](https://github.com/StarRocks/starrocks/pull/6560) [#6804](https://github.com/StarRocks/starrocks/pull/6804)
- 提升了StarRocks在高频加载数据时的性能。#6532 [ ](https://github.com/StarRocks/starrocks/pull/6532)#6533[ ](https://github.com/StarRocks/starrocks/pull/6533)

### Bug 修复

修复了以下问题：

- ALTER操作日志未记录有关LOAD语句的所有信息。因此，在对例行加载作业执行ALTER操作后，创建检查点后，作业的元数据将丢失。[#6936](https://github.com/StarRocks/starrocks/issues/6936)
- 如果停止例行加载作业，则可能会发生死锁。[#6450](https://github.com/StarRocks/starrocks/issues/6450)
- 默认情况下，后端（BE）使用默认的UTC+8时区进行加载作业。如果服务器使用UTC时区，则使用Spark加载作业加载的表的DateTime列中的时间戳将添加8小时。[#6592](https://github.com/StarRocks/starrocks/issues/6592)
- GET_JSON_STRING函数无法处理非JSON字符串。如果从JSON对象或数组中提取JSON值，则该函数将返回NULL。该函数已经过优化，可为JSON对象或数组返回等效的JSON格式的STRING值。[#6426](https://github.com/StarRocks/starrocks/issues/6426)
- 如果数据量很大，则架构更改可能会因内存消耗过多而失败。已进行优化，允许您在架构更改的所有阶段指定内存消耗限制。[#6705](https://github.com/StarRocks/starrocks/pull/6705)
- 如果正在压缩的表的列中的重复值数超过0x40000000，则将暂停压缩。[#6513](https://github.com/StarRocks/starrocks/issues/6513)
- 在FE重启后，由于BDB JE v7.3.8中的一些问题，会遇到I/O过高和磁盘使用率异常增加的问题，并且没有恢复正常的迹象。FE回滚到BDB JE v7.3.7后恢复正常。[#6634](https://github.com/StarRocks/starrocks/issues/6634)

## 2.1.7

发布日期：2022年5月26日

### 改进

对于窗口函数中的框架设置为ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW的情况，如果涉及计算的分区较大，StarRocks会在执行计算之前缓存该分区的所有数据。在这种情况下，会消耗大量的内存资源。StarRocks已经优化，不再在这种情况下缓存分区的所有数据。[5829](https://github.com/StarRocks/starrocks/issues/5829)

### Bug 修复

修复了以下问题：

- 当数据加载到使用主键表的表中时，如果由于系统时间倒移等原因导致系统中存储的各数据版本的创建时间没有单调增加，可能会出现数据处理错误。此类数据处理错误会导致后端（BE）停止。[#6046](https://github.com/StarRocks/starrocks/issues/6046)
- 一些图形用户界面（GUI）工具会自动配置set_sql_limit变量。因此，将忽略SQL语句ORDER BY LIMIT，导致查询返回的行数不正确。[#5966](https://github.com/StarRocks/starrocks/issues/5966)
- 如果对数据库执行DROP SCHEMA语句，则该数据库将被强制删除，无法恢复。[#6201](https://github.com/StarRocks/starrocks/issues/6201)
- 当加载JSON格式的数据时，如果数据包含JSON格式错误，BE将停止。例如，键值对未用逗号（，）分隔。[#6098](https://github.com/StarRocks/starrocks/issues/6098)

- 当大量数据以高并发方式加载时，写入数据到磁盘的任务会堆积在BE上。在这种情况下，BE可能会停止。[#3877](https://github.com/StarRocks/starrocks/issues/3877)
- StarRocks在对表进行Schema更改之前会预估所需的内存量。如果表中包含大量STRING字段，则内存预估结果可能不准确。在这种情况下，如果预估所需的内存量超过单个Schema更改操作允许的最大内存，本应正确运行的Schema更改操作会遇到错误。[#6322](https://github.com/StarRocks/starrocks/issues/6322)
- 在对使用主键表的表执行Schema更改后，当数据加载到该表中时，可能会出现“重复键xxx”错误。[#5878](https://github.com/StarRocks/starrocks/issues/5878)
- 如果在Shuffle Join操作期间执行低基数优化，可能会出现分区错误。[#4890](https://github.com/StarRocks/starrocks/issues/4890)
- 如果colocation group（CG）包含大量表，并且数据频繁加载到表中，则CG可能无法保持稳定状态。在这种情况下，JOIN语句不支持Colocate Join操作。StarRocks已经优化，在数据加载过程中等待的时间会稍长一些。这样，可以最大限度地提高加载数据的平板电脑副本的完整性。

## 2.1.6

发布日期：2022年5月10日

### Bug修复

以下错误已修复：

- 在执行多个DELETE操作后运行查询时，如果对查询执行低基数列的优化，可能会获得不正确的查询结果。[#5712](https://github.com/StarRocks/starrocks/issues/5712)
- 如果平板电脑在特定的数据引入阶段进行迁移，数据将继续写入存储平板电脑的原始磁盘。因此，数据会丢失，查询无法正常运行。[#5160](https://github.com/StarRocks/starrocks/issues/5160)
- 如果在DECIMAL和STRING数据类型之间转换值，则返回值的精度可能出乎意料。[#5608](https://github.com/StarRocks/starrocks/issues/5608)
- 如果将DECIMAL值乘以BIGINT值，则可能会发生算术溢出。为了修复此错误，我们进行了一些调整和优化。[#4211](https://github.com/StarRocks/starrocks/pull/4211)

## 2.1.5

发布日期：2022年4月27日

### Bug修复

以下错误已修复：

- 当十进制乘法溢出时，计算结果不正确。修复bug后，当十进制乘法溢出时返回NULL。
- 当统计信息与实际统计信息有相当大的偏差时，并置联接的优先级可能低于广播联接。因此，查询规划器可能不会选择“共置联接”作为更合适的联接策略。[#4817](https://github.com/StarRocks/starrocks/pull/4817)
- 查询失败，因为当要联接的表超过4个时，复杂表达式的计划是错误的。
- 当随机排列列是低基数列时，BE可能会在随机联接下停止工作。[#4890](https://github.com/StarRocks/starrocks/issues/4890)
- 当SPLIT函数使用NULL参数时，BE可能会停止工作。[#4092](https://github.com/StarRocks/starrocks/issues/4092)

## 2.1.4

发布日期：2022年4月8日

### 新功能

- 支持`UUID_NUMERIC`函数，该函数返回LARGEINT值。与`UUID`函数相比，`UUID_NUMERIC`函数的性能可以提高近2个数量级。

### Bug修复

以下错误已修复：

- 删除列、添加新分区和克隆平板电脑后，新旧平板电脑中列的唯一ID可能不同，这可能会导致BE停止工作，因为系统使用共享平板电脑架构。[#4514](https://github.com/StarRocks/starrocks/issues/4514)
- 当数据加载到StarRocks外部表时，如果目标StarRocks集群配置的FE不是Leader，会导致FE停止工作。[#4573](https://github.com/StarRocks/starrocks/issues/4573)
- `CAST`函数在StarRocks 1.19和2.1版本的结果不同。[#4701](https://github.com/StarRocks/starrocks/pull/4701)
- 当Duplicate Key表同时执行架构更改并创建具体化视图时，查询结果可能不正确。[#4839](https://github.com/StarRocks/starrocks/issues/4839)

## 2.1.3

发布日期：2022年3月19日

### Bug修复

以下错误已修复：

- 由于BE失败可能导致数据丢失的问题（使用批量发布版本解决）。[#3140](https://github.com/StarRocks/starrocks/issues/3140)
- 某些查询可能会因不当的执行计划导致内存限制超出错误。
- 在不同的压缩过程中，副本之间的校验和可能不一致。[#3438](https://github.com/StarRocks/starrocks/issues/3438)
- 在某些情况下，当未正确处理JSON重新排序投影时，查询可能会失败。[#4056](https://github.com/StarRocks/starrocks/pull/4056)

## 2.1.2

发布日期：2022年3月14日

### Bug修复

以下错误已修复：

- 在从版本1.19到2.1的滚动升级中，由于两个版本之间的块大小不匹配，BE节点停止工作。[#3834](https://github.com/StarRocks/starrocks/issues/3834)
- StarRocks从2.0版本升级到2.1版本时，加载任务可能会失败。[#3828](https://github.com/StarRocks/starrocks/issues/3828)
- 当单平板电脑表联接没有适当的执行计划时，查询将失败。[#3854](https://github.com/StarRocks/starrocks/issues/3854)
- 当FE节点收集信息构建全局字典进行低基数优化时，可能会出现死锁问题。[#3839](https://github.com/StarRocks/starrocks/issues/3839)
- 当BE节点由于死锁而处于挂起动画状态时，查询失败。
- 当SHOW VARIABLES命令失败时，BI工具无法连接到StarRocks。[#3708](https://github.com/StarRocks/starrocks/issues/3708)

## 2.1.0

发布日期：2022年2月24日

### 新功能

- [预览] StarRocks现已支持Iceberg外部表。
- [预览] 管道引擎现已推出。它是专为多核调度而设计的新执行引擎。无需设置parallel_fragment_exec_instance_num参数即可自适应调整查询并行度。这也提高了高并发方案中的性能。
- 支持CTAS（CREATE TABLE AS SELECT）语句，使ETL和表的创建更加容易。
- 支持SQL指纹。SQL指纹在audit.log中生成，方便定位慢查询。
- 支持ANY_VALUE、ARRAY_REMOVE和SHA2函数。

### 改进

- 优化了压实度。平面表最多可以包含10,000列。
- 优化了首次扫描和页面缓存的性能。减少了随机I/O以提高首次扫描性能。如果首次扫描发生在SATA磁盘上，则改进会更加明显。StarRocks的页面缓存可以存储原始数据，无需进行bitshuffle编码和不必要的解码。这样可以提高缓存命中率和查询效率。
- 主键表中支持架构更改。可以使用添加、删除和修改位图索引`Alter table`。
- [预览] 字符串的大小最大为1MB。
- 优化了JSON加载性能。您可以在单个文件中加载超过100MB的JSON数据。
- 优化了位图索引性能。
- 优化了StarRocks Hive外表性能。可以读取CSV格式的数据。
- 语句中支持DEFAULT CURRENT_TIMESTAMP`create table`。
- StarRocks支持加载带有多个分隔符的CSV文件。

### Bug修复

以下错误已修复：

- 如果在用于加载JSON数据的命令中指定了jsonpaths，则自动__op映射不会生效。
- BE节点失败，因为在使用Broker Load加载数据期间源数据发生了变化。
- 某些SQL语句在创建物化视图后报错。
- 由于引用了jsonpaths，例程加载不起作用。
- 当查询的列数超过200时，查询并发性急剧下降。

### 行为更改
禁用主机托管组的 API 已从 `DELETE /api/colocate/group_stable` 更改为 `POST /api/colocate/group_unstable`。

### 其他

flink-connector-starrocks 现在可用于 Flink 批量读取 StarRocks 数据，相较于 JDBC 连接器，这提高了数据读取效率。