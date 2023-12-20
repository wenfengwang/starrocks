---
displayed_sidebar: English
---

# StarRocks版本2.3

## 2.3.18

发布日期：2023年10月11日

### Bug修复

修复了以下问题：

- 第三方库librdkafka的bug导致Routine Load作业的加载任务在数据加载过程中卡住，并且新创建的加载任务也无法执行。[#28301](https://github.com/StarRocks/starrocks/pull/28301)
- Spark或Flink连接器可能会因为内存统计不准确而导致数据导出失败。[#31200](https://github.com/StarRocks/starrocks/pull/31200) [#30751](https://github.com/StarRocks/starrocks/pull/30751)
- 当Stream Load作业使用关键字`if`时，BE崩溃。[#31926](https://github.com/StarRocks/starrocks/pull/31926)
- 数据加载到分区的StarRocks外部表时，报告错误`"get TableMeta failed from TNetworkAddress"`。[#30466](https://github.com/StarRocks/starrocks/pull/30466)

## 2.3.17

发布日期：2023年9月4日

### Bug修复

修复了以下问题：

- Routine Load作业无法消费数据。[#29883](https://github.com/StarRocks/starrocks/issues/29883) [#18550](https://github.com/StarRocks/starrocks/pull/18550)

## 2.3.16

发布日期：2023年8月4日

### Bug修复

修复了以下问题：

由阻塞的LabelCleaner线程导致的FE内存泄漏。[#28311](https://github.com/StarRocks/starrocks/pull/28311) [#28636](https://github.com/StarRocks/starrocks/pull/28636)

## 2.3.15

发布日期：2023年7月31日

## 改进

- 优化了tablet调度逻辑，防止tablet长时间处于pending状态或在某些情况下FE崩溃。[#21647](https://github.com/StarRocks/starrocks/pull/21647) [#23062](https://github.com/StarRocks/starrocks/pull/23062) [#25785](https://github.com/StarRocks/starrocks/pull/25785)
- 优化了TabletChecker的调度逻辑，防止检查器重复调度未修复的tablet。[#27648](https://github.com/StarRocks/starrocks/pull/27648)
- 分区元数据记录visibleTxnId，对应tablet副本的可见版本。当副本的版本与其他版本不一致时，更容易追踪创建该版本的事务。[#27924](https://github.com/StarRocks/starrocks/pull/27924)

### Bug修复

修复了以下问题：

- FE中不正确的表级扫描统计信息导致与表查询和加载相关的指标不准确。[#28022](https://github.com/StarRocks/starrocks/pull/28022)
- 如果Join键是大型BINARY列，BE可能崩溃。[#25084](https://github.com/StarRocks/starrocks/pull/25084)
- 在某些场景下，聚合操作符可能触发线程安全问题，导致BE崩溃。[#26092](https://github.com/StarRocks/starrocks/pull/26092)
- 使用[RESTORE](../sql-reference/sql-statements/data-definition/RESTORE.md)恢复数据后，tablet在BE和FE之间的版本号不一致。[#26518](https://github.com/StarRocks/starrocks/pull/26518/files)
- 使用[RECOVER](../sql-reference/sql-statements/data-definition/RECOVER.md)恢复表后，无法自动创建分区。[#26813](https://github.com/StarRocks/starrocks/pull/26813)
- 如果使用INSERT INTO加载的数据不符合质量要求，并且启用了数据加载的严格模式，则加载事务会陷入Pending状态，DDL语句也会挂起。[#27140](https://github.com/StarRocks/starrocks/pull/27140)
- 如果启用了低基数优化，某些INSERT作业返回`[42000][1064] Dict Decode failed, Dict can't take cover all key :0`。[#27395](https://github.com/StarRocks/starrocks/pull/27395)
- 在某些情况下，当Pipeline未启用时，INSERT INTO SELECT操作超时。[#26594](https://github.com/StarRocks/starrocks/pull/26594)
- 查询条件为`WHERE partition_column < xxx`且`xxx`中的值仅精确到小时而不是分钟和秒时，查询返回无数据，例如`2023-7-21 22`。[#27780](https://github.com/StarRocks/starrocks/pull/27780)

## 2.3.14

发布日期：2023年6月28日

### 改进

- 优化了CREATE TABLE超时返回的错误信息，并增加了参数调优提示。[#24510](https://github.com/StarRocks/starrocks/pull/24510)
- 优化了具有大量累积tablet版本的主键表的内存使用。[#20760](https://github.com/StarRocks/starrocks/pull/20760)
- StarRocks外部表元数据的同步已更改为在数据加载期间进行。[#24739](https://github.com/StarRocks/starrocks/pull/24739)
- 删除了NetworkTime对系统时钟的依赖，以修复由于服务器之间系统时钟不一致而导致的NetworkTime不正确。[#24858](https://github.com/StarRocks/starrocks/pull/24858)

### Bug修复

修复了以下问题：

- 当对频繁执行TRUNCATE操作的小表应用低基数字典优化时，查询可能遇到错误。[#23185](https://github.com/StarRocks/starrocks/pull/23185)
- 当查询包含UNION且其第一个子查询为常量NULL的视图时，BE可能崩溃。[#13792](https://github.com/StarRocks/starrocks/pull/13792)
- 在某些情况下，基于Bitmap Index的查询可能返回错误。[#23484](https://github.com/StarRocks/starrocks/pull/23484)
- 在BE中，将DOUBLE或FLOAT值舍入为DECIMAL值的结果与FE中的结果不一致。[#23152](https://github.com/StarRocks/starrocks/pull/23152)
- 如果数据加载与schema change同时发生，schema change有时可能会挂起。[#23456](https://github.com/StarRocks/starrocks/pull/23456)
- 使用Broker Load、Spark连接器或Flink连接器将Parquet文件加载到StarRocks时，可能出现BE OOM问题。[#25254](https://github.com/StarRocks/starrocks/pull/25254)
- 在查询中指定常量在ORDER BY子句中，并且存在LIMIT子句时，返回错误消息`unknown error`。[#25538](https://github.com/StarRocks/starrocks/pull/25538)

## 2.3.13

发布日期：2023年6月1日

### 改进

- 优化了因小`thrift_server_max_worker_thread`值导致的INSERT INTO ... SELECT过期错误消息。[#21964](https://github.com/StarRocks/starrocks/pull/21964)
- 减少了使用`bitmap_contains`函数的多表连接的内存消耗并优化了性能。[#20617](https://github.com/StarRocks/starrocks/pull/20617) [#20653](https://github.com/StarRocks/starrocks/pull/20653)

### Bug修复

修复了以下错误：

- 截断分区失败，因为TRUNCATE操作对分区名称大小写敏感。[#21809](https://github.com/StarRocks/starrocks/pull/21809)
- 从Parquet文件加载int96时间戳数据导致数据溢出。[#22355](https://github.com/StarRocks/starrocks/issues/22355)
- 删除物化视图后，停用BE失败。[#22743](https://github.com/StarRocks/starrocks/issues/22743)
- 当查询的执行计划包含Broadcast Join后跟Bucket Shuffle Join时，例如`SELECT * FROM t1 JOIN [Broadcast] t2 ON t1.a = t2.b JOIN [Bucket] t3 ON t2.b = t3.c`，并且Broadcast Join中左表的equijoin key的数据在发送到Bucket Shuffle Join之前被删除，BE可能崩溃。[#23227](https://github.com/StarRocks/starrocks/pull/23227)
- 当查询的执行计划包含Cross Join后跟Hash Join，并且分片实例中Hash Join的右表为空时，返回结果可能不正确。[#23877](https://github.com/StarRocks/starrocks/pull/23877)
- 由于在为物化视图创建临时分区时失败，停用BE失败。[#22745](https://github.com/StarRocks/starrocks/pull/22745)
- 如果SQL语句包含多个转义字符的STRING值，则无法解析SQL语句。[#23119](https://github.com/StarRocks/starrocks/issues/23119)
- 查询分区列中最大值的数据失败。[#23153](https://github.com/StarRocks/starrocks/issues/23153)
- StarRocks从v2.4回滚到v2.3后，加载作业失败。[#23642](https://github.com/StarRocks/starrocks/pull/23642)
- 与列修剪和重用相关的问题。[#16624](https://github.com/StarRocks/starrocks/issues/16624)

## 2.3.12

发布日期：2023年4月25日

### 改进

支持隐式转换，如果表达式的返回值可以转换为有效的布尔值。[#21792](https://github.com/StarRocks/starrocks/pull/21792)

### Bug修复

修复了以下错误：

- 如果在表级别授予用户LOAD_PRIV，则在事务回滚时，如果加载作业失败，会出现错误消息`Access denied; you need (at least one of) the LOAD privilege(s) for this operation`。[#21129](https://github.com/StarRocks/starrocks/issues/21129)
- 执行ALTER SYSTEM DROP BACKEND删除BE后，该BE上复制数为2的表的副本无法修复。在这种情况下，将数据加载到这些表中会失败。[#20681](https://github.com/StarRocks/starrocks/pull/20681)
- 当在CREATE TABLE中使用不支持的数据类型时，返回NPE。[#20999](https://github.com/StarRocks/starrocks/issues/20999)
- Broadcast Join的短路逻辑异常，导致查询结果不正确。[#20952](https://github.com/StarRocks/starrocks/issues/20952)
- 使用物化视图后，磁盘使用量可能显著增加。[#20590](https://github.com/StarRocks/starrocks/pull/20590)
- 无法完全卸载Audit Loader插件。[#20468](https://github.com/StarRocks/starrocks/issues/20468)
- `INSERT INTO XXX SELECT`的结果中显示的行数可能与`SELECT COUNT(*) FROM XXX`的结果不匹配。[#20084](https://github.com/StarRocks/starrocks/issues/20084)
- 如果子查询使用窗口函数并且父查询使用GROUP BY子句，则查询结果无法聚合。[#19725](https://github.com/StarRocks/starrocks/issues/19725)
- 当BE启动时，BE进程存在但所有BE的端口都无法打开。[#19347](https://github.com/StarRocks/starrocks/pull/19347)
- 如果磁盘I/O过高，主键表上的事务提交缓慢，因此这些表上的查询可能会返回错误"backend not found"。[#18835](https://github.com/StarRocks/starrocks/issues/18835)
```
- 弃用了FE参数`default_storage_medium`。表的存储介质由系统自动推断。[#14394](https://github.com/StarRocks/starrocks/pull/14394)

### Bug修复

修复了以下错误：

- 如果一个用户的LOAD_PRIV在表级别被授予，当加载作业失败时，在事务回滚时返回错误消息`Access denied; you need (at least one of) the LOAD privilege(s) for this operation`。[# 21129](https://github.com/StarRocks/starrocks/issues/21129)
- 执行ALTER SYSTEM DROP BACKEND以删除BE后，该BE上设置副本数为2的表的副本无法修复。在这种情况下，加载到这些表中的数据失败。[# 20681](https://github.com/StarRocks/starrocks/pull/20681)
- 当在CREATE TABLE中使用不受支持的数据类型时，返回NPE错误。[# 20999](https://github.com/StarRocks/starrocks/issues/20999)
- 广播连接包含UNION的视图的第一个子节点是常量NULL时，查询该视图会导致BE崩溃。[# 13792](https://github.com/StarRocks/starrocks/pull/13792)
- 在某些情况下，基于位图索引的查询可能返回错误。[# 23484](https://github.com/StarRocks/starrocks/pull/23484)
- BE中DOUBLE或FLOAT值舍入为DECIMAL值的结果与FE中的结果不一致。[# 23152](https://github.com/StarRocks/starrocks/pull/23152)
- 如果在数据加载同时执行模式更改，模式更改有时可能会挂起。[# 23456](https://github.com/StarRocks/starrocks/pull/23456)
- 当使用INSERT INTO将Parquet文件加载到StarRocks中时，如果Broker Load、Spark连接器或Flink连接器加载的Parquet文件中存在BE OOM问题。[# 25254](https://github.com/StarRocks/starrocks/pull/25254)
- 当在ORDER BY子句中指定常量并且查询中有LIMIT子句时，返回错误消息`unknown error`。[# 25538](https://github.com/StarRocks/starrocks/pull/25538)
发布日期：2022年8月22日

### 新功能

- 主键表支持完整的DELETE WHERE语法。有关更多信息，请参见[DELETE](../sql-reference/sql-statements/data-manipulation/DELETE.md#delete-data-by-primary-key)。
- 主键表支持持久化主键索引。您可以选择将主键索引持久化到磁盘而不是内存中，从而显著减少内存使用量。有关更多信息，请参见[主键表](../table_design/table_types/primary_key_table.md)。
- 全局字典可以在实时数据导入期间进行更新，优化查询性能，并为字符串数据提供2倍的查询性能。
- CREATE TABLE AS SELECT语句可以异步执行。有关更多信息，请参见[CREATE TABLE AS SELECT](../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)。
- 支持以下与资源组相关的功能：
  - 监控资源组：您可以在审核日志中查看查询的资源组，并通过调用API获取资源组的指标。有关更多信息，请参见[监控和警报](../administration/Monitor_and_Alert.md#monitor-and-alerting)。
  - 限制大查询在CPU、内存和I/O资源上的消耗：您可以基于分类器或通过配置会话变量将查询路由到特定的资源组。有关更多信息，请参见[资源组](../administration/resource_group.md)。
- JDBC外部表可用于方便地查询Oracle、PostgreSQL、MySQL、SQLServer、ClickHouse等数据库中的数据。StarRocks还支持谓词下推，提高查询性能。有关更多信息，请参见[JDBC兼容数据库的外部表](../data_source/External_table.md#external-table-for-a-JDBC-compatible-database)。
- [预览]发布了新的数据源连接器框架，支持外部目录。您可以使用外部目录直接访问和查询Hive数据，而无需创建外部表。有关更多信息，请参见[使用目录管理内部和外部数据](../data_source/catalog/query_external_data.md)。
- 添加了以下函数：
  - [window_funnel](../sql-reference/sql-functions/aggregate-functions/window_funnel.md)
  - [ntile](../sql-reference/sql-functions/Window_function.md)
  - [bitmap_union_count](../sql-reference/sql-functions/bitmap-functions/bitmap_union_count.md)、[base64_to_bitmap](../sql-reference/sql-functions/bitmap-functions/base64_to_bitmap.md)、[array_to_bitmap](../sql-reference/sql-functions/array-functions/array_to_bitmap.md)
  - [week](../sql-reference/sql-functions/date-time-functions/week.md)、[time_slice](../sql-reference/sql-functions/date-time-functions/time_slice.md)

### 改进

- 合并大量元数据的压缩机制更快。这可以防止元数据挤压和过多的磁盘使用，这些问题可能在频繁数据更新后不久发生。
- 优化了加载Parquet文件和压缩文件的性能。
- 优化了创建物化视图的机制。优化后，物化视图的创建速度比以前快10倍。
- 优化了以下运算符的性能：
  - TopN和排序运算符
  - 包含函数的等价比较运算符在被推入扫描运算符时可以使用Zone Map索引。
- 优化了Apache Hive™外部表。
  - 当Apache Hive™表存储为Parquet、ORC或CSV格式时，当您在相应的Hive外部表上执行REFRESH语句时，由Hive引起的ADD COLUMN或REPLACE COLUMN引起的架构更改可以同步到StarRocks。有关更多信息，请参见[Hive外部表](../data_source/External_table.md#hive-external-table)。
  - 可以修改`hive.metastore.uris`以用于Hive资源。有关更多信息，请参见[ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md)。
- 优化了Apache Iceberg外部表的性能。可以使用自定义目录创建Iceberg资源。有关更多信息，请参见[Apache Iceberg外部表](../data_source/External_table.md#apache-iceberg-external-table)。
- 优化了Elasticsearch外部表的性能。可以禁用对Elasticsearch集群中的数据节点地址进行嗅探。有关更多信息，请参见[Elasticsearch外部表](../data_source/External_table.md#elasticsearch-external-table)。
- 优化了Elasticsearch外部表的性能。可以禁用对Elasticsearch集群中的数据节点地址进行嗅探。有关更多信息，请参见[Elasticsearch外部表](../data_source/External_table.md#elasticsearch-external-table)。
- 当sum()函数接受数字字符串时，它会隐式转换为数字字符串。
- year()、month()和day()函数支持DATE数据类型。

### Bug修复

修复了以下错误：

- 由于tablet数量过多，CPU利用率激增。
- 导致“无法准备tablet reader”错误的问题。
- FEs无法重新启动。[#5642](https://github.com/StarRocks/starrocks/issues/5642 )  [#4969](https://github.com/StarRocks/starrocks/issues/4969 )  [#5580](https://github.com/StarRocks/starrocks/issues/5580)
- CTAS语句在包含JSON函数时无法成功运行。[#6498](https://github.com/StarRocks/starrocks/issues/6498)

### 其他

- StarGo是一个集群管理工具，可以部署、启动、升级和回滚集群，并管理多个集群。有关更多信息，请参见[使用StarGo部署StarRocks](../administration/stargo.md)。
- 在将StarRocks升级到2.3版本或部署StarRocks时，默认情况下启用了管道引擎。管道引擎可以提高高并发场景下和复杂查询的性能。如果您在使用StarRocks 2.3时检测到性能显著下降，可以通过执行`SET GLOBAL`语句将`enable_pipeline_engine`设置为`false`来禁用管道引擎。
- [SHOW GRANTS](../sql-reference/sql-statements/account-management/SHOW_GRANTS.md)语句与MySQL语法兼容，并以GRANT语句的形式显示分配给用户的权限。
- 建议将memory_limitation_per_thread_for_schema_change（BE配置项）使用默认值2 GB，并在数据量超过此限制时将数据写入磁盘。因此，如果您以前将此参数设置为较大的值，建议将其设置为2 GB，否则架构更改任务可能会占用大量内存。

### 升级说明

要回滚到升级前使用的上一个版本，请将`ignore_unknown_log_id`参数添加到每个FE的**fe.conf**文件中，并将该参数设置为`true`。该参数是必需的，因为在StarRocks v2.2.0中添加了新类型的日志。如果您不添加该参数，则无法回滚到以前的版本。我们建议在创建检查点后将`ignore_unknown_log_id`参数设置为`false`，然后重新启动FE以将FE恢复到以前的配置。
发布日期：2022 年 9 月 7 日

### 新功能

- 支持 Late Materialization，以加速 Parquet 格式外部表上基于范围过滤器的查询。[#9738](https://github.com/StarRocks/starrocks/pull/9738)
- 新增 `SHOW AUTHENTICATION` 语句，用于显示用户认证相关信息。[#9996](https://github.com/StarRocks/starrocks/pull/9996)

### 改进

- 提供了一个配置项，用于控制 StarRocks 是否递归遍历用于查询数据的分桶 Hive 表中的所有数据文件。[#10239](https://github.com/StarRocks/starrocks/pull/10239)
- 将资源组类型 `realtime` 更名为 `short_query`。[#10247](https://github.com/StarRocks/starrocks/pull/10247)
- StarRocks 默认不再区分 Hive 外部表中的大小写字母。[#10187](https://github.com/StarRocks/starrocks/pull/10187)

### Bug 修复

修复了以下错误：

- 当 Elasticsearch 外部表被划分为多个分片时，查询可能会意外退出。[#10369](https://github.com/StarRocks/starrocks/pull/10369)
- 当子查询被重写为公共表达式 (CTE) 时，StarRocks 会抛出错误。[#10397](https://github.com/StarRocks/starrocks/pull/10397)
- 当加载大量数据时，StarRocks 会抛出错误。[#10370](https://github.com/StarRocks/starrocks/issues/10370) [#10380](https://github.com/StarRocks/starrocks/issues/10380)
- 当多个目录配置了相同的 Thrift 服务 IP 地址时，删除一个目录会使其他目录的增量元数据更新失效。[#10511](https://github.com/StarRocks/starrocks/pull/10511)
- BE 的内存消耗统计不准确。[#9837](https://github.com/StarRocks/starrocks/pull/9837)
- StarRocks 在查询主键表时抛出错误。[#10811](https://github.com/StarRocks/starrocks/pull/10811)
- 即使用户对逻辑视图具有 SELECT 权限，也不允许查询这些视图。[#10563](https://github.com/StarRocks/starrocks/pull/10563)
- StarRocks 对逻辑视图的命名没有限制。现在逻辑视图需要遵循与表相同的命名规范。[#10558](https://github.com/StarRocks/starrocks/pull/10558)

### 行为变更

- 添加 BE 配置 `max_length_for_bitmap_function`，默认值为 1000000，用于位图函数，以及 `max_length_for_to_base64`，默认值为 200000，用于 base64 函数，以防止崩溃。[#10851](https://github.com/StarRocks/starrocks/pull/10851)

## 2.3.1

发布日期：2022 年 8 月 22 日

### 改进

- Broker Load 支持将 Parquet 文件中的 List 类型转换为非嵌套的 ARRAY 数据类型。[#9150](https://github.com/StarRocks/starrocks/pull/9150)
- 优化了 JSON 相关函数（json_query、get_json_string 和 get_json_int）的性能。[#9623](https://github.com/StarRocks/starrocks/pull/9623)
- 优化错误提示：在查询 Hive、Iceberg 或 Hudi 时，如果查询的列数据类型不受 StarRocks 支持，系统会抛出异常。[#10139](https://github.com/StarRocks/starrocks/pull/10139)
- 减少资源组的调度延迟，优化资源隔离性能。[#10122](https://github.com/StarRocks/starrocks/pull/10122)

### Bug 修复

修复了以下错误：

- 由于错误地推送了 `limit` 运算符，导致 Elasticsearch 外部表的查询结果错误。[#9952](https://github.com/StarRocks/starrocks/pull/9952)
- 使用 `limit` 运算符时，Oracle 外部表的查询失败。[#9542](https://github.com/StarRocks/starrocks/pull/9542)
- 当所有 Kafka Broker 在 Routine Load 过程中停止时，BE 会被阻塞。[#9935](https://github.com/StarRocks/starrocks/pull/9935)
- 当查询的 Parquet 文件数据类型与对应外部表不匹配时，BE 崩溃。[#10107](https://github.com/StarRocks/starrocks/pull/10107)
- 查询超时，因为外部表的扫描范围为空。[#10091](https://github.com/StarRocks/starrocks/pull/10091)
- 当子查询中包含 `ORDER BY` 子句时，系统抛出异常。[#10180](https://github.com/StarRocks/starrocks/pull/10180)
- Hive Metastore 在异步重新加载 Hive 元数据时挂起。[#10132](https://github.com/StarRocks/starrocks/pull/10132)

## 2.3.0

发布日期：2022 年 7 月 29 日

### 新功能

- 主键表支持完整的 DELETE WHERE 语法。有关详细信息，请参见 [DELETE](../sql-reference/sql-statements/data-manipulation/DELETE.md#delete-data-by-primary-key)。
- 主键表支持持久化主键索引。您可以选择将主键索引持久化到磁盘而不是内存中，显著减少内存使用。有关详细信息，请参见 [主键表](../table_design/table_types/primary_key_table.md)。
- 全局字典可以在实时数据摄取过程中更新，优化查询性能，为字符串数据提供 2 倍的查询性能。
- CREATE TABLE AS SELECT 语句可以异步执行。有关详细信息，请参见 [CREATE TABLE AS SELECT](../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)。
- 支持以下资源组相关功能：
  - 监控资源组：您可以在审计日志中查看查询的资源组，并通过调用 API 获取资源组的指标。有关详细信息，请参见 [监控和报警](../administration/Monitor_and_Alert.md#monitor-and-alerting)。
  - 限制大型查询对 CPU、内存和 I/O 资源的消耗：您可以根据分类器或通过配置会话变量将查询路由到特定资源组。有关详细信息，请参见 [资源组](../administration/resource_group.md)。
- JDBC 外部表可以方便地查询 Oracle、PostgreSQL、MySQL、SQLServer、ClickHouse 等数据库中的数据。StarRocks 还支持谓词下推，提高查询性能。有关详细信息，请参见 [JDBC 兼容数据库的外部表](../data_source/External_table.md#external-table-for-a-JDBC-compatible-database)。
- [预览] 发布了新的数据源连接器框架，以支持外部目录。您可以使用外部目录直接访问和查询 Hive 数据，无需创建外部表。有关详细信息，请参见 [使用目录管理内部和外部数据](../data_source/catalog/query_external_data.md)。
- 新增以下函数：
  - [window_funnel](../sql-reference/sql-functions/aggregate-functions/window_funnel.md)
  - [ntile](../sql-reference/sql-functions/Window_function.md)
  - [bitmap_union_count](../sql-reference/sql-functions/bitmap-functions/bitmap_union_count.md)、[base64_to_bitmap](../sql-reference/sql-functions/bitmap-functions/base64_to_bitmap.md)、[array_to_bitmap](../sql-reference/sql-functions/array-functions/array_to_bitmap.md)
  - [week](../sql-reference/sql-functions/date-time-functions/week.md)、[time_slice](../sql-reference/sql-functions/date-time-functions/time_slice.md)

### 改进

- 压缩机制可以更快地合并大量元数据。这防止了频繁数据更新后不久发生的元数据挤压和过多的磁盘使用。
- 优化了加载 Parquet 文件和压缩文件的性能。
- 优化了物化视图的创建机制。优化后，物化视图的创建速度比之前快了 10 倍。
- 优化了以下算子的性能：
  - TopN 和排序算子
  - 当包含函数的等价比较算子被下推到扫描算子时，可以使用 Zone Map 索引。
- 优化了 Apache Hive™ 外部表。
  - 当 Apache Hive™ 表以 Parquet、ORC 或 CSV 格式存储时，通过对相应 Hive 外部表执行 REFRESH 语句，可以将 Hive 上的 ADD COLUMN 或 REPLACE COLUMN 引起的架构更改同步到 StarRocks。有关更多信息，请参阅 [Hive 外部表](../data_source/External_table.md#hive-external-table)。
  - 可以修改 Hive 资源的 `hive.metastore.uris`。有关更多信息，请参阅 [ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md)。
- 优化了 Apache Iceberg 外部表的性能。可以使用自定义目录创建 Iceberg 资源。有关更多信息，请参阅 [Apache Iceberg 外部表](../data_source/External_table.md#apache-iceberg-external-table)。
- 优化了 Elasticsearch 外部表的性能。可以禁用对 Elasticsearch 集群中数据节点地址的嗅探。有关更多信息，请参阅 [Elasticsearch 外部表](../data_source/External_table.md#elasticsearch-external-table)。
- 当 sum() 函数接受数字字符串时，它会隐式转换数字字符串。
- year()、month() 和 day() 函数支持 DATE 数据类型。

### Bug 修复

修复了以下错误：

- 由于平板电脑数量过多，CPU 利用率激增。
- 导致出现“无法准备平板电脑阅读器”的问题。
- FE 重启失败。[#5642](https://github.com/StarRocks/starrocks/issues/5642)  [#4969](https://github.com/StarRocks/starrocks/issues/4969)  [#5580](https://github.com/StarRocks/starrocks/issues/5580)
- 当 CTAS 语句包含 JSON 函数时，无法成功运行。[#6498](https://github.com/StarRocks/starrocks/issues/6498)

### 其他

- StarGo 是一个集群管理工具，可以部署、启动、升级和回滚集群，并管理多个集群。有关更多信息，请参阅 [使用 StarGo 部署 StarRocks](../administration/stargo.md)。
- 当您将 StarRocks 升级到版本 2.3 或部署 StarRocks 时，默认启用管道引擎。管道引擎可以提高高并发场景下简单查询和复杂查询的性能。如果您在使用 StarRocks 2.3 时检测到显著的性能下降，您可以通过执行 `SET GLOBAL` 语句将 `enable_pipeline_engine` 设置为 `false` 来禁用管道引擎。
- [SHOW GRANTS](../sql-reference/sql-statements/account-management/SHOW_GRANTS.md) 语句与 MySQL 语法兼容，以 GRANT 语句的形式显示分配给用户的权限。
- 建议将 BE 配置项 `memory_limitation_per_thread_for_schema_change` 的值设置为默认的 2GB，当数据量超过此限制时，数据将被写入磁盘。因此，如果您之前将此参数设置得较大，建议您将其调整为 2GB，否则 schema change 任务可能会占用大量内存。

### 升级注意事项
