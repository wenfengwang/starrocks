---
displayed_sidebar: "Chinese"
---

# StarRocks 版本 2.4

## 2.4.5

发布日期：2023年4月21日

### 改进

- 禁止使用 List Partition 语法，因为它可能导致升级元数据时出错。[#15401](https://github.com/StarRocks/starrocks/pull/15401)
- 支持位图（BITMAP）、HLL 和百分位数（PERCENTILE）类型的物化视图。[#15731](https://github.com/StarRocks/starrocks/pull/15731)
- 优化了 `storage_medium` 的推断。当 BE 同时使用 SSD 和 HDD 作为存储设备时，如果指定属性 `storage_cooldown_time`，StarRocks 将 `storage_medium` 设置为 `SSD`。否则，StarRocks 将 `storage_medium` 设置为 `HDD`。[#18649](https://github.com/StarRocks/starrocks/pull/18649)
- 优化了线程转储的准确性。[#16748](https://github.com/StarRocks/starrocks/pull/16748)
- 通过在加载之前触发元数据压缩来优化加载效率。[#19347](https://github.com/StarRocks/starrocks/pull/19347)
- 优化了流加载规划器的超时时间。[#18992](https://github.com/StarRocks/starrocks/pull/18992/files)
- 通过禁止对值列进行统计信息收集，优化了唯一键表的性能。[#19563](https://github.com/StarRocks/starrocks/pull/19563)

### Bug 修复

已修复以下 bug：

- 在 CREATE TABLE 中使用不受支持的数据类型时返回 NPE。[#20999](https://github.com/StarRocks/starrocks/issues/20999)
- 使用短路广播连接时返回错误结果。[#20952](https://github.com/StarRocks/starrocks/issues/20952)
- 由错误的数据截断逻辑导致的磁盘占用问题。[#20590](https://github.com/StarRocks/starrocks/pull/20590)
- AuditLoader 插件无法安装或删除。[#20468](https://github.com/StarRocks/starrocks/issues/20468)
- 当调度一个 tablet 时抛出异常，同一批次中的其他 tablet 将永远无法被调度。[#20681](https://github.com/StarRocks/starrocks/pull/20681)
- 在创建同步物化视图时使用不受支持的 SQL 函数返回未知错误。[#20348](https://github.com/StarRocks/starrocks/issues/20348)
- 多个 COUNT DISTINCT 计算被错误重写。[#19714](https://github.com/StarRocks/starrocks/pull/19714)
- 在压缩的 tablet 上执行查询返回错误结果。[#20084](https://github.com/StarRocks/starrocks/issues/20084)
- 在具有聚合的查询中返回错误结果。[#19725](https://github.com/StarRocks/starrocks/issues/19725)
- 将 NULL Parquet 数据加载到非 NULL 列时未返回错误消息。[#19885](https://github.com/StarRocks/starrocks/pull/19885)
- 在连续达到资源组并发限制时，并发度指标增长缓慢。[#19363](https://github.com/StarRocks/starrocks/pull/19363)
- 在重放 `InsertOverwriteJob` 状态更改日志时，FE 启动失败。[#19061](https://github.com/StarRocks/starrocks/issues/19061)
- 主键表死锁。[#18488](https://github.com/StarRocks/starrocks/pull/18488)
- 对于 Co-location 表，可以通过类似 `ADMIN SET REPLICA STATUS PROPERTIES ("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");` 的语句手动将副本状态指定为 `bad`。如果 BEs 的数量小于或等于副本的数量，则无法修复损坏的副本。[#17876](https://github.com/StarRocks/starrocks/issues/17876)
- 由 ARRAY 相关函数引起的问题。[#18556](https://github.com/StarRocks/starrocks/pull/18556)

## 2.4.4

发布日期：2023年2月22日

### 改进

- 支持快速取消加载。[#15514](https://github.com/StarRocks/starrocks/pull/15514) [#15398](https://github.com/StarRocks/starrocks/pull/15398) [#15969](https://github.com/StarRocks/starrocks/pull/15969)
- 优化了压缩框架的 CPU 使用率。[#11747](https://github.com/StarRocks/starrocks/pull/11747)
- 支持对缺失版本的 tablet 进行累积压缩。[#17030](https://github.com/StarRocks/starrocks/pull/17030)

### Bug 修复

已修复以下 bug：

- 当指定无效的 DATE 值时，会创建过多的动态分区。[#17966](https://github.com/StarRocks/starrocks/pull/17966)
- 无法连接默认 HTTPS 端口的 Elasticsearch 外部表。[#13726](https://github.com/StarRocks/starrocks/pull/13726)
- BE 在事务超时后无法取消流加载事务。[#15738](https://github.com/StarRocks/starrocks/pull/15738)
- 在单个 BE 上进行本地 Shuffle 聚合时返回错误的查询结果。[#17845](https://github.com/StarRocks/starrocks/pull/17845)
- 查询可能因“wait_for_version version: failed: apply stopped”错误消息而失败。[#17848](https://github.com/StarRocks/starrocks/pull/17850)
- 因 OLAP 扫描桶表达式未正确清除而返回错误查询结果。[#17666](https://github.com/StarRocks/starrocks/pull/17666)
- Co-locate Group 中动态分区表的 bucket 数量无法修改，并返回错误消息。[#17418](https://github.com/StarRocks/starrocks/pull/17418)
- 当连接到非 Leader FE 节点并发送 SQL 语句 `USE <catalog_name>.<database_name>` 时，非 Leader FE 节点将 SQL 语句转发到 Leader FE 节点，不包括 `<catalog_name>`，导致 Leader FE 节点选择使用 `default_catalog`，最终无法找到指定的数据库。[#17302](https://github.com/StarRocks/starrocks/pull/17302)
- 在重写之前对字典映射的检查逻辑不正确。[#17405](https://github.com/StarRocks/starrocks/pull/17405)
- 如果 FE 偶尔发送心跳到 BE，且心跳连接超时，FE 将视 BE 为不可用，导致 BE 上的事务失败。[#16386](https://github.com/StarRocks/starrocks/pull/16386)
- `get_applied_rowsets` 对 follower FE 上新克隆的 tablet 执行的查询失败。[#17192](https://github.com/StarRocks/starrocks/pull/17192)
- 在 follower FE 上执行 `SET variable = default` 时引起 NPE。[#17549](https://github.com/StarRocks/starrocks/pull/17549)
- 投影中的表达式未被字典重写。[#17558](https://github.com/StarRocks/starrocks/pull/17558)
- 基于周基础的动态分区创建逻辑不正确。[#17163](https://github.com/StarRocks/starrocks/pull/17163)
- 在本地 Shuffle 中返回错误查询结果。[#17130](https://github.com/StarRocks/starrocks/pull/17130)
- 增量克隆可能失败。[#16930](https://github.com/StarRocks/starrocks/pull/16930)
- 在某些情况下，CBO 可能使用不正确的逻辑来比较两个运算符是否等效。[#17199](https://github.com/StarRocks/starrocks/pull/17199) [#17227](https://github.com/StarRocks/starrocks/pull/17227)
- 由于 JuiceFS 架构未经检查和解析，无法访问 JuiceFS。[#16940](https://github.com/StarRocks/starrocks/pull/16940)

## 2.4.3

发布日期：2023年1月19日

### 改进

- StarRocks 在分析过程中检查相应的数据库和表是否存在，以防止 NPE。[#14467](https://github.com/StarRocks/starrocks/pull/14467)
- 不支持的数据类型的列不会被物化为外部表的查询结果。[#13305](https://github.com/StarRocks/starrocks/pull/13305)
- 为 FE 启动脚本 **start_fe.sh** 增加 Java 版本检查。[#14333](https://github.com/StarRocks/starrocks/pull/14333)

### Bug 修复

已修复以下 bug：

- 当未设置超时时，流加载可能会失败。[#16241](https://github.com/StarRocks/starrocks/pull/16241)
- 当内存使用率高时，bRPC发送会崩溃。[#16046](https://github.com/StarRocks/starrocks/issues/16046)
- StarRocks无法从早期版本的StarRocks实例中的外部表中加载数据。[#16130](https://github.com/StarRocks/starrocks/pull/16130)
- 材质化视图刷新失败可能导致内存泄漏。[#16041](https://github.com/StarRocks/starrocks/pull/16041)
- 模式更改在发布阶段挂起。[#14148](https://github.com/StarRocks/starrocks/issues/14148)
- 材质化视图QeProcessorImpl问题导致的内存泄漏。[#15699](https://github.com/StarRocks/starrocks/pull/15699)
- “limit”查询结果不一致。[#13574](https://github.com/StarRocks/starrocks/pull/13574)
- 由INSERT导致的内存泄漏。[#14718](https://github.com/StarRocks/starrocks/pull/14718)
- 主键表执行Tablet Migration。[#13720](https://github.com/StarRocks/starrocks/pull/13720)
- 经纪人Kerberos票据在经纪人加载期间超时。[#16149](https://github.com/StarRocks/starrocks/pull/16149)
- 在表的视图中，“nullable”信息推断不正确。[#15744](https://github.com/StarRocks/starrocks/pull/15744)

### 行为更改

- 修改了Thrift Listen的默认积压数为`1024`。[#13911](https://github.com/StarRocks/starrocks/pull/13911)
- 添加了SQL模式`FORBID_INVALID_DATES`。该SQL模式默认为禁用状态。启用后，StarRocks验证DATE类型的输入，并在输入无效时返回错误。[#14143](https://github.com/StarRocks/starrocks/pull/14143)

## 2.4.2

发布日期：2022年12月14日

### 改进

- 当存在大量桶时，优化了Bucket Hint的性能。[#13142](https://github.com/StarRocks/starrocks/pull/13142)

### Bug修复

已修复以下错误：

- 刷新主键索引可能导致BE崩溃。[#14857](https://github.com/StarRocks/starrocks/pull/14857) [#14819](https://github.com/StarRocks/starrocks/pull/14819)
- 无法正确识别材质化视图类型通过`SHOW FULL TABLES`。[#13954](https://github.com/StarRocks/starrocks/pull/13954)
- 将StarRocks v2.2升级到v2.4可能导致BE崩溃。[#13795](https://github.com/StarRocks/starrocks/pull/13795)
- 经纪人加载可能导致BE崩溃。[#13973](https://github.com/StarRocks/starrocks/pull/13973)
- 会话变量`statistic_collect_parallel`不会生效。[#14352](https://github.com/StarRocks/starrocks/pull/14352)
- INSERT INTO可能会导致BE崩溃。[#14818](https://github.com/StarRocks/starrocks/pull/14818)
- JAVA UDF可能会导致BE崩溃。[#13947](https://github.com/StarRocks/starrocks/pull/13947)
- 部分更新期间克隆副本可能导致BE崩溃并且无法重新启动。[#13683](https://github.com/StarRocks/starrocks/pull/13683)
- Co-located Join可能不会生效。[#13561](https://github.com/StarRocks/starrocks/pull/13561)

### 行为更改

- 将会话变量`query_timeout`限制为上限`259200`和下限`1`。
- 已弃用FE参数`default_storage_medium`。表的存储介质将由系统自动推断。[#14394](https://github.com/StarRocks/starrocks/pull/14394)

## 2.4.1

发布日期：2022年11月14日

### 新功能

- 支持非等值连接 - LEFT SEMI JOIN 和 ANTI JOIN。优化了JOIN功能。[#13019](https://github.com/StarRocks/starrocks/pull/13019)

### 改进

- 支持`HeartbeatResponse`中的`aliveStatus`属性。`aliveStatus`指示集群中的节点是否存活。进一步优化了判断`aliveStatus`的机制。[#12713](https://github.com/StarRocks/starrocks/pull/12713)

- 优化了Routine Load的错误信息。[#12155](https://github.com/StarRocks/starrocks/pull/12155)

### Bug修复

- 从v2.4.0RC升级到v2.4后，BE崩溃。[#13128](https://github.com/StarRocks/starrocks/pull/13128)

- 迟到的材质化导致数据湖上的查询结果不正确。[#13133](https://github.com/StarRocks/starrocks/pull/13133)

- `get_json_int`函数抛出异常。[#12997](https://github.com/StarRocks/starrocks/pull/12997)

- 从具有持久索引的主键表中删除数据后，数据可能不一致。[#12719](https://github.com/StarRocks/starrocks/pull/12719)

- 在主键表进行压缩期间，BE可能会崩溃。[#12914](https://github.com/StarRocks/starrocks/pull/12914)

- 当其输入包含空字符串时，`json_object`函数返回不正确的结果。[#13030](https://github.com/StarRocks/starrocks/issues/13030)

- BE由于`RuntimeFilter`崩溃。[#12807](https://github.com/StarRocks/starrocks/pull/12807)

- FE由于CBO中过度递归计算而挂起。[#12788](https://github.com/StarRocks/starrocks/pull/12788)

- BE在正常退出时可能崩溃或报告错误。[#12852](https://github.com/StarRocks/starrocks/pull/12852)

- 从表中删除数据后，压缩会崩溃。[#12907](https://github.com/StarRocks/starrocks/pull/12907)

- 由于不正确的机制，数据可能不一致。OLAP外部表元数据同步。[#12368](https://github.com/StarRocks/starrocks/pull/12368)

- 当一个BE崩溃时，其他BE可能会执行相关查询直到超时。[#12954](https://github.com/StarRocks/starrocks/pull/12954)

### 行为更改

- 当解析Hive外部表失败时，StarRocks会抛出错误消息，而不是将相关列转换为NULL列。[#12382](https://github.com/StarRocks/starrocks/pull/12382)

## 2.4.0

发布日期：2022年10月20日

### 新功能

- 支持基于多个基表创建异步材质化视图，以加速具有JOIN操作的查询。异步材质化视图支持所有[table types](../table_design/table_types/table_types.md)。更多信息，请参阅[Materialized View](../using_starrocks/Materialized_view.md)。

- 支持通过INSERT OVERWRITE覆盖数据。更多信息，请参阅[使用INSERT加载数据](../loading/InsertInto.md)。

- [预览]提供了可以进行水平扩展的无状态Compute Nodes (CN)。您可以使用StarRocks Operator将CN部署到您的Kubernetes (K8s)集群，以实现自动水平扩展。更多信息，请参阅[使用StarRocks Operator在Kubernetes上部署和管理CN](../deployment/sr_operator.md)。

- 外部连接支持非等值连接，其中连接项由包括`<`、`<=`、`>`、`>=`和`<>`的比较运算符相关联。更多信息，请参阅[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)。

- 支持创建Iceberg和Hudi目录，允许直接查询来自Apache Iceberg和Apache Hudi的数据。更多信息，请参阅[Iceberg catalog](../data_source/catalog/iceberg_catalog.md)和[Hudi catalog](../data_source/catalog/hudi_catalog.md)。

- 支持以CSV格式从Apache Hive™表中查询ARRAY类型列。更多信息，请参阅[外部表](../data_source/External_table.md)。

- 支持通过DESC查看外部数据的模式。更多信息，请参阅[DESC](../sql-reference/sql-statements/Utility/DESCRIBE.md)。

- 支持通过GRANT向用户授予特定角色或IMPERSONATE权限，并通过REVOKE来撤销，同时支持使用IMPERSONATE权限执行SQL语句通过EXECUTE AS。有关更多信息，请参阅[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)，[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md)和[EXECUTE AS](../sql-reference/sql-statements/account-management/EXECUTE_AS.md)。

- 支持FDQN访问：现在可以使用域名或主机名和端口的组合作为BE或FE节点的唯一标识，以防止由于IP地址更改而导致的访问失败。有关更多信息，请参阅[启用FDQN访问](../administration/enable_fqdn.md)。

- flink-connector-starrocks支持主键表部分更新。有关更多信息，请参阅[使用flink-connector-starrocks加载数据](../loading/Flink-connector-starrocks.md)。

- 提供以下新函数：

  - array_contains_all：检查特定数组是否是另一个数组的子集。有关更多信息，请参阅[array_contains_all](../sql-reference/sql-functions/array-functions/array_contains_all.md)。
  - percentile_cont：使用线性插值计算百分位值。有关更多信息，请参阅[percentile_cont](../sql-reference/sql-functions/aggregate-functions/percentile_cont.md)。

### 改进

- 主键表支持将VARCHAR类型主键索引刷新到磁盘。从版本2.4.0开始，无论是否打开持久性主键索引，主键表都支持相同的主键索引数据类型。

- 优化了外部表的查询性能。

  - 支持在Parquet格式的外部表查询期间进行延迟物化，以优化涉及小规模过滤的数据湖上的查询性能。
  - 可以合并小的I/O操作以减少对数据湖查询的延迟，从而改善外部表的查询性能。

- 优化了窗口函数的性能。

- 通过支持谓词下推来优化交叉连接的性能。

- CBO统计信息中添加了直方图。进一步优化了完整统计信息收集。有关更多信息，请参阅[收集CBO统计信息](../using_starrocks/Cost_based_optimizer.md)。

- 启用了适应性多线程用于表扫描，以减少扫描性能对表格数量的依赖。因此，您可以更容易地设置存储桶的数量。有关更多信息，请参阅[确定存储桶的数量](../table_design/Data_distribution.md#how-to-determine-the-number-of-buckets)。

- 支持在Apache Hive中查询压缩的TXT文件。

- 调整了默认PageCache大小计算和内存一致性检查的机制，以避免在多实例部署期间出现OOM问题。

- 通过移除final_merge操作，将主键表的大尺寸批量加载性能提高了两倍。

- 支持流加载事务接口以实现从外部系统（如Apache Flink®和Apache Kafka®）加载数据的两阶段提交（2PC），从而提高高并发流加载的性能。

- 函数：

  - 您可以在一条语句中使用多个COUNT(DISTINCT)。有关更多信息，请参阅[count](../sql-reference/sql-functions/aggregate-functions/count.md)。
  - 窗口函数min()和max()支持滑动窗口。有关更多信息，请参阅[窗口函数](../sql-reference/sql-functions/Window_function.md)。
  - 优化了window_funnel函数的性能。有关更多信息，请参阅[window_funnel](../sql-reference/sql-functions/aggregate-functions/window_funnel.md)。

### Bug修复

已修复以下错误：

- DESC返回的DECIMAL数据类型与CREATE TABLE语句中指定的不同。[#7309](https://github.com/StarRocks/starrocks/pull/7309)

- 会影响FE稳定性的FE元数据管理问题。[#6685](https://github.com/StarRocks/starrocks/pull/6685) [#9445](https://github.com/StarRocks/starrocks/pull/9445) [#7974](https://github.com/StarRocks/starrocks/pull/7974) [#7455](https://github.com/StarRocks/starrocks/pull/7455)

- 与数据加载相关的问题：

  - 当指定ARRAY列时，Load失败。[#9158](https://github.com/StarRocks/starrocks/pull/9158)
  - 通过Broker Load将数据加载到非重复键表后，复制品不一致。[#8714](https://github.com/StarRocks/starrocks/pull/8714)
  - 执行ALTER ROUTINE LOAD会引发NPE。[#7804](https://github.com/StarRocks/starrocks/pull/7804)

- 数据湖分析相关问题：

  - 在Hive外部表的Parquet数据上执行的查询失败。[#7413](https://github.com/StarRocks/starrocks/pull/7413) [#7482](https://github.com/StarRocks/starrocks/pull/7482) [#7624](https://github.com/StarRocks/starrocks/pull/7624)
  - 在具有复杂数据类型的Apache Iceberg表上执行带有`limit`子句的查询返回不正确的结果。[#9226](https://github.com/StarRocks/starrocks/pull/9226)
  - 在Leader FE和Follower FE节点之间的元数据不一致。[#11215](https://github.com/StarRocks/starrocks/pull/11215)

  

- 当BITMAP数据大小超过2GB时，BE崩溃。[#11178](https://github.com/StarRocks/starrocks/pull/11178)

### 行为更改

- 默认情况下启用了Page Cache（"disable_storage_page_cache" = "false"）。默认的缓存大小(`storage_page_cache_limit`)为系统内存的20%。
- 默认情况下启用了CBO。不建议使用会话变量`enable_cbo`。
- 默认情况下启用了矢量化引擎。不建议使用会话变量`vectorized_engine_enable`。

### 其他

- 宣布Resource Group的稳定发布。
- 宣布JSON数据类型及其相关函数的稳定发布。