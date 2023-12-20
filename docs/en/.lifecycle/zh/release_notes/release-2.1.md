---
displayed_sidebar: English
---

# StarRocks版本2.1

## 2.1.13

发布日期：2022年9月6日

### 改进

- 新增了BE配置项`enable_check_string_lengths`，用于检查加载数据的长度。此机制有助于防止因VARCHAR数据大小超出范围而导致的压缩失败。[#10380](https://github.com/StarRocks/starrocks/issues/10380)
- 当查询包含超过1000个OR运算符时，优化了查询性能。[#9332](https://github.com/StarRocks/starrocks/pull/9332)

### Bug修复

修复了以下bug：

- 当您从使用聚合表的表中查询ARRAY列（通过使用`REPLACE_IF_NOT_NULL`函数计算）时，可能会出现错误并导致BE崩溃。[#10144](https://github.com/StarRocks/starrocks/issues/10144)
- 如果查询中嵌套了多个`IFNULL()`函数，则查询结果不正确。[#5028](https://github.com/StarRocks/starrocks/issues/5028) [#10486](https://github.com/StarRocks/starrocks/pull/10486)
- 动态分区被截断后，分区中的tablet数量会从动态分区配置的值变更为默认值。[#10435](https://github.com/StarRocks/starrocks/issues/10435)
- 如果在使用Routine Load将数据加载到StarRocks时Kafka集群停止，可能会导致死锁，影响查询性能。[#8947](https://github.com/StarRocks/starrocks/issues/8947)
- 当查询同时包含子查询和`ORDER BY`子句时，会发生错误。[#10066](https://github.com/StarRocks/starrocks/pull/10066)

## 2.1.12

发布日期：2022年8月9日

### 改进

新增了两个参数`bdbje_cleaner_threads`和`bdbje_replay_cost_percent`，以加快BDB JE中的元数据清理速度。[#8371](https://github.com/StarRocks/starrocks/pull/8371)

### Bug修复

修复了以下bug：

- 一些查询被转发到Leader FE，导致`/api/query_detail`操作返回关于SQL语句（如SHOW FRONTENDS）的错误执行信息。[#9185](https://github.com/StarRocks/starrocks/issues/9185)
- BE终止后，当前进程未完全终止，导致BE无法成功重启。[#9175](https://github.com/StarRocks/starrocks/pull/9267)
- 当创建多个Broker Load作业来加载相同的HDFS数据文件时，如果一个作业遇到异常，其他作业可能无法正确读取数据，从而失败。[#9506](https://github.com/StarRocks/starrocks/issues/9506)
- 当表的schema发生变化时，相关变量未重置，导致查询表时出现错误（`no delete vector found tablet`）。[#9192](https://github.com/StarRocks/starrocks/issues/9192)

## 2.1.11

发布日期：2022年7月9日

### Bug修复

修复了以下bug：

- 当频繁向主键表加载数据时，表中的数据加载可能会暂停。[#7763](https://github.com/StarRocks/starrocks/issues/7763)
- 在低基数优化过程中，聚合表达式的处理顺序错误，导致`count distinct`函数返回意外结果。[#7659](https://github.com/StarRocks/starrocks/issues/7659)
- 由于LIMIT子句中的修剪规则无法正确处理，导致未返回结果。[#7894](https://github.com/StarRocks/starrocks/pull/7894)
- 如果在定义为查询连接条件的列上应用低基数优化的全局字典，则查询返回意外结果。[#8302](https://github.com/StarRocks/starrocks/issues/8302)

## 2.1.10

发布日期：2022年6月24日

### Bug修复

修复了以下bug：

- 反复切换Leader FE节点可能导致所有加载作业挂起并失败。[#7350](https://github.com/StarRocks/starrocks/issues/7350)
- 使用`DESC` SQL检查表架构时，DECIMAL(18,2)类型字段显示为DECIMAL64(18,2)。[#7309](https://github.com/StarRocks/starrocks/pull/7309)
- 当MemTable的内存使用估计超过4GB时，BE会崩溃，因为在数据倾斜加载期间，某些字段可能占用大量内存资源。[#7161](https://github.com/StarRocks/starrocks/issues/7161)
- 在合并过程中，如果输入行很多，由于max_rows_per_segment计算溢出，会创建大量小的segment文件。[#5610](https://github.com/StarRocks/starrocks/issues/5610)

## 2.1.8

发布日期：2022年6月9日

### 改进

- 优化了用于内部处理工作负载（如架构变更）的并发控制机制，减轻了前端（FE）元数据的压力。因此，如果这些加载作业同时运行以加载大量数据，加载作业不太可能堆积并减慢速度。[#6560](https://github.com/StarRocks/starrocks/pull/6560) [#6804](https://github.com/StarRocks/starrocks/pull/6804)
- 提高了StarRocks在高频数据加载方面的性能。[#6532](https://github.com/StarRocks/starrocks/pull/6532) [#6533](https://github.com/StarRocks/starrocks/pull/6533)

### Bug修复

修复了以下bug：

- ALTER操作日志未记录LOAD语句的所有信息。因此，在对例行加载作业执行ALTER操作后，创建检查点后作业的元数据丢失。[#6936](https://github.com/StarRocks/starrocks/issues/6936)
- 如果停止例行加载作业，可能发生死锁。[#6450](https://github.com/StarRocks/starrocks/issues/6450)
- 默认情况下，后端（BE）使用默认UTC+8时区进行加载作业。如果您的服务器使用UTC时区，使用Spark加载作业加载的表的DateTime列的时间戳将增加8小时。[#6592](https://github.com/StarRocks/starrocks/issues/6592)
- GET_JSON_STRING函数无法处理非JSON字符串。如果从JSON对象或数组中提取JSON值，函数返回NULL。该函数已优化，现在可以为JSON对象或数组返回等效的JSON格式化STRING值。[#6426](https://github.com/StarRocks/starrocks/issues/6426)
- 如果数据量大，架构变更可能因内存消耗过多而失败。已进行优化，允许您在架构变更的所有阶段指定内存消耗限制。[#6705](https://github.com/StarRocks/starrocks/pull/6705)
- 如果正在压缩的表中某列的重复值数量超过0x40000000，则压缩会暂停。[#6513](https://github.com/StarRocks/starrocks/issues/6513)
- FE重启后，由于BDB JE v7.3.8的问题，出现高I/O和磁盘使用量异常增加的情况，且没有恢复正常的迹象。FE在回滚到BDB JE v7.3.7后恢复正常。[#6634](https://github.com/StarRocks/starrocks/issues/6634)

## 2.1.7

发布日期：2022年5月26日

### 改进

对于窗口函数，其中框架设置为ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW，如果涉及的分区很大，StarRocks在执行计算之前会缓存分区的所有数据。在这种情况下，会消耗大量内存资源。StarRocks已优化，不再在这种情况下缓存分区的所有数据。[#5829](https://github.com/StarRocks/starrocks/issues/5829)

### Bug修复

修复了以下bug：

- 当数据加载到使用主键表的表时，如果系统中存储的各个数据版本的创建时间不是单调递增（可能由于系统时间回拨或相关未知bug），可能会导致数据处理错误。这类数据处理错误可能导致后端（BE）停止。[#6046](https://github.com/StarRocks/starrocks/issues/6046)
- 一些图形用户界面（GUI）工具自动配置了`set_sql_limit`变量。结果，SQL语句`ORDER BY LIMIT`被忽略，因此查询返回的行数不正确。[#5966](https://github.com/StarRocks/starrocks/issues/5966)
- 如果对数据库执行`DROP SCHEMA`语句，数据库将被强制删除且无法恢复。[#6201](https://github.com/StarRocks/starrocks/issues/6201)
```
- 当加载 JSON 格式的数据时，如果数据包含 JSON 格式错误，BE 可能会崩溃。例如，键值对之间没有用逗号（,）分隔。[#6098](https://github.com/StarRocks/starrocks/issues/6098)
- 当大量数据高并发加载时，写入磁盘的任务在 BE 上堆积，可能导致 BE 崩溃。[#3877](https://github.com/StarRocks/starrocks/issues/3877)
- StarRocks 在对表执行 schema change 之前会估计所需的内存量。如果表中包含大量 STRING 类型字段，内存估算结果可能不准确。在这种情况下，如果所需的估计内存量超过了单个 schema change 操作允许的最大内存，本应正常运行的 schema change 操作可能会出错。[#6322](https://github.com/StarRocks/starrocks/issues/6322)
- 在对使用 Primary Key 模型的表执行 schema change 后，向该表加载数据时可能会出现 "duplicate key xxx" 错误。[#5878](https://github.com/StarRocks/starrocks/issues/5878)
- 如果在 Shuffle Join 操作期间执行低基数优化，可能会出现分区错误。[#4890](https://github.com/StarRocks/starrocks/issues/4890)
- 如果 Colocation Group (CG) 包含大量表且数据频繁加载到这些表中，CG 可能无法保持稳定状态。在这种情况下，JOIN 语句不支持 Colocate Join 操作。StarRocks 已经过优化，在数据加载过程中会等待更长时间，以最大化保证加载数据的 tablet 副本的完整性。

## 2.1.6

发布日期：2022年5月10日

### Bug 修复

修复了以下 bug：

- 当执行多次 DELETE 操作后运行查询时，如果对查询进行了低基数列优化，则可能会得到不正确的查询结果。[#5712](https://github.com/StarRocks/starrocks/issues/5712)
- 如果在特定的数据摄取阶段迁移 tablet，数据会继续写入存储 tablet 的原始磁盘。结果，数据丢失，查询无法正常运行。[#5160](https://github.com/StarRocks/starrocks/issues/5160)
- 如果在 DECIMAL 和 STRING 数据类型之间转换值，返回值的精度可能会出现意外。[#5608](https://github.com/StarRocks/starrocks/issues/5608)
- 如果将 DECIMAL 值乘以 BIGINT 值，可能会发生算术溢出。为了修复这个 bug，进行了一些调整和优化。[#4211](https://github.com/StarRocks/starrocks/pull/4211)

## 2.1.5

发布日期：2022年4月27日

### Bug 修复

修复了以下 bug：

- 当小数乘法发生溢出时，计算结果不正确。修复后，小数乘法溢出时将返回 NULL。
- 当统计数据与实际统计数据有较大偏差时，Collocate Join 的优先级可能低于 Broadcast Join。因此，查询规划器可能不会选择 Collocate Join 作为更合适的 Join 策略。[#4817](https://github.com/StarRocks/starrocks/pull/4817)
- 当要连接的表超过 4 个时，由于复杂表达式的计划错误，查询可能失败。
- 当 Shuffle Join 下的 shuffle 列是低基数列时，BE 可能会崩溃。[#4890](https://github.com/StarRocks/starrocks/issues/4890)
- 当 SPLIT 函数使用 NULL 参数时，BE 可能会崩溃。[#4092](https://github.com/StarRocks/starrocks/issues/4092)

## 2.1.4

发布日期：2022年4月8日

### 新特性

- 支持 `UUID_NUMERIC` 函数，该函数返回 LARGEINT 值。与 `UUID` 函数相比，`UUID_NUMERIC` 函数的性能可以提高近两个数量级。

### Bug 修复

修复了以下 bug：

- 删除列、添加新分区和克隆 tablet 后，新旧 tablet 中列的唯一 ID 可能不相同，这可能会导致 BE 崩溃，因为系统使用共享的 tablet schema。[#4514](https://github.com/StarRocks/starrocks/issues/4514)
- 当数据加载到 StarRocks 外部表时，如果目标 StarRocks 集群配置的 FE 不是 Leader，它可能会导致 FE 崩溃。[#4573](https://github.com/StarRocks/starrocks/issues/4573)
- 在 StarRocks 1.19 和 2.1 版本中，`CAST` 函数的结果不同。[#4701](https://github.com/StarRocks/starrocks/pull/4701)
- 当 Duplicate Key 表执行 schema change 并同时创建物化视图时，查询结果可能不正确。[#4839](https://github.com/StarRocks/starrocks/issues/4839)

## 2.1.3

发布日期：2022年3月19日

### Bug 修复

修复了以下 bug：

- 由于 BE 故障可能导致数据丢失的问题（通过使用批量发布版本解决）。[#3140](https://github.com/StarRocks/starrocks/issues/3140)
- 由于不适当的执行计划，某些查询可能导致内存限制超出错误。
- 在不同的压缩过程中，副本之间的校验和可能不一致。[#3438](https://github.com/StarRocks/starrocks/issues/3438)
- 在某些情况下，当 JSON 重排序投影未正确处理时，查询可能失败。[#4056](https://github.com/StarRocks/starrocks/pull/4056)

## 2.1.2

发布日期：2022年3月14日

### Bug 修复

修复了以下 bug：

- 在从版本 1.19 到 2.1 的滚动升级中，由于两个版本之间的 chunk 大小不匹配，BE 节点可能停止工作。[#3834](https://github.com/StarRocks/starrocks/issues/3834)
- 当 StarRocks 从版本 2.0 更新到 2.1 时，加载任务可能失败。[#3828](https://github.com/StarRocks/starrocks/issues/3828)
- 当单表连接没有合适的执行计划时，查询可能失败。[#3854](https://github.com/StarRocks/starrocks/issues/3854)
- 当 FE 节点收集信息以构建全局字典进行低基数优化时，可能出现死锁问题。[#3839](https://github.com/StarRocks/starrocks/issues/3839)
- 当 BE 节点因死锁而处于假死状态时，查询可能失败。
- 当 BI 工具无法连接到 StarRocks 且 SHOW VARIABLES 命令失败时。[#3708](https://github.com/StarRocks/starrocks/issues/3708)

## 2.1.0

发布日期：2022年2月24日

### 新特性

- [预览] StarRocks 现在支持 Iceberg 外部表。
- [预览] 现在可用的 pipeline engine 是一个为多核调度设计的新执行引擎。查询并行度可以自适应调整，无需设置 `parallel_fragment_exec_instance_num` 参数。这也提高了高并发场景下的性能。
- 支持 CTAS（CREATE TABLE AS SELECT）语句，使 ETL 和表创建更加容易。
- 支持 SQL 指纹。在 audit.log 中生成 SQL 指纹，便于定位慢查询。
- 支持 ANY_VALUE、ARRAY_REMOVE 和 SHA2 函数。

### 改进

- 优化了压缩。平面表最多可以包含 10,000 列。
- 优化了首次扫描和页面缓存的性能。减少随机 I/O 以提高首次扫描性能。如果首次扫描发生在 SATA 磁盘上，则改进更加明显。StarRocks 的页面缓存可以存储原始数据，从而消除了 bitshuffle 编码和不必要的解码。这提高了缓存命中率和查询效率。
- 主键表支持 schema change。可以使用 `ALTER TABLE` 添加、删除和修改 bitmap 索引。
- [预览] 字符串大小最大可达 1 MB。
- 优化了 JSON 加载性能。可以在单个文件中加载超过 100 MB 的 JSON 数据。
- 优化了位图索引性能。
- 优化了 StarRocks Hive 外部表的性能。可以读取 CSV 格式的数据。
- `CREATE TABLE` 语句中支持 DEFAULT CURRENT_TIMESTAMP。
- StarRocks 支持加载带有多个分隔符的 CSV 文件。

### Bug 修复

修复了以下 bug：

- 如果在加载 JSON 数据的命令中指定了 jsonpaths，则自动 __op 映射不会生效。
- 使用 Broker Load 加载数据期间，如果源数据发生变化，BE 节点可能失败。
- 创建物化视图后，某些 SQL 语句可能报错。
- 由于指定了 `jsonpaths`，例行加载不起作用。
- 当查询的列数超过200时，查询并发性显著下降。

### 行为变更

禁用 Colocation Group 的 API 从 `DELETE /api/colocate/group_stable` 更改为 `POST /api/colocate/group_unstable`。

### 其他
