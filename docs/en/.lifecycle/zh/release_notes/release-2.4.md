---
displayed_sidebar: English
---

# StarRocks 2.4 版本

## 2.4.5

发布日期：2023年4月21日

### 改进

- 禁止使用List Partition语法，因为在升级元数据时可能会导致错误。[#15401](https://github.com/StarRocks/starrocks/pull/15401)
- 支持在具体化视图中使用BITMAP、HLL和PERCENTILE类型。[#15731](https://github.com/StarRocks/starrocks/pull/15731)
- 优化了`storage_medium`的推断。当BE同时使用SSD和HDD作为存储设备时，如果指定了`storage_cooldown_time`属性，StarRocks将`storage_medium`设置为`SSD`。否则，StarRocks将`storage_medium`设置为`HDD`。[#18649](https://github.com/StarRocks/starrocks/pull/18649)
- 优化了线程转储的准确性。[#16748](https://github.com/StarRocks/starrocks/pull/16748)
- 在加载之前通过触发元数据压缩来优化加载效率。[#19347](https://github.com/StarRocks/starrocks/pull/19347)
- 优化了Stream Load Planner的超时设置。[#18992](https://github.com/StarRocks/starrocks/pull/18992/files)
- 通过禁止从值列收集统计信息来优化唯一键表的性能。[#19563](https://github.com/StarRocks/starrocks/pull/19563)

### Bug修复

修复了以下问题：

- 在CREATE TABLE中使用不受支持的数据类型时将返回NPE。[#20999](https://github.com/StarRocks/starrocks/issues/20999)
- 使用短路广播连接的查询将返回错误的结果。[#20952](https://github.com/StarRocks/starrocks/issues/20952)
- 由于错误的数据截断逻辑导致的磁盘占用问题。[#20590](https://github.com/StarRocks/starrocks/pull/20590)
- 无法安装或删除AuditLoader插件。[#20468](https://github.com/StarRocks/starrocks/issues/20468)
- 如果在调度平板时引发异常，同一批次中的其他平板将永远不会被调度。[#20681](https://github.com/StarRocks/starrocks/pull/20681)
- 在创建同步具体化视图时使用不受支持的SQL函数将返回未知错误。[#20348](https://github.com/StarRocks/starrocks/issues/20348)
- 多个COUNT DISTINCT计算被错误地重写。[#19714](https://github.com/StarRocks/starrocks/pull/19714)
- 在处于压缩状态的平板上的查询将返回错误的结果。[#20084](https://github.com/StarRocks/starrocks/issues/20084)
- 具有聚合操作的查询将返回错误的结果。[#19725](https://github.com/StarRocks/starrocks/issues/19725)
- 将NULL Parquet数据加载到非NULL列中时，不会返回错误消息。[#19885](https://github.com/StarRocks/starrocks/pull/19885)
- 当资源组的并发限制持续达到时，查询并发度指标下降缓慢。[#19363](https://github.com/StarRocks/starrocks/pull/19363)
- 在重放`InsertOverwriteJob`状态更改日志时，FE无法启动。[#19061](https://github.com/StarRocks/starrocks/issues/19061)
- 主键表死锁。[#18488](https://github.com/StarRocks/starrocks/pull/18488)
- 对于共置表，可以使用类似`ADMIN SET REPLICA STATUS PROPERTIES ("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");`的语句手动指定副本状态为`bad`。如果BE的数量小于或等于副本的数量，则无法修复损坏的副本。[#17876](https://github.com/StarRocks/starrocks/issues/17876)
- 由ARRAY相关函数引起的问题。[#18556](https://github.com/StarRocks/starrocks/pull/18556)

## 2.4.4

发布日期：2023年2月22日

### 改进

- 支持快速取消加载。[#15514](https://github.com/StarRocks/starrocks/pull/15514) [#15398](https://github.com/StarRocks/starrocks/pull/15398) [#15969](https://github.com/StarRocks/starrocks/pull/15969)
- 优化了压缩框架的CPU使用率。[#11747](https://github.com/StarRocks/starrocks/pull/11747)
- 支持在缺少版本的平板上进行累积压缩。[#17030](https://github.com/StarRocks/starrocks/pull/17030)

### Bug修复

修复了以下问题：

- 当指定无效的DATE值时，将创建过多的动态分区。[#17966](https://github.com/StarRocks/starrocks/pull/17966)
- 使用默认的HTTPS端口连接Elasticsearch外部表失败。[#13726](https://github.com/StarRocks/starrocks/pull/13726)
- BE在事务超时后无法取消Stream Load事务。[#15738](https://github.com/StarRocks/starrocks/pull/15738)
- 从单个BE上的本地随机聚合返回错误的查询结果。[#17845](https://github.com/StarRocks/starrocks/pull/17845)
- 查询可能会失败，并显示错误消息“wait_for_version版本：失败：应用已停止”。[#17848](https://github.com/StarRocks/starrocks/pull/17850)
- 由于未正确清除OLAP扫描存储桶表达式，因此返回了错误的查询结果。[#17666](https://github.com/StarRocks/starrocks/pull/17666)
- 无法修改共置组中动态分区表的桶数，并返回错误信息。[#17418](https://github.com/StarRocks/starrocks/pull/17418)
- 当您连接到非Leader FE节点并发送SQL语句时，非Leader FE节点会将SQL语句转发给Leader FE节点，但最终无法找到指定的数据库。[#17302](https://github.com/StarRocks/starrocks/pull/17302)
- 重写前的字典映射检查逻辑不正确。[#17405](https://github.com/StarRocks/starrocks/pull/17405)
- 如果FE偶尔向BE发送心跳，并且心跳连接超时，FE会认为BE不可用，导致BE上的事务失败。[#16386](https://github.com/StarRocks/starrocks/pull/16386)
- `get_applied_rowsets`在跟随者FE上新克隆的平板上查询失败。[#17192](https://github.com/StarRocks/starrocks/pull/17192)
- NPE由`SET variable = default`在跟随者FE上执行引起。[#17549](https://github.com/StarRocks/starrocks/pull/17549)
- 投影中的表达式不会被字典重写。[#17558](https://github.com/StarRocks/starrocks/pull/17558)
- 每周创建动态分区的逻辑不正确。[#17163](https://github.com/StarRocks/starrocks/pull/17163)
- 从本地随机播放返回错误的查询结果。[#17130](https://github.com/StarRocks/starrocks/pull/17130)
- 增量克隆可能会失败。[#16930](https://github.com/StarRocks/starrocks/pull/16930)
- 在某些情况下，CBO可能会使用不正确的逻辑来比较两个运算符是否等效。[#17199](https://github.com/StarRocks/starrocks/pull/17199) [#17227](https://github.com/StarRocks/starrocks/pull/17227)
- 由于JuiceFS模式未经正确检查和解析，无法访问JuiceFS。[#16940](https://github.com/StarRocks/starrocks/pull/16940)

## 2.4.3

发布日期：2023年1月19日

### 改进

- StarRocks在Analyze时会检查对应的数据库和表是否存在，以防止NPE。[#14467](https://github.com/StarRocks/starrocks/pull/14467)
- 对于外部表的查询，不会对不受支持的数据类型的列进行具体化。[#13305](https://github.com/StarRocks/starrocks/pull/13305)
- 为FE启动脚本start_fe.sh添加了Java版本检查。[#14333](https://github.com/StarRocks/starrocks/pull/14333)

### Bug修复

修复了以下问题：

- 未设置超时时，流加载可能会失败。[#16241](https://github.com/StarRocks/starrocks/pull/16241)
- bRPC Send在内存使用率较高时崩溃。[#16046](https://github.com/StarRocks/starrocks/issues/16046)
- 无法从早期版本的StarRocks实例加载外部表中的数据。[#16130](https://github.com/StarRocks/starrocks/pull/16130)
- 具体化视图刷新失败可能会导致内存泄漏。[#16041](https://github.com/StarRocks/starrocks/pull/16041)
- 架构更改在“发布”阶段挂起。[#14148](https://github.com/StarRocks/starrocks/issues/14148)
- 具体化视图QeProcessorImpl问题导致的内存泄漏。[#15699](https://github.com/StarRocks/starrocks/pull/15699)
- 查询结果limit不一致。[#13574](https://github.com/StarRocks/starrocks/pull/13574)
- INSERT导致的内存泄漏。[#14718](https://github.com/StarRocks/starrocks/pull/14718)
- 主键表执行平板迁移。[#13720](https://github.com/StarRocks/starrocks/pull/13720)
- Broker Kerberos票证在Broker Load期间超时。[#16149](https://github.com/StarRocks/starrocks/pull/16149)
- 在表的视图中错误地推断出`nullable`信息。[#15744](https://github.com/StarRocks/starrocks/pull/15744)

### 行为变更

- 将Thrift Listen的默认积压工作修改为`1024`。[#13911](https://github.com/StarRocks/starrocks/pull/13911)
- 添加了SQL模式`FORBID_INVALID_DATES`。默认情况下，此SQL模式处于禁用状态。启用后，StarRocks会验证DATE类型的输入，当输入无效时返回错误。[#14143](https://github.com/StarRocks/starrocks/pull/14143)

## 2.4.2

发布日期：2022年12月14日

### 改进

- 优化了在存在大量Bucket时的Bucket Hint性能。[#13142](https://github.com/StarRocks/starrocks/pull/13142)

### Bug修复

修复了以下问题：

- 刷新主键索引可能会导致BE崩溃。[#14857](https://github.com/StarRocks/starrocks/pull/14857) [#14819](https://github.com/StarRocks/starrocks/pull/14819)
- 无法正确识别具体化视图类型的`SHOW FULL TABLES`。[#13954](https://github.com/StarRocks/starrocks/pull/13954)
- 从StarRocks v2.2升级到v2.4可能会导致BE崩溃。[#13795](https://github.com/StarRocks/starrocks/pull/13795)
- 代理负载可能会导致BE崩溃。[#13973](https://github.com/StarRocks/starrocks/pull/13973)
- 会话变量 `statistic_collect_parallel` 无法生效。 [#14352](https://github.com/StarRocks/starrocks/pull/14352)
- 执行 INSERT INTO 可能导致 BE 崩溃。 [#14818](https://github.com/StarRocks/starrocks/pull/14818)
- JAVA UDF 可能导致 BE 崩溃。 [#13947](https://github.com/StarRocks/starrocks/pull/13947)
- 在部分更新期间克隆副本可能导致 BE 崩溃，并且无法重新启动。 [#13683](https://github.com/StarRocks/starrocks/pull/13683)
- Colocated Join 可能不会生效。 [#13561](https://github.com/StarRocks/starrocks/pull/13561)

### 行为更改

- 对会话变量 `query_timeout` 进行了约束，上限为 `259200`，下限为 `1`。
- 弃用了 FE 参数 `default_storage_medium`。表的存储介质由系统自动推断。 [#14394](https://github.com/StarRocks/starrocks/pull/14394)

## 2.4.1

发布日期：2022年11月14日

### 新特性

- 支持非等值连接 - LEFT SEMI JOIN 和 ANTI JOIN。优化了 JOIN 功能。 [#13019](https://github.com/StarRocks/starrocks/pull/13019)

### 改进

- 支持 `HeartbeatResponse` 中的 `aliveStatus` 属性。`aliveStatus` 指示集群中节点的活动状态。进一步优化了判断 `aliveStatus` 的机制。 [#12713](https://github.com/StarRocks/starrocks/pull/12713)

- 优化了 Routine Load 的错误消息。 [#12155](https://github.com/StarRocks/starrocks/pull/12155)

### Bug 修复

- 从 v2.4.0RC 升级到 v2.4.0 后，BE 崩溃。 [#13128](https://github.com/StarRocks/starrocks/pull/13128)

- 晚期具体化导致数据湖上的查询结果不正确。 [#13133](https://github.com/StarRocks/starrocks/pull/13133)

- get_json_int 函数引发异常。 [#12997](https://github.com/StarRocks/starrocks/pull/12997)

- 从具有持久索引的 PRIMARY KEY 表中删除数据后，数据可能不一致。[#12719](https://github.com/StarRocks/starrocks/pull/12719)

- 在对 PRIMARY KEY 表进行压缩期间，BE 可能会崩溃。 [#12914](https://github.com/StarRocks/starrocks/pull/12914)

- 当 json_object 函数的输入包含空字符串时，该函数返回不正确的结果。 [#13030](https://github.com/StarRocks/starrocks/issues/13030)

- BE 由于 `RuntimeFilter` 而崩溃。 [#12807](https://github.com/StarRocks/starrocks/pull/12807)

- FE 由于 CBO 中的递归计算过多而挂起。 [#12788](https://github.com/StarRocks/starrocks/pull/12788)

- BE 可能在正常退出时崩溃或报告错误。 [#12852](https://github.com/StarRocks/starrocks/pull/12852)

- 从表中删除数据后，添加了新列，压缩会崩溃。 [#12907](https://github.com/StarRocks/starrocks/pull/12907)

- 由于 OLAP 外部表元数据同步机制不正确，数据可能不一致。 [#12368](https://github.com/StarRocks/starrocks/pull/12368)

- 当一个 BE 崩溃时，其他 BE 可能执行相关查询，直到超时。 [#12954](https://github.com/StarRocks/starrocks/pull/12954)

### 行为更改

- 当解析 Hive 外部表失败时，StarRocks 会抛出错误消息，而不是将相关列转换为 NULL 列。 [#12382](https://github.com/StarRocks/starrocks/pull/12382)

## 2.4.0

发布日期：2022年10月20日

### 新特性

- 支持基于多个基表创建异步物化视图，以加速带有 JOIN 操作的查询。异步物化视图支持所有 [表类型](../table_design/table_types/table_types.md)。有关详细信息，请参阅 [物化视图](../using_starrocks/Materialized_view.md)。

- 支持通过 INSERT OVERWRITE 覆盖数据。有关详细信息，请参阅 [使用 INSERT 加载数据](../loading/InsertInto.md)。

- [预览] 提供可水平扩展的无状态计算节点 （CN）。您可以使用 StarRocks Operator 将 CN 部署到 Kubernetes （K8s） 集群中，实现自动水平扩展。更多信息，请参见[使用 StarRocks Operator 在 Kubernetes 上部署和管理 CN](../deployment/sr_operator.md)。

- 外部联接支持非等价联接，其中联接项通过比较运算符（包括 `<`、 `<=`, `>` `>=` 和 `<>` ）进行关联。有关详细信息，请参阅 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)。

- 支持创建 Iceberg 目录和 Hudi 目录，允许直接查询来自 Apache Iceberg 和 Apache Hudi 的数据。有关详细信息，请参阅 [Iceberg 目录](../data_source/catalog/iceberg_catalog.md)和 [Hudi 目录](../data_source/catalog/hudi_catalog.md)。

- 支持以 CSV 格式查询 Apache Hive™ 表中的 ARRAY 类型列。有关详细信息，请参阅 [外部表](../data_source/External_table.md)。

- 支持通过 DESC 查看外部数据的 Schema。有关详细信息，请参阅 [DESC](../sql-reference/sql-statements/Utility/DESCRIBE.md)。

- 支持通过 GRANT 向用户授予特定角色或 IMPERSONATE 权限，并通过 REVOKE 撤销该权限，支持通过 EXECUTE AS 执行具有 IMPERSONATE 权限的 SQL 语句。有关详细信息，请参阅 [GRANT、](../sql-reference/sql-statements/account-management/GRANT.md)[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md) 和 [EXECUTE AS](../sql-reference/sql-statements/account-management/EXECUTE_AS.md)。

- 支持 FDQN 访问：现在您可以使用域名或主机名和端口的组合作为 BE 或 FE 节点的唯一标识。这样可以防止因更改 IP 地址而导致的访问失败。有关详细信息，请参阅 [启用 FQDN 访问](../administration/enable_fqdn.md)。

- flink-connector-starrocks 支持主键表局部更新。更多信息，请参见 [使用 flink-connector-starrocks 加载数据](../loading/Flink-connector-starrocks.md)。

- 提供以下新功能：

  - array_contains_all：检查特定数组是否是另一个数组的子集。有关详细信息，请参阅 [array_contains_all](../sql-reference/sql-functions/array-functions/array_contains_all.md)。
  - percentile_cont：使用线性插值计算百分位值。有关详细信息，请参阅 [percentile_cont](../sql-reference/sql-functions/aggregate-functions/percentile_cont.md)。

### 改进

- 主键表支持将 VARCHAR 类型的主键索引刷新到磁盘。从版本 2.4.0 开始，无论是否打开持久性主键索引，主键表都支持相同的主键索引数据类型。

- 优化了外部表的查询性能。

  - 支持对 Parquet 格式的外部表进行查询时的后期物化，以优化涉及小规模过滤的数据湖上的查询性能。
  - 支持合并小的 I/O 操作，减少查询数据湖的延迟，从而提高对外部表的查询性能。

- 优化了窗口函数的性能。

- 通过支持谓词下推优化了 Cross Join 的性能。

- 将直方图添加到 CBO 统计信息中。进一步优化了全量统计采集。有关详细信息，请参阅 [收集 CBO 统计信息](../using_starrocks/Cost_based_optimizer.md)。

- 为平板电脑扫描启用自适应多线程，以减少扫描性能对平板电脑数量的依赖性。因此，您可以更轻松地设置存储桶的数量。有关更多信息，请参阅[确定存储桶的数量](../table_design/Data_distribution.md#how-to-determine-the-number-of-buckets)。

- 支持在 Apache Hive 中查询压缩的 TXT 文件。

- 调整了默认的 PageCache 大小计算和内存一致性检查机制，以避免在多实例部署期间出现 OOM 问题。

- 通过删除 final_merge 操作，将主键表上的大型批量加载性能提高了多达 2 倍。

- 支持流加载事务接口，对从外部系统（如 Apache Flink®、Apache Kafka®）加载数据的事务实现两阶段提交（2PC），提高高并发流加载的性能。

- 功能：

  - 您可以在一个语句中使用多个 COUNT(DISTINCT)。有关详细信息，请参阅 [count](../sql-reference/sql-functions/aggregate-functions/count.md)。
  - 窗口函数 min() 和 max() 支持滑动窗口。有关详细信息，请参阅 [窗口函数](../sql-reference/sql-functions/Window_function.md)。
  - 优化了 window_funnel 功能的性能。有关详细信息，请参阅 [window_funnel](../sql-reference/sql-functions/aggregate-functions/window_funnel.md)。

### Bug 修复

修复了以下错误：

- DESC 返回的 DECIMAL 数据类型与 CREATE TABLE 语句中指定的数据类型不同。 [#7309](https://github.com/StarRocks/starrocks/pull/7309)

- 影响 FE 稳定性的 FE 元数据管理问题。 #[6685 #9445 #](https://github.com/StarRocks/starrocks/pull/6685)7974[ ](https://github.com/StarRocks/starrocks/pull/9445)#7455[ ](https://github.com/StarRocks/starrocks/pull/7974)[ ](https://github.com/StarRocks/starrocks/pull/7455)

- 与数据加载相关的问题：

  - 指定 ARRAY 列时，Broke Load 失败。 [#9158](https://github.com/StarRocks/starrocks/pull/9158)
  - 通过 Broker Load 将数据加载到非重复键表后，副本不一致。 [#8714](https://github.com/StarRocks/starrocks/pull/8714)
  - 执行 ALTER ROUTINE LOAD 引发 NPE。 [#7804](https://github.com/StarRocks/starrocks/pull/7804)

- 与数据湖分析相关的问题：

  - 对 Hive 外部表中的 Parquet 数据的查询失败。 #7413 [  #](https://github.com/StarRocks/starrocks/pull/7413)7482[ ](https://github.com/StarRocks/starrocks/pull/7482)#7624[ ](https://github.com/StarRocks/starrocks/pull/7624)
  - 对于对 Elasticsearch 外部表具有 `limit` 子句的查询，将返回不正确的结果。[#9226](https://github.com/StarRocks/starrocks/pull/9226)
  - 在对具有复杂数据类型的 Apache Iceberg 表进行查询时引发未知错误。 [#11298](https://github.com/StarRocks/starrocks/pull/11298)

- Leader FE 和 Follower FE 节点之间的元数据不一致。 [#11215](https://github.com/StarRocks/starrocks/pull/11215)

- 当 BITMAP 数据的大小超过 2 GB 时，BE 崩溃。 [#11178](https://github.com/StarRocks/starrocks/pull/11178)

### 行为更改


- 默认情况下，页面缓存已启用（“disable_storage_page_cache” = “false”）。默认缓存大小 (`storage_page_cache_limit`) 为系统内存的20%。
- 默认情况下，CBO已启用。已弃用会话变量 `enable_cbo`。
- 默认情况下，矢量化引擎已启用。已弃用会话变量 `vectorized_engine_enable`。

### 其他

- 宣布资源组稳定版本的发布。
- 宣布JSON数据类型及其相关函数的稳定版本发布。