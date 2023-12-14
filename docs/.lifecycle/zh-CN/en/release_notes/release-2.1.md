---
displayed_sidebar: "English"
---

# StarRocks版本2.1

## 2.1.13

发布日期：2022年9月6日

### 改进

- 添加了BE配置项`enable_check_string_lengths`以检查加载数据的长度。此机制有助于防止VARCHAR数据大小超出范围导致压缩失败。[#10380](https://github.com/StarRocks/starrocks/issues/10380)
- 优化了查询性能，当查询包含超过1000个OR运算符时。[#9332](https://github.com/StarRocks/starrocks/pull/9332)

### Bug修复

已修复以下错误：

- 当从使用Aggregate表的表中查询ARRAY列（通过使用REPLACE_IF_NOT_NULL函数进行计算）时，可能会出现错误并导致BE崩溃。[#10144](https://github.com/StarRocks/starrocks/issues/10144)
- 如果查询中嵌套了多个IFNULL()函数，则查询结果不正确。[#5028](https://github.com/StarRocks/starrocks/issues/5028) [#10486](https://github.com/StarRocks/starrocks/pull/10486)
- 在动态分区被截断后，分区中的Tablet数量从动态分区配置的值更改为默认值。[#10435](https://github.com/StarRocks/starrocks/issues/10435)
- 如果在使用Routine Load将数据加载到StarRocks时，Kafka集群停止，则可能会发生死锁，影响查询性能。[#8947](https://github.com/StarRocks/starrocks/issues/8947)
- 当查询同时包含子查询和ORDER BY子句时，将会出现错误。[#10066](https://github.com/StarRocks/starrocks/pull/10066)

## 2.1.12

发布日期：2022年8月9日

### 改进

添加了两个参数`bdbje_cleaner_threads`和`bdbje_replay_cost_percent`，以加快BDB JE中的元数据清理。[#8371](https://github.com/StarRocks/starrocks/pull/8371)

### Bug修复

已修复以下错误：

- 一些查询被转发到Leader FE，导致`/api/query_detail`操作返回关于SQL语句执行信息的错误。[#9185](https://github.com/StarRocks/starrocks/issues/9185)
- 在BE终止后，当前进程未完全终止，导致BE无法成功重启。[#9175](https://github.com/StarRocks/starrocks/pull/9267)
- 当创建多个Broker Load作业以加载相同的HDFS数据文件时，如果一个作业遇到异常，则其他作业可能无法正确读取数据，因此失败。[#9506](https://github.com/StarRocks/starrocks/issues/9506)
- 当表的模式更改时，相关变量未被重置，导致在查询表时出现错误（`no delete vector found tablet`）。[#9192](https://github.com/StarRocks/starrocks/issues/9192)

## 2.1.11

发布日期：2022年7月9日

### Bug修复

已修复以下错误：

- 在频繁向Primary Key表加载数据时，会暂停加载事件。[#7763](https://github.com/StarRocks/starrocks/issues/7763)
- 在低基数优化期间处理聚合表达式的顺序不正确，导致`count distinct`函数返回意外结果。[#7659](https://github.com/StarRocks/starrocks/issues/7659)
- 由于无法正确处理LIMIT子句中的修剪规则，因此不会返回任何结果。[#7894](https://github.com/StarRocks/starrocks/pull/7894)
- 如果在查询的连接条件中应用了基数优化全局字典，则会返回意外结果。[#8302](https://github.com/StarRocks/starrocks/issues/8302)

## 2.1.10

发布日期：2022年6月24日

### Bug修复

已修复以下错误：

- 反复切换Leader FE节点可能导致所有加载作业挂起和失败。[#7350](https://github.com/StarRocks/starrocks/issues/7350)
- 使用`DESC` SQL检查表模式时，DECIMAL(18,2)类型的字段显示为DECIMAL64(18,2)。[#7309](https://github.com/StarRocks/starrocks/pull/7309)
- 当MemTable的内存使用量估计超过4GB时，由于在加载过程中发生数据倾斜，某些字段可能占用大量内存资源，导致BE崩溃。[#7161](https://github.com/StarRocks/starrocks/issues/7161)
- 当压缩过程中存在大量输入行时，由于最大行数每个段的计算溢出，将创建大量小段文件。[#5610](https://github.com/StarRocks/starrocks/issues/5610)

## 2.1.8

发布日期：2022年6月9日

### 改进

- 优化了用于内部处理工作负载（如模式更改）的并发控制机制，以减轻前端（FE）元数据的压力。因此，如果这些加载作业同时运行以加载大量数据，则加载作业不太可能堆积并减慢速度。[#6560](https://github.com/StarRocks/starrocks/pull/6560) [#6804](https://github.com/StarRocks/starrocks/pull/6804)
- 改进了StarRocks在高频率加载数据时的性能。[#6532](https://github.com/StarRocks/starrocks/pull/6532) [#6533](https://github.com/StarRocks/starrocks/pull/6533)

### Bug修复

已修复以下错误：

- ALTER操作日志未记录有关LOAD语句的所有信息。因此，在对常规加载作业执行ALTER操作后，检查点创建之后作业的元数据将丢失。[#6936](https://github.com/StarRocks/starrocks/issues/6936)
- 如果停止常规加载作业可能会导致死锁。[#6450](https://github.com/StarRocks/starrocks/issues/6450)
- 默认情况下，后端（BE）对于加载作业使用默认的UTC+8时区。如果服务器使用UTC时区，则会在通过Spark加载作业加载的表的DateTime列中添加8小时的时间戳。[#6592](https://github.com/StarRocks/starrocks/issues/6592)
- GET_JSON_STRING函数无法处理非JSON字符串。如果从JSON对象或数组中提取JSON值，则该函数将返回NULL。该函数已经优化，以便为JSON对象或数组返回等效的JSON格式字符串值。[#6426](https://github.com/StarRocks/starrocks/issues/6426)
- 如果数据量较大，则由于内存消耗过多可能导致模式更改失败。已进行优化，使您能够在模式更改的所有阶段指定内存消耗限制。[#6705](https://github.com/StarRocks/starrocks/pull/6705)
- 如果表的列中的重复值数量超过0x40000000，则会暂停压缩。[#6513](https://github.com/StarRocks/starrocks/issues/6513)
- 重启FE后，由于BDB JE v7.3.8中的一些问题和磁盘使用量异常增加，FE遇到高I/O并显示没有恢复为正常状态。FE在回滚到BDB JE v7.3.7后恢复正常。[#6634](https://github.com/StarRocks/starrocks/issues/6634)

## 2.1.7

发布日期：2022年5月26日

### 改进

对于窗口函数，其中frame设置为ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW，如果计算涉及的分区很大，StarRocks在执行计算之前会缓存分区的所有数据。在这种情况下，会消耗大量内存资源。StarRocks已经优化，在这种情况下不再缓存分区的所有数据。[5829](https://github.com/StarRocks/starrocks/issues/5829)

### Bug修复

已修复以下错误：

- 当数据加载到使用Primary Key表的表中时，如果系统中存储的每个数据版本的创建时间不是单调增加的，可能会发生数据处理错误，导致BE停止。[#6046](https://github.com/StarRocks/starrocks/issues/6046)
- 一些图形用户界面（GUI）工具会自动配置 set_sql_limit 变量。结果，SQL 语句 ORDER BY LIMIT 将被忽略，因此查询将返回不正确数量的行。[#5966](https://github.com/StarRocks/starrocks/issues/5966)
- 如果在数据库上执行 DROP SCHEMA 语句，则该数据库将被强制删除且无法恢复。[#6201](https://github.com/StarRocks/starrocks/issues/6201)
- 当加载 JSON 格式的数据时，如果数据含有 JSON 格式错误，BE 将停止运行。例如，键值对之间没有用逗号（,）分隔。[#6098](https://github.com/StarRocks/starrocks/issues/6098)
- 当大量数据高并发加载时，运行的用于将数据写入磁盘的任务会在 BE 上堆积。在这种情况下，BE 可能会停止运行。[#3877](https://github.com/StarRocks/starrocks/issues/3877)
- StarRocks 在对表执行模式更改之前会评估所需的内存量。如果表包含大量的 STRING 字段，则内存评估结果可能不准确。在这种情况下，如果所需的估计内存量超过了单个模式更改操作允许的最大内存，则应正确运行的模式更改操作将遇到错误。[#6322](https://github.com/StarRocks/starrocks/issues/6322)
- 在对使用主键表的表执行模式更改后，当向该表加载数据时可能会出现“重复键 xxx”错误。[#5878](https://github.com/StarRocks/starrocks/issues/5878)
- 如果在 Shuffle Join 操作期间执行低基数优化，则可能会发生分区错误。[#4890](https://github.com/StarRocks/starrocks/issues/4890)
- 如果一个共置组（CG）包含大量的表且频繁向这些表加载数据，则 CG 可能无法保持稳定状态。在这种情况下，JOIN 语句不支持共置连接操作。StarRocks 已经优化以在数据加载期间等待更长时间。这样可以最大程度地保证数据加载到的平板副本的完整性。

## 2.1.6

发行日期：2022 年 5 月 10 日

### 错误修复

已修复以下错误：

- 在执行多个 DELETE 操作后运行查询时，如果对查询执行了低基数列的优化，可能会得到不正确的查询结果。[#5712](https://github.com/StarRocks/starrocks/issues/5712)
- 如果在特定的数据摄入阶段迁移平板，数据将继续写入平板所在的原始磁盘。结果，数据丢失，查询无法正常运行。[#5160](https://github.com/StarRocks/starrocks/issues/5160)
- 如果在 DECIMAL 和 STRING 数据类型之间转换值，则返回值可能具有意外的精度。[#5608](https://github.com/StarRocks/starrocks/issues/5608)
- 如果将 DECIMAL 值乘以 BIGINT 值，可能会发生算术溢出。已进行一些调整和优化以修复此错误。[#4211](https://github.com/StarRocks/starrocks/pull/4211)

## 2.1.5

发行日期：2022 年 4 月 27 日

### 错误修复

已修复以下错误：

- 当 DECIMAL 乘法溢出时，计算结果不正确。错误修复后，当 DECIMAL 乘法溢出时返回 NULL。
- 当统计数据与实际统计数据有较大偏差时，共置连接的优先级可能低于广播连接。结果，查询规划程序可能不会选择共置连接作为更合适的连接策略。[#4817](https://github.com/StarRocks/starrocks/pull/4817)
- 当存在超过 4 个表加入时，复杂表达式的计划错误导致查询失败。
- 当 Shuffle Join 中的随机键列是低基数列时，BE 可能停止工作。[#4890](https://github.com/StarRocks/starrocks/issues/4890)
- 当 SPLIT 函数使用 NULL 参数时，BE 可能停止工作。[#4092](https://github.com/StarRocks/starrocks/issues/4092)

## 2.1.4

发行日期：2022 年 4 月 8 日

### 新功能

- 支持 `UUID_NUMERIC` 函数，返回 LARGEINT 值。与 `UUID` 函数相比，`UUID_NUMERIC` 函数的性能可以提高近 2 个数量级。

### 错误修复

已修复以下错误：

- 删除列、添加新分区并克隆平板后，新旧平板中的列唯一标识可能不相同，这可能会导致 BE 停止工作，因为系统使用共享的平板模式。[#4514](https://github.com/StarRocks/starrocks/issues/4514)
- 当数据加载到 StarRocks 外部表时，如果目标 StarRocks 集群的配置 FE 不是主领导者，则会导致 FE 停止工作。[#4573](https://github.com/StarRocks/starrocks/issues/4573)
- `CAST` 函数在 StarRocks 版本 1.19 和 2.1 中的结果不同。[#4701](https://github.com/StarRocks/starrocks/pull/4701)
- 在 Duplicate Key 表执行模式更改并同时创建物化视图时，可能导致查询结果不正确。[#4839](https://github.com/StarRocks/starrocks/issues/4839)

## 2.1.3

发行日期：2022 年 3 月 19 日

### 错误修复

已修复以下错误：

- 由于使用批处理发布版本，BE 失败可能导致可能数据丢失的问题。[#3140](https://github.com/StarRocks/starrocks/issues/3140)
- 由于不当的执行计划，一些查询可能会因内存限制超出而失败。
- 在不同的整理过程中，副本之间的校验和可能不一致。[#3438](https://github.com/StarRocks/starrocks/issues/3438)
- 在某些情况下，当 JSON 重排序投影无法正确处理时，查询可能失败。[#4056](https://github.com/StarRocks/starrocks/pull/4056)

## 2.1.2

发行日期：2022 年 3 月 14 日

### 错误修复

已修复以下错误：

- 在从版本 1.19 升级到 2.1 的滚动升级中，由于两个版本之间的不匹配的块大小，BE 节点停止工作。[#3834](https://github.com/StarRocks/starrocks/issues/3834)
- 当 StarRocks 从版本 2.0 更新到 2.1 时，加载任务可能失败。[#3828](https://github.com/StarRocks/starrocks/issues/3828)
- 在单平板表联接中没有合适的执行计划时，查询可能失败。[#3854](https://github.com/StarRocks/starrocks/issues/3854)
- 当 FE 节点收集信息以构建低基数优化的全局字典时可能会发生死锁问题。[#3839](https://github.com/StarRocks/starrocks/issues/3839)
- 当 BE 节点由于死锁而处于暂停状态时，查询可能失败。
- 当 SHOW VARIABLES 命令失败时，BI 工具无法连接到 StarRocks。[#3708](https://github.com/StarRocks/starrocks/issues/3708)

## 2.1.0

发行日期：2022 年 2 月 24 日

### 新功能

- [预览] 现在支持 Iceberg 外部表。
- [预览] 管道引擎现已可用。这是一种新的多核调度设计的执行引擎。查询并发性可以自适应调整，而无需设置 parallel_fragment_exec_instance_num 参数。这也改进了高并发场景下的性能。
- 支持 CTAS（CREATE TABLE AS SELECT）语句，使 ETL 和表创建更加容易。
- 支持 SQL 指纹。SQL 指纹在 audit.log 中生成，有助于定位慢查询。
- 支持 ANY_VALUE、ARRAY_REMOVE 和 SHA2 函数。

### 改进

- 优化了整理。平面表可以包含多达 10,000 个列。
- 优化了首次扫描和页面缓存的性能。减少了随机 I/O 以提高首次扫描性能。如果首次扫描发生在 SATA 硬盘上，改进将更加明显。StarRocks 的页面缓存可以存储原始数据，消除了对 bitshuffle 编码和不必要解码的需求。这提高了缓存命中率和查询效率。
- 支持主键表中的模式更改。您可以使用 `Alter table` 添加、删除和修改位图索引。
- [预览] 字符串的大小可达到 1 MB。
- 优化了 JSON 加载性能。可以在单个文件中加载超过 100 MB 的 JSON 数据。
- 优化了位图索引的性能。
- 优化了 StarRocks Hive 外部表的性能。可以读取 CSV 格式的数据。
- 在 `create table` 语句中支持 DEFAULT CURRENT_TIMESTAMP。
- StarRocks支持加载具有多个分隔符的CSV文件。

### Bug 修复

已修复以下bug：

- 如果在用于加载JSON数据的命令中指定了jsonpaths，则自动__op映射不会生效。
- 在使用Broker Load进行数据加载时，BE节点失败，因为源数据发生了更改。
- 创建了物化视图后，某些SQL语句报错。
- 由于带引号的jsonpaths，例行负载无法正常工作。
- 当要查询的列数超过200时，查询并发性急剧下降。

### 行为变更

禁用Colocation Group的API从`DELETE /api/colocate/group_stable`更改为`POST /api/colocate/group_unstable`。

### 其他

flink-connector-starrocks现在可用于Flink批量读取StarRocks数据。与JDBC连接器相比，这提高了数据读取效率。