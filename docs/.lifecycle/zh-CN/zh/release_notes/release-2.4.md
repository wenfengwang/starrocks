---
displayed_sidebar: "汉语"
---

# StarRocks 版本 2.4

## 2.4.5

发布日期：2023年4月21日

### 功能优化

- 禁止使用 List Partition 语法，因为它可能导致元数据升级出错。[#15401](https://github.com/StarRocks/starrocks/pull/15401)
- 物化视图支持 BITMAP、HLL 和 PERCENTILE 类型。[#15731](https://github.com/StarRocks/starrocks/pull/15731)
- 优化 `storage_medium` 的推导机制。当 BE 同时使用 SSD 和 HDD 作为存储介质时，如果配置了 `storage_cooldown_time`，StarRocks 设置 `storage_medium` 为 `SSD`。反之，则 StarRocks 设置 `storage_medium` 为 `HDD`。[#18649](https://github.com/StarRocks/starrocks/pull/18649)
- 优化 Thread Dump 的准确性。[#16748](https://github.com/StarRocks/starrocks/pull/16748)
- 通过在导入前触发元数据 Compaction 来优化导入效率。[#19347](https://github.com/StarRocks/starrocks/pull/19347)
- 优化 Stream Load Planner 超时。[#18992](https://github.com/StarRocks/starrocks/pull/18992/files)
- 通过禁止收集 Value 列统计数据优化更新模型表性能。[#19563](https://github.com/StarRocks/starrocks/pull/19563)

### 问题修复

修复了如下问题：

- 在 CREATE TABLE 中使用不支持的数据类型时返回 NPE。[#20999](https://github.com/StarRocks/starrocks/issues/20999)
- 使用 Broadcast Join 和 short-circuit 的查询返回错误结果。[#20952](https://github.com/StarRocks/starrocks/issues/20952)
- 错误的数据删除逻辑导致的磁盘占用问题。[#20590](https://github.com/StarRocks/starrocks/pull/20590)
- AuditLoader 插件既无法被安装也不能被删除。[#20468](https://github.com/StarRocks/starrocks/issues/20468)
- 如果调度一个 Tablet 时抛出异常，则与其同一批的其他 Tablet 无法被调度。[#20681](https://github.com/StarRocks/starrocks/pull/20681)
- 在创建同步物化视图时使用了不支持的 SQL 函数时，返回未知错误。[#20348](https://github.com/StarRocks/starrocks/issues/20348)
- 多个 COUNT DISTINCT 重写错误。[#19714](https://github.com/StarRocks/starrocks/pull/19714)
- 查询正在 Compaction 中的 Tablet 返回错误结果。[#20084](https://github.com/StarRocks/starrocks/issues/20084)
- 聚合查询返回错误结果。[#19725](https://github.com/StarRocks/starrocks/issues/19725)
- 将 NULL Parquet 数据加载到 NOT NULL 列时不返回错误消息。[#19885](https://github.com/StarRocks/starrocks/pull/19885)
- 在持续触发资源隔离的并发限制时，查询并发数量指标下降缓慢。[#19363](https://github.com/StarRocks/starrocks/pull/19363)
- 重放 `InsertOverwriteJob` 状态变更日志时，FE 启动失败。[#19061](https://github.com/StarRocks/starrocks/issues/19061)
- 主键模型表死锁。[#18488](https://github.com/StarRocks/starrocks/pull/18488)
- 对于 Colocation 表，可以通过命令手动指定副本状态为 `bad`：`ADMIN SET REPLICA STATUS PROPERTIES("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");`，如果 BE 数量小于等于副本数量，则该副本无法被修复。[#17876](https://github.com/StarRocks/starrocks/issues/17876)
- ARRAY 相关函数引起的问题。[#18556](https://github.com/StarRocks/starrocks/pull/18556)

## 2.4.4

发布日期：2023年2月22日

### 功能优化

- 支持快速取消导入。[#15514](https://github.com/StarRocks/starrocks/pull/15514) [#15398](https://github.com/StarRocks/starrocks/pull/15398) [#15969](https://github.com/StarRocks/starrocks/pull/15969)
- 优化了 Compaction 框架的 CPU 使用率。[#11747](https://github.com/StarRocks/starrocks/pull/11747)
- 支持对缺失数据版本的 Tablet 进行 Cumulative Compaction。[#17030](https://github.com/StarRocks/starrocks/pull/17030)

### 问题修复

修复了如下问题：

- 创建动态分区时，指定非法 DATE 值时会导致系统创建大量动态分区。[#17966](https://github.com/StarRocks/starrocks/pull/17966)
- 无法使用默认的 HTTPS Port 连接到 Elasticsearch 外部表。[#13726](https://github.com/StarRocks/starrocks/pull/13726)
- Stream Load 事务超时后，BE 无法取消事务。[#15738](https://github.com/StarRocks/starrocks/pull/15738)
- 单个 BE 上的 Local Shuffle 聚合返回错误查询结果。[#17845](https://github.com/StarRocks/starrocks/pull/17845)
- 查询可能会失败，并返回错误消息 “wait_for_version version：failed：apply stopped”。[#17848](https://github.com/StarRocks/starrocks/pull/17850)
- OLAP Scan Bucket 表达式未被正确清除，导致返回错误的查询结果。[#17666](https://github.com/StarRocks/starrocks/pull/17666)
- Colocate 组内的动态分区表无法修改分桶数，并返回报错信息。[#17418](https://github.com/StarRocks/starrocks/pull/17418)
- 连接非 Leader FE 节点，发送请求 `USE <catalog_name>.<database_name>`，非 Leader 节点中将请求转发给 Leader FE 时，没有转发 `catalog_name`，导致 Leader FE 使用 `default_catalog`，因此无法找到对应的 database。[#17302](https://github.com/StarRocks/starrocks/pull/17302)
- 执行重写前 dictmapping 检查的逻辑错误。[#17405](https://github.com/StarRocks/starrocks/pull/17405)
- 如果 FE 向 BE 发送单次偶发的心跳，心跳连接超时，FE 会认为该 BE 不可用，最终导致该 BE 上的事务运行失败。[#16386](https://github.com/StarRocks/starrocks/pull/16386)
- 在 FE follower 上查询新克隆的 Tablet，`get_applied_rowsets` 失败。[#17192](https://github.com/StarRocks/starrocks/pull/17192)
- 在 FE follower 上执行 `SET variable = default` 导致空指针（NPE）。[#17549](https://github.com/StarRocks/starrocks/pull/17549)
- Projection 中的表达式不会被字典改写。[#17558](https://github.com/StarRocks/starrocks/pull/17558)
- 以周为粒度创建动态分区的逻辑错误。[#17163](https://github.com/StarRocks/starrocks/pull/17163)
- Local Shuffle 返回错误查询结果。[#17130](https://github.com/StarRocks/starrocks/pull/17130)
- 增量克隆可能会失败。[#16930](https://github.com/StarRocks/starrocks/pull/16930)
- 在某些情况下，CBO 比较操作符是否相等的逻辑存在错误。 [#17199](https://github.com/StarRocks/starrocks/pull/17199) [#17227](https://github.com/StarRocks/starrocks/pull/17227)
- 未检测并正确解析 JuiceFS 架构导致无法访问 JuiceFS。 [#16940](https://github.com/StarRocks/starrocks/pull/16940)

## 2.4.3

发布日期：2023 年 1 月 19 日

### 功能优化

- 在执行 Analyze 前，StarRocks 会提前检查数据库和表是否存在以避免空指针异常（NPE）。 [#14467](https://github.com/StarRocks/starrocks/pull/14467)
- 当查询外部表时，如果列的数据类型不支持，则不会对该列进行物化。 [#13305](https://github.com/StarRocks/starrocks/pull/13305)
- 为 FE 启动脚本 **start_fe.sh** 添加了 Java 版本检验。 [#14333](https://github.com/StarRocks/starrocks/pull/14333)

### 问题修复

修复了以下问题：

- 未设置超时导致 Stream Load 失败。 [#16241](https://github.com/StarRocks/starrocks/pull/16241)
- bRPC 发送在内存消耗大的情况下会导致崩溃。 [#16046](https://github.com/StarRocks/starrocks/issues/16046)
- 无法通过早期版本的 StarRocks 外部表导入数据。 [#16130](https://github.com/StarRocks/starrocks/pull/16130)
- 物化视图刷新失败可能会导致内存泄漏。 [#16041](https://github.com/StarRocks/starrocks/pull/16041)
- Schema Change 在发布阶段卡住。 [#14148](https://github.com/StarRocks/starrocks/issues/14148)
- 物化视图 QeProcessorImpl 问题可能引起内存泄漏。 [#15699](https://github.com/StarRocks/starrocks/pull/15699)
- Limit 查询结果不一致。 [#13574](https://github.com/StarRocks/starrocks/pull/13574)
- INSERT 导入导致内存泄漏。 [#14718](https://github.com/StarRocks/starrocks/pull/14718)
- 主键表执行 Tablet Migration。 [#13720](https://github.com/StarRocks/starrocks/pull/13720)
- 当 Broker Load 任务持续运行时，Broker Kerberos 票据可能会超时。 [#16149](https://github.com/StarRocks/starrocks/pull/16149)
- 在基于表生成视图的过程中，列的 `nullable` 属性转换错误。 [#15744](https://github.com/StarRocks/starrocks/pull/15744)

### 行为变更

- 修改了 Thrift 监听的 `Backlog` 默认值为 `1024`。 [#13911](https://github.com/StarRocks/starrocks/pull/13911)
- 添加了 `FORBID_INVALID_DATES` 的 SQL 模式，默认关闭。启用后将验证日期类型输入，并在日期时间非法时报错。[#14143](https://github.com/StarRocks/starrocks/pull/14143)

## 2.4.2

发布日期：2022 年 12 月 14 日

### 功能优化

- 优化了在存在大量 Bucket 时 Bucket Hint 的性能。 [#13142](https://github.com/StarRocks/starrocks/pull/13142)

### 问题修复

修复了以下问题：

- 主键索引持久化可能引起 BE 崩溃。 [#14857](https://github.com/StarRocks/starrocks/pull/14857) [#14819](https://github.com/StarRocks/starrocks/pull/14819)
- 物化视图表类型未被 `SHOW FULL TABLES` 正确识别。 [#13954](https://github.com/StarRocks/starrocks/pull/13954)
- StarRocks 从 v2.2 升级到 v2.4 可能导致 BE 崩溃。 [#13795](https://github.com/StarRocks/starrocks/pull/13795)
- Broker Load 可能导致 BE 崩溃。 [#13973](https://github.com/StarRocks/starrocks/pull/13973)
- 会话变量 `statistic_collect_parallel` 不生效。 [#14352](https://github.com/StarRocks/starrocks/pull/14352)
- INSERT INTO 可能引起 BE 崩溃。 [#14818](https://github.com/StarRocks/starrocks/pull/14818)
- JAVA UDF 可能导致 BE 崩溃。 [#13947](https://github.com/StarRocks/starrocks/pull/13947)
- Partial Update 过程中副本克隆可能会导致 BE 崩溃，并且无法重新启动。 [#13683](https://github.com/StarRocks/starrocks/pull/13683)
- Colocate Join 可能不生效。 [#13561](https://github.com/StarRocks/starrocks/pull/13561)

### 行为变更

- 会话变量 `query_timeout` 添加了最大值 `259200` 和最小值 `1` 的限制。
- 取消了 FE 参数 `default_storage_medium`，改为由系统自动推导表存储介质。 [#14394](https://github.com/StarRocks/starrocks/pull/14394)

## 2.4.1

发布日期：2022 年 11 月 14 日

### 新特性

- 新增了对非等值 LEFT SEMI/ANTI JOIN 的支持，以补充 JOIN 功能。 [#13019](https://github.com/StarRocks/starrocks/pull/13019)

### 功能优化

- 在 `HeartbeatResponse` 中添加了 `aliveStatus` 属性，用以判定节点的在线状态，优化了节点在线判断逻辑。 [#12713](https://github.com/StarRocks/starrocks/pull/12713)

- 优化了 Routine Load 报错信息的显示。 [#12155](https://github.com/StarRocks/starrocks/pull/12155)

### 问题修复

修复了以下问题：

- 从 2.4.0 RC 升级到 2.4.0 后导致 BE 崩溃。 [#13128](https://github.com/StarRocks/starrocks/pull/13128)

- 查询数据湖时，延迟物化可能导致查询结果不正确。 [#13133](https://github.com/StarRocks/starrocks/pull/13133)

- 函数 get_json_int 报错。 [#12997](https://github.com/StarRocks/starrocks/pull/12997)

- 主键表的索引持久化删除数据时可能导致数据不一致。 [#12719](https://github.com/StarRocks/starrocks/pull/12719)

- 主键表在执行 Compaction 时可能会导致 BE 崩溃。 [#12914](https://github.com/StarRocks/starrocks/pull/12914)

- 当函数 json_object 的输入包含空字符串时返回错误结果。 [#13030](https://github.com/StarRocks/starrocks/issues/13030)

- RuntimeFilter 可能会导致 BE 崩溃。 [#12807](https://github.com/StarRocks/starrocks/pull/12807)

- CBO 内部过多的递归计算会导致 FE 挂起。 [#12788](https://github.com/StarRocks/starrocks/pull/12788)

- 优雅退出时，BE 可能会崩溃或报错。 [#12852](https://github.com/StarRocks/starrocks/pull/12852)

- 在添加新列后进行删除操作可能会引起 Compaction 崩溃。 [#12907](https://github.com/StarRocks/starrocks/pull/12907)

- OLAP 外表元数据同步可能会导致数据不一致。 [#12368](https://github.com/StarRocks/starrocks/pull/12368)

- 当一个 BE 崩溃后，相关查询有小概率在其他 BE 上持续执行直到超时。[#12954](https://github.com/StarRocks/starrocks/pull/12954)

### 行为变更

- 当 Hive 外部表解析出错时，StarRocks 将报错，而不是将相关列设为 NULL。 [#12382](https://github.com/StarRocks/starrocks/pull/12382)

## 2.4.0

发布日期： 2022 年 10 月 20 日

### 新增特性

- 支持构建异步多表物化视图以加速多表 JOIN 查询。异步物化视图适用于所有[数据模型](../table_design/table_types/table_types.md)。相关文档请参见 [物化视图](../using_starrocks/Materialized_view.md)。

- 支持通过 INSERT OVERWRITE 语句批量写入并覆盖数据。相关文档请参见 [INSERT 导入](../loading/InsertInto.md)。

- [公测中] 提供无状态计算节点（Compute Node，简称 CN）功能。这些计算节点支持无状态的扩展，可以通过 StarRocks Operator 部署并基于 Kubernetes 来管理容器化的计算节点以自动扩展。相关文档请参见[使用 StarRocks Operator 在 Kubernetes 上部署和管理 CN](../deployment/sr_operator.md)。

- Outer Join 现支持使用 `<`、`<=`、`>`、`>=`、`<>` 等比较运算符进行非等值多表关联。相关文档请参见 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)。

- 支持创建 Iceberg Catalog 和 Hudi Catalog，创建后可直接查询 Apache Iceberg 和 Apache Hudi 数据。相关文档请参见 [Iceberg catalog](../data_source/catalog/iceberg_catalog.md) 和 [Hudi catalog](../data_source/catalog/hudi_catalog.md)。

- 支持查询 CSV 格式 Apache Hive™ 表中的 ARRAY 列。相关文档请参见[外部表](../data_source/External_table.md#创建-hive-外表)。

- 支持通过 DESC 语句查看外部数据表的结构。相关文档请参见 [DESC](../sql-reference/sql-statements/Utility/DESCRIBE.md)。

- 支持通过 GRANT 或 REVOKE 语句分配或收回用户特定角色或 IMPERSONATE 权限，并支持使用 EXECUTE AS 语句在当前会话中执行 IMPERSONATE 权限。相关文档请参见 [GRANT](../sql-reference/sql-statements/account-management/GRANT.md)、[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md) 和 [EXECUTE AS](../sql-reference/sql-statements/account-management/EXECUTE_AS.md)。

- 支持以 FQDN（完全限定域名）来访问：现在可以使用域名或结合主机名与端口的形式来唯一标识 FE 或 BE 节点，从而避免 IP 变化导致的访问问题。相关文档请参见 [启用 FQDN 访问](../administration/enable_fqdn.md)。

- flink-connector-starrocks 现支持主键模型 Partial Update。相关文档请参见[使用 flink-connector-starrocks 导入至 StarRocks](../loading/Flink-connector-starrocks.md)。

- 函数相关：
  - 新增 array_contains_all 函数用于判断一个数组是否为另一数组的子集。相关文档请参见[array_contains_all](../sql-reference/sql-functions/array-functions/array_contains_all.md)。
  - 新增 percentile_cont 函数，用于计算百分位数（通过线性插值法）。相关文档请参见[percentile_cont](../sql-reference/sql-functions/aggregate-functions/percentile_cont.md)。

### 功能优化

- 主键模型支持持久化存储 VARCHAR 类型的主键索引。从 2.4.0 版本开始，主键模型的主键索引磁盘持久化模式和内存常驻模式支持相同的数据类型。

- 优化外部表查询性能。
  - 在查询 Parquet 格式文件时支持延迟物化，提升小范围筛选下的数据湖查询性能。
  - 在数据湖查询中，支持合并小型 I/O 以降低对存储系统的访问延迟，进而提升外部表的查询性能。

- 优化窗口函数性能。

- Cross Join 现支持谓词下推，以提升性能。

- 统计信息中加入直方图，并继续完善全量统计信息收集。相关文档请参见[CBO 统计信息](../using_starrocks/Cost_based_optimizer.md)。

- 支持 Tablet 自适应多线程 Scan，减少对同一磁盘 Tablet 数量的依赖，简化分桶数量的设置。相关文档请参见[决定分桶数量](../table_design/Data_distribution.md#确定分桶数量)。

- 支持查询 Apache Hive 中的压缩文本文件（.txt）。

- 调整计算默认 PageCache Size 和一致性校验内存的方法，以避免在多实例部署的情况下出现 OOM 问题。

- 取消了导入主键模型数据的 final_merge 操作，提高了主键模型大数据量单批次导入的性能，速度提升至两倍。

- 支持 Stream Load 事务接口：实现与 Apache Flink®、Apache Kafka® 等其他系统之间跨系统的两阶段提交，以及提升高并发 Stream Load 导入的性能。

- 函数相关：
  - 支持一条 SELECT 语句中使用多个 COUNT(DISTINCT)。相关文档请参见[count](../sql-reference/sql-functions/aggregate-functions/count.md)。
  - 窗口函数 min 和 max 现支持滑动窗口。相关文档请参见[窗口函数](../sql-reference/sql-functions/Window_function.md#使用-MAX()-窗口函数)。
  - 优化了 window_funnel 函数的性能。相关文档请参见[window_funnel](../sql-reference/sql-functions/aggregate-functions/window_funnel.md)。

### 问题修复

修复了以下问题：

- 使用 DESC 查看表结构信息时，显示的字段类型与创建表时指定的类型不符。[#7309](https://github.com/StarRocks/starrocks/pull/7309)

- 影响 FE 稳定性的元数据问题。[#6685](https://github.com/StarRocks/starrocks/pull/6685) [#9445](https://github.com/StarRocks/starrocks/pull/9445) [#7974](https://github.com/StarRocks/starrocks/pull/7974) [#7455](https://github.com/StarRocks/starrocks/pull/7455)

- 导入相关问题：
  - 在 Broker Load 导入过程中，设置 ARRAY 列失败。 [#9158](https://github.com/StarRocks/starrocks/pull/9158)
  - 通过 Broker Load 导入数据到非明细模型表后，副本数据不一致。[#8714](https://github.com/StarRocks/starrocks/pull/8714)
  - 在执行 ALTER ROUTINE LOAD 过程中遇到 NPE 错误。 [#7804](https://github.com/StarRocks/starrocks/pull/7804)

- 数据湖分析相关问题：
  - 在查询 Hive 外部表中 Parquet 格式数据时失败。 [#7413](https://github.com/StarRocks/starrocks/pull/7413) [#7482](https://github.com/StarRocks/starrocks/pull/7482) [#7624](https://github.com/StarRocks/starrocks/pull/7624)
  - Elasticsearch 外部表 Limit 查询结果不正确。[#9226](https://github.com/StarRocks/starrocks/pull/9226)
  - 查询存有复杂数据类型的 Apache Iceberg 表返回未知错误。[#11298](https://github.com/StarRocks/starrocks/pull/11298)

- Leader FE 节点与 Follower FE 节点间元数据不同步。[#11215](https://github.com/StarRocks/starrocks/pull/11215)

- 当 BITMAP 类型数据大于 2GB 时，BE 停止服务。[#11178](https://github.com/StarRocks/starrocks/pull/11178)

### 行为变更

- 默认启用 Page Cache（"disable_storage_page_cache" = "false"）。cache size（`storage_page_cache_limit`）为系统内存大小的 20%。
- 默认启用 CBO 优化器，废弃 session 变量 `enable_cbo`。
- 默认启用向量化引擎，废弃 session 变量 `vectorized_engine_enable`。

### 其他

- 现已正式支持资源隔离功能。
- 现已正式支持 JSON 数据类型及相关函数。