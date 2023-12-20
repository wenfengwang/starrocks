---
displayed_sidebar: English
---

# StarRocks版本2.4

## 2.4.5

发布日期：2023年4月21日

### 改进

- 禁止使用List Partition语法，因为它可能会导致升级元数据时出现错误。[#15401](https://github.com/StarRocks/starrocks/pull/15401)
- 支持物化视图的BITMAP、HLL和PERCENTILE类型。[#15731](https://github.com/StarRocks/starrocks/pull/15731)
- 优化了`storage_medium`的推断。当BE使用SSD和HDD作为存储设备时，如果指定了属性`storage_cooldown_time`，StarRocks将`storage_medium`设置为`SSD`。否则，StarRocks将`storage_medium`设置为`HDD`。[#18649](https://github.com/StarRocks/starrocks/pull/18649)
- 优化了线程转储的准确性。[#16748](https://github.com/StarRocks/starrocks/pull/16748)
- 通过在加载前触发元数据压缩来优化加载效率。[#19347](https://github.com/StarRocks/starrocks/pull/19347)
- 优化了Stream Load规划器的超时。[#18992](https://github.com/StarRocks/starrocks/pull/18992/files)
- 通过禁止从值列收集统计信息来优化Unique key表的性能。[#19563](https://github.com/StarRocks/starrocks/pull/19563)

### Bug修复

修复了以下错误：

- 当在CREATE TABLE中使用不支持的数据类型时，返回NPE。[#20999](https://github.com/StarRocks/starrocks/issues/20999)
- 使用带有短路的Broadcast Join的查询返回错误结果。[#20952](https://github.com/StarRocks/starrocks/issues/20952)
- 由错误的数据截断逻辑引起的磁盘占用问题。[#20590](https://github.com/StarRocks/starrocks/pull/20590)
- AuditLoader插件既不能安装也不能删除。[#20468](https://github.com/StarRocks/starrocks/issues/20468)
- 如果在调度tablet时抛出异常，同一批中的其他tablet将永远不会被调度。[#20681](https://github.com/StarRocks/starrocks/pull/20681)
- 在创建同步物化视图时使用不支持的SQL函数返回未知错误。[#20348](https://github.com/StarRocks/starrocks/issues/20348)
- 多个COUNT DISTINCT计算被错误地重写。[#19714](https://github.com/StarRocks/starrocks/pull/19714)
- 在压缩中的tablet上的查询返回错误结果。[#20084](https://github.com/StarRocks/starrocks/issues/20084)
- 带有聚合的查询返回错误结果。[#19725](https://github.com/StarRocks/starrocks/issues/19725)
- 将NULL Parquet数据加载到NOT NULL列时没有返回错误消息。[#19885](https://github.com/StarRocks/starrocks/pull/19885)
- 当资源组的并发限制连续达到时，查询并发度指标下降缓慢。[#19363](https://github.com/StarRocks/starrocks/pull/19363)
- FE在重放`InsertOverwriteJob`状态更改日志时无法启动。[#19061](https://github.com/StarRocks/starrocks/issues/19061)
- Primary Key表死锁。[#18488](https://github.com/StarRocks/starrocks/pull/18488)
- 对于Colocation表，可以使用如`ADMIN SET REPLICA STATUS PROPERTIES ("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");`的语句手动指定副本状态为`bad`。如果BE的数量小于或等于副本的数量，则无法修复损坏的副本。[#17876](https://github.com/StarRocks/starrocks/issues/17876)
- 由ARRAY相关函数引起的问题。[#18556](https://github.com/StarRocks/starrocks/pull/18556)

## 2.4.4

发布日期：2023年2月22日

### 改进

- 支持快速取消加载操作。[#15514](https://github.com/StarRocks/starrocks/pull/15514) [#15398](https://github.com/StarRocks/starrocks/pull/15398) [#15969](https://github.com/StarRocks/starrocks/pull/15969)
- 优化了压缩框架的CPU使用率。[#11747](https://github.com/StarRocks/starrocks/pull/11747)
- 支持对缺少版本的tablet进行累积压缩。[#17030](https://github.com/StarRocks/starrocks/pull/17030)

### Bug修复

修复了以下错误：

- 当指定无效的DATE值时，创建了过多的动态分区。[#17966](https://github.com/StarRocks/starrocks/pull/17966)
- 无法使用默认HTTPS端口连接到Elasticsearch外部表。[#13726](https://github.com/StarRocks/starrocks/pull/13726)
- BE在Stream Load事务超时后无法取消事务。[#15738](https://github.com/StarRocks/starrocks/pull/15738)
- 单个BE上的本地shuffle聚合返回错误的查询结果。[#17845](https://github.com/StarRocks/starrocks/pull/17845)
- 查询可能会失败，并显示错误消息“wait_for_version version: failed: apply stopped”。[#17848](https://github.com/StarRocks/starrocks/pull/17850)
- 由于OLAP扫描桶表达式未正确清除，导致查询结果错误。[#17666](https://github.com/StarRocks/starrocks/pull/17666)
- Colocate Group中动态分区表的桶数无法修改，并返回错误消息。[#17418](https://github.com/StarRocks/starrocks/pull/17418)
- 当连接到非Leader FE节点并发送SQL语句`USE <catalog_name>.<database_name>`时，非Leader FE节点将SQL语句（不包含`<catalog_name>`）转发给Leader FE节点。因此，Leader FE节点选择使用`default_catalog`，最终无法找到指定的数据库。[#17302](https://github.com/StarRocks/starrocks/pull/17302)
- 重写前的dictmapping检查逻辑不正确。[#17405](https://github.com/StarRocks/starrocks/pull/17405)
- 如果FE偶尔向BE发送心跳，并且心跳连接超时，FE会认为BE不可用，导致BE上的事务失败。[#16386](https://github.com/StarRocks/starrocks/pull/16386)
- `get_applied_rowsets`对于从属FE上新克隆的tablet的查询失败。[#17192](https://github.com/StarRocks/starrocks/pull/17192)
- 在从属FE上执行`SET variable = default`时引起的NPE。[#17549](https://github.com/StarRocks/starrocks/pull/17549)
- 投影中的表达式未被字典重写。[#17558](https://github.com/StarRocks/starrocks/pull/17558)
- 每周创建动态分区的逻辑不正确。[#17163](https://github.com/StarRocks/starrocks/pull/17163)
- 本地shuffle返回错误的查询结果。[#17130](https://github.com/StarRocks/starrocks/pull/17130)
- 增量克隆可能失败。[#16930](https://github.com/StarRocks/starrocks/pull/16930)
- 在某些情况下，CBO可能使用不正确的逻辑比较两个运算符是否等效。[#17199](https://github.com/StarRocks/starrocks/pull/17199) [#17227](https://github.com/StarRocks/starrocks/pull/17227)
- 由于未正确检查和解析JuiceFS架构，导致无法访问JuiceFS。[#16940](https://github.com/StarRocks/starrocks/pull/16940)

## 2.4.3

发布日期：2023年1月19日

### 改进

- StarRocks在执行Analyze时会检查相应的数据库和表是否存在，以防止NPE。[#14467](https://github.com/StarRocks/starrocks/pull/14467)
- 对于外部表的查询，不会物化不支持的数据类型的列。[#13305](https://github.com/StarRocks/starrocks/pull/13305)
- 为FE启动脚本**start_fe.sh**添加了Java版本检查。[#14333](https://github.com/StarRocks/starrocks/pull/14333)

### Bug修复

修复了以下错误：

- 当未设置超时时，Stream Load可能失败。[#16241](https://github.com/StarRocks/starrocks/pull/16241)
- 当内存使用率高时，bRPC Send可能崩溃。[#16046](https://github.com/StarRocks/starrocks/issues/16046)
- StarRocks无法从早期版本的StarRocks实例加载外部表中的数据。[#16130](https://github.com/StarRocks/starrocks/pull/16130)
- 物化视图刷新失败可能导致内存泄漏。[#16041](https://github.com/StarRocks/starrocks/pull/16041)
- Schema Change在发布阶段挂起。[#14148](https://github.com/StarRocks/starrocks/issues/14148)
- 由物化视图QeProcessorImpl问题引起的内存泄漏。[#15699](https://github.com/StarRocks/starrocks/pull/15699)
- 带有`limit`的查询结果不一致。[#13574](https://github.com/StarRocks/starrocks/pull/13574)
- INSERT操作引起的内存泄漏。[#14718](https://github.com/StarRocks/starrocks/pull/14718)
- Primary Key表执行Tablet Migration。[#13720](https://github.com/StarRocks/starrocks/pull/13720)
- Broker Load期间Broker Kerberos票据超时。[#16149](https://github.com/StarRocks/starrocks/pull/16149)
- 表视图中`nullable`信息的推断不正确。[#15744](https://github.com/StarRocks/starrocks/pull/15744)

### 行为变更

- 将默认的Thrift Listen backlog修改为`1024`。[#13911](https://github.com/StarRocks/starrocks/pull/13911)
- 添加了SQL模式`FORBID_INVALID_DATES`。默认情况下此SQL模式被禁用。启用后，StarRocks将验证DATE类型的输入，并在输入无效时返回错误。[#14143](https://github.com/StarRocks/starrocks/pull/14143)

## 2.4.2

发布日期：2022年12月14日

### 改进

- 当存在多个桶时，优化了Bucket Hint的性能。[#13142](https://github.com/StarRocks/starrocks/pull/13142)

### Bug修复

修复了以下错误：

- 刷新Primary Key索引可能导致BE崩溃。[#14857](https://github.com/StarRocks/starrocks/pull/14857) [#14819](https://github.com/StarRocks/starrocks/pull/14819)
- `SHOW FULL TABLES`无法正确识别物化视图类型。[#13954](https://github.com/StarRocks/starrocks/pull/13954)
- 将StarRocks v2.2升级到v2.4可能导致BE崩溃。[#13795](https://github.com/StarRocks/starrocks/pull/13795)
```
- Broker Load 可能会导致 BE 崩溃。[#13973](https://github.com/StarRocks/starrocks/pull/13973)
- 会话变量 `statistic_collect_parallel` 不生效。[#14352](https://github.com/StarRocks/starrocks/pull/14352)
- INSERT INTO 可能会导致 BE 崩溃。[#14818](https://github.com/StarRocks/starrocks/pull/14818)
- JAVA UDF 可能会导致 BE 崩溃。[#13947](https://github.com/StarRocks/starrocks/pull/13947)
- 在部分更新期间克隆副本可能会导致 BE 崩溃并无法重新启动。[#13683](https://github.com/StarRocks/starrocks/pull/13683)
- Colocate Join 可能不会生效。[#13561](https://github.com/StarRocks/starrocks/pull/13561)

### 行为改变

- 将会话变量 `query_timeout` 的限制范围设定为上限 `259200`，下限 `1`。
- 已弃用 FE 参数 `default_storage_medium`。表的存储介质现在由系统自动推断。[#14394](https://github.com/StarRocks/starrocks/pull/14394)

## 2.4.1

发布日期：2022年11月14日

### 新功能

- 支持非等值连接 - LEFT SEMI JOIN 和 ANTI JOIN。优化了 JOIN 功能。[#13019](https://github.com/StarRocks/starrocks/pull/13019)

### 改进

- 在 `HeartbeatResponse` 中支持属性 `aliveStatus`。`aliveStatus` 指示节点在集群中是否处于活动状态。进一步优化了判断 `aliveStatus` 的机制。[#12713](https://github.com/StarRocks/starrocks/pull/12713)

- 优化了 Routine Load 的错误提示。[#12155](https://github.com/StarRocks/starrocks/pull/12155)

### Bug 修复

- BE 从 **v2.4.0RC** 升级到 **v2.4.0** 后崩溃。[#13128](https://github.com/StarRocks/starrocks/pull/13128)

- 延迟物化会导致数据湖查询结果不正确。[#13133](https://github.com/StarRocks/starrocks/pull/13133)

- `get_json_int` 函数抛出异常。[#12997](https://github.com/StarRocks/starrocks/pull/12997)

- 在具有持久索引的 PRIMARY KEY 表删除数据后，数据可能会不一致。[#12719](https://github.com/StarRocks/starrocks/pull/12719)

- BE 可能会在 PRIMARY KEY 表压缩期间崩溃。[#12914](https://github.com/StarRocks/starrocks/pull/12914)

- 当 `json_object` 函数的输入包含空字符串时，它会返回不正确的结果。[#13030](https://github.com/StarRocks/starrocks/issues/13030)

- BE 因 `RuntimeFilter` 而崩溃。[#12807](https://github.com/StarRocks/starrocks/pull/12807)

- FE 由于 CBO 中的过度递归计算而挂起。[#12788](https://github.com/StarRocks/starrocks/pull/12788)

- BE 在正常退出时可能会崩溃或报错。[#12852](https://github.com/StarRocks/starrocks/pull/12852)

- 在向表中添加了新列后删除数据，压缩可能会导致崩溃。[#12907](https://github.com/StarRocks/starrocks/pull/12907)

- 由于 OLAP 外部表元数据同步机制不正确，数据可能会不一致。[#12368](https://github.com/StarRocks/starrocks/pull/12368)

- 当一个 BE 崩溃时，其他 BE 可能会执行相关查询直至超时。[#12954](https://github.com/StarRocks/starrocks/pull/12954)

### 行为改变

- 当解析 Hive 外部表失败时，StarRocks 会抛出错误消息，而不是将相关列转换为 NULL 列。[#12382](https://github.com/StarRocks/starrocks/pull/12382)

## 2.4.0

发布日期：2022年10月20日

### 新功能

- 支持创建基于多个基表的异步物化视图，以加速包含 JOIN 操作的查询。异步物化视图支持所有 [table types](../table_design/table_types/table_types.md)。有关更多信息，请参阅 [Materialized View](../using_starrocks/Materialized_view.md)。

- 支持通过 INSERT OVERWRITE 覆盖数据。有关详细信息，请参阅 [使用 INSERT 加载数据](../loading/InsertInto.md)。

- [预览] 提供可水平扩展的无状态计算节点（CN）。您可以使用 StarRocks Operator 将 CN 部署到 Kubernetes（K8s）集群中，以实现自动水平扩展。有关更多信息，请参阅 [使用 StarRocks Operator 在 Kubernetes 上部署和管理 CN](../deployment/sr_operator.md)。

- 外连接支持非等值连接，其中连接项通过比较运算符（包括 `<`、`<=`、`>`、`>=` 和 `<>`）进行关联。有关详细信息，请参阅 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)。

- 支持创建 Iceberg 目录和 Hudi 目录，这允许直接查询 Apache Iceberg 和 Apache Hudi 中的数据。有关更多信息，请参阅 [Iceberg 目录](../data_source/catalog/iceberg_catalog.md) 和 [Hudi 目录](../data_source/catalog/hudi_catalog.md)。

- 支持从 CSV 格式的 Apache Hive™ 表中查询 ARRAY 类型的列。有关详细信息，请参阅 [外部表](../data_source/External_table.md)。

- 支持通过 DESC 查看外部数据的 schema。有关详细信息，请参阅 [DESC](../sql-reference/sql-statements/Utility/DESCRIBE.md)。

- 支持通过 GRANT 向用户授予特定角色或 IMPERSONATE 权限，并通过 REVOKE 撤销它们，同时支持通过 EXECUTE AS 执行具有 IMPERSONATE 权限的 SQL 语句。有关详细信息，请参阅 [GRANT](../sql-reference/sql-statements/account-management/GRANT.md)、[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md) 和 [EXECUTE AS](../sql-reference/sql-statements/account-management/EXECUTE_AS.md)。

- 支持 FQDN 访问：现在可以使用域名或主机名和端口的组合作为 BE 或 FE 节点的唯一标识。这可以防止因 IP 地址变更而导致的访问失败。有关详细信息，请参阅 [启用 FQDN 访问](../administration/enable_fqdn.md)。

- flink-connector-starrocks 支持 Primary Key 表的部分更新。有关更多信息，请参阅 [使用 flink-connector-starrocks 加载数据](../loading/Flink-connector-starrocks.md)。

- 提供以下新功能：

  - array_contains_all：检查特定数组是否是另一个数组的子集。有关详细信息，请参阅 [array_contains_all](../sql-reference/sql-functions/array-functions/array_contains_all.md)。
  - percentile_cont：使用线性插值计算百分位值。有关详细信息，请参阅 [percentile_cont](../sql-reference/sql-functions/aggregate-functions/percentile_cont.md)。

### 改进

- 主键表支持将 VARCHAR 类型主键索引刷新到磁盘。从 2.4.0 版本开始，无论是否开启持久主键索引，主键表都支持相同的主键索引数据类型。

- 优化了外部表的查询性能。

  - 支持在查询 Parquet 格式的外部表时进行后期物化，以优化涉及小规模过滤的数据湖的查询性能。
  - 可以合并小 I/O 操作，减少查询数据湖的延迟，从而提高外部表的查询性能。

- 优化了窗口函数的性能。

- 通过支持谓词下推优化了 Cross Join 的性能。

- CBO 统计数据中新增了直方图。全量统计收集进一步优化。有关详细信息，请参阅 [Gather CBO statistics](../using_starrocks/Cost_based_optimizer.md)。

- 平板扫描启用自适应多线程，降低了扫描性能对平板数量的依赖。因此，您可以更轻松地设置存储桶的数量。有关详细信息，请参阅 [Determine the number of buckets](../table_design/Data_distribution.md#how-to-determine-the-number-of-buckets)。

- 支持在 Apache Hive 中查询压缩的 TXT 文件。

- 调整了默认 PageCache 大小计算和内存一致性检查机制，避免了多实例部署时出现 OOM 问题。

- 通过移除 final_merge 操作，将主键表上的大容量批量加载的性能提高了两倍。

- 支持 Stream Load 事务接口，对从 Apache Flink®、Apache Kafka® 等外部系统加载数据的事务实现两阶段提交（2PC），提高了高并发流加载的性能。

- 功能：

  - 您可以在一个语句中使用多个 COUNT(DISTINCT)。有关详细信息，请参阅 [count](../sql-reference/sql-functions/aggregate-functions/count.md)。
  - 窗口函数 min() 和 max() 支持滑动窗口。有关详细信息，请参阅 [窗口函数](../sql-reference/sql-functions/Window_function.md)。
  - 优化了 window_funnel 函数的性能。有关详细信息，请参阅 [window_funnel](../sql-reference/sql-functions/aggregate-functions/window_funnel.md)。

### Bug 修复

修复了以下错误：

- DESC 返回的 DECIMAL 数据类型与 CREATE TABLE 语句中指定的数据类型不同。[#7309](https://github.com/StarRocks/starrocks/pull/7309)

- FE 元数据管理问题影响了 FE 的稳定性。[#6685](https://github.com/StarRocks/starrocks/pull/6685) [#9445](https://github.com/StarRocks/starrocks/pull/9445) [#7974](https://github.com/StarRocks/starrocks/pull/7974) [#7455](https://github.com/StarRocks/starrocks/pull/7455)

- 数据加载相关问题：

  - 当指定 ARRAY 列时，Broker Load 失败。[#9158](https://github.com/StarRocks/starrocks/pull/9158)
  - 通过 Broker Load 将数据加载到非 Duplicate Key 表后，副本不一致。[#8714](https://github.com/StarRocks/starrocks/pull/8714)
  - 执行 ALTER ROUTINE LOAD 会引发 NPE。[#7804](https://github.com/StarRocks/starrocks/pull/7804)

- 数据湖分析相关问题：

  - 对 Hive 外部表中的 Parquet 数据的查询失败。[#7413](https://github.com/StarRocks/starrocks/pull/7413) [#7482](https://github.com/StarRocks/starrocks/pull/7482) [#7624](https://github.com/StarRocks/starrocks/pull/7624)
  - 使用 `limit` 子句对 Elasticsearch 外部表的查询返回不正确的结果。[#9226](https://github.com/StarRocks/starrocks/pull/9226)
  - 在对具有复杂数据类型的 Apache Iceberg 表进行查询时出现了未知错误。[#11298](https://github.com/StarRocks/starrocks/pull/11298)

- Leader FE 和 Follower FE 节点之间的元数据不一致。[#11215](https://github.com/StarRocks/starrocks/pull/11215)

- 当 BITMAP 数据大小超过 2 GB 时，BE 会崩溃。[#11178](https://github.com/StarRocks/starrocks/pull/11178)

### 行为改变

- 默认启用 Page Cache（"disable_storage_page_cache" = "false"）。默认缓存大小（`storage_page_cache_limit`）为系统内存的 20%。
- 默认启用 CBO。弃用了会话变量 `enable_cbo`。
- 默认启用向量化引擎。弃用了会话变量 `vectorized_engine_enable`。

### 其他

- 宣布 Resource Group 功能稳定发布。
- 宣布 JSON 数据类型及其相关函数稳定发布。
- 默认情况下启用页面缓存（`disable_storage_page_cache` = `false`）。默认缓存大小（`storage_page_cache_limit`）为系统内存的 20%。
- 默认情况下启用 CBO。已弃用会话变量 `enable_cbo`。
- 默认情况下启用向量化引擎。已弃用会话变量 `vectorized_engine_enable`。

### 其他

- 宣布资源组功能的稳定发布。