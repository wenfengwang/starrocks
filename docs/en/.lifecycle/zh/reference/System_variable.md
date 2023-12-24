---
displayed_sidebar: English
---

# 系统变量

StarRocks 提供了许多系统变量，可以进行设置和修改以满足您的需求。本节描述了 StarRocks 支持的变量。您可以通过在 MySQL 客户端上运行 [SHOW VARIABLES](../sql-reference/sql-statements/Administration/SHOW_VARIABLES.md) 命令来查看这些变量的设置。您还可以使用 [SET](../sql-reference/sql-statements/Administration/SET.md) 命令动态设置或修改变量。您可以使这些变量在整个系统上全局生效，仅在当前会话中生效，或者仅在单个查询语句中生效。

StarRocks 中的变量是指 MySQL 中的变量集，但有些变量仅兼容 MySQL 客户端协议，在 MySQL 数据库上不起作用。

> **注意**
>
> 任何用户都有权运行 SHOW VARIABLES 并使变量在会话级别生效。但是，只有具有 SYSTEM 级 OPERATE 权限的用户才能使变量全局生效。全局有效变量将对所有未来会话（不包括当前会话）生效。
>
> 如果要对当前会话进行设置更改，并将该设置更改应用于所有将来的会话，则可以进行两次更改，一次不带修饰符 `GLOBAL`，一次使用修饰符。例如：
>
> ```SQL
> SET query_mem_limit = 137438953472; -- 应用于当前会话。
> SET GLOBAL query_mem_limit = 137438953472; -- 应用于所有将来的会话。
> ```

## 变量层次结构和类型

StarRocks 支持三种类型（级别）的变量：全局变量、会话变量和 `SET_VAR` 提示。它们的层次结构关系如下：

* 全局变量在全局级别生效，并且可以被会话变量和 `SET_VAR` 提示覆盖。
* 会话变量仅在当前会话上生效，并且可以被 `SET_VAR` 提示覆盖。
* `SET_VAR` 提示仅对当前查询语句生效。

## 查看变量

您可以使用 `SHOW VARIABLES [LIKE 'xxx']` 查看所有或部分变量。例如：

```SQL
-- 显示系统中的所有变量。
SHOW VARIABLES;

-- 显示与特定模式匹配的变量。
SHOW VARIABLES LIKE '%time_zone%';
```

## 设置变量

### 全局设置变量或为单个会话设置变量

您可以将变量设置为**全局生效**或**仅在当前会话上**生效。设置为全局时，新值将用于所有将来的会话，而当前会话仍使用原始值。当设置为“仅限当前会话”时，该变量将仅在当前会话上生效。

通过 `SET <var_name> = xxx;` 设置的变量仅对当前会话生效。例如：

```SQL
SET query_mem_limit = 137438953472;

SET forward_to_master = true;

SET time_zone = "Asia/Shanghai";
```

通过 `SET GLOBAL <var_name> = xxx;` 设置的变量将全局生效。例如：

```SQL
SET GLOBAL query_mem_limit = 137438953472;
```

以下变量仅全局生效。它们不能对单个会话生效，这意味着您必须使用 `SET GLOBAL <var_name> = xxx;` 设置这些变量。如果尝试为单个会话设置此类变量（`SET <var_name> = xxx;`），将返回错误。

* activate_all_roles_on_login
* character_set_database
* default_rowset_type
* enable_query_queue_select
* enable_query_queue_statistic
* enable_query_queue_load
* init_connect
* lower_case_table_names
* license
* language
* query_cache_size
* query_queue_fresh_resource_usage_interval_ms
* query_queue_concurrency_limit
* query_queue_mem_used_pct_limit
* query_queue_cpu_used_permille_limit
* query_queue_pending_timeout_second
* query_queue_max_queued_queries
* system_time_zone
* version_comment
* version

此外，变量设置还支持常量表达式，例如：

```SQL
SET query_mem_limit = 10 * 1024 * 1024 * 1024;
```

```SQL
SET forward_to_master = concat('tr', 'u', 'e');
```

### 在单个查询语句中设置变量

在某些情况下，可能需要专门为某些查询设置变量。通过使用 `SET_VAR` 提示，您可以设置仅在单个语句中生效的会话变量。例如：

```sql
SELECT /*+ SET_VAR(query_mem_limit = 8589934592) */ name FROM people ORDER BY name;

SELECT /*+ SET_VAR(query_timeout = 1) */ sleep(3);
```

> **注意**
>
> `SET_VAR` 只能放在 `SELECT` 关键字之后，并括在 `/*+...*/` 中。

您还可以在单个语句中设置多个变量。例如：

```sql
SELECT /*+ SET_VAR
  (
  exec_mem_limit = 515396075520,
  query_timeout=10000000,
  batch_size=4096,
  parallel_fragment_exec_instance_num=32
  )
  */ * FROM TABLE;
```

## 变量说明

变量按**字母顺序描述**。带有标签 `global` 的变量只能全局生效。其他变量可以全局生效，也可以对单个会话生效。

### activate_all_roles_on_login（全局）

当 StarRocks 用户连接到 StarRocks 集群时，是否为其启用所有角色（包括默认角色和授予角色）。从 v3.0 开始支持此变量。

* 如果启用（true），则用户的所有角色都将在用户登录时激活。这优先于 [SET DEFAULT ROLE 设置的角色](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md)。
* 如果禁用（false），则激活 SET DEFAULT ROLE 设置的角色。

默认值：false。

如果要激活在会话中指派给您的角色，请使用 [SET ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md) 命令。

### auto_increment_increment

用于 MySQL 客户端兼容性。没有实际用途。

### autocommit

用于 MySQL 客户端兼容性。没有实际用途。

### batch_size

用于指定查询执行过程中每个节点传输的单个数据包的行数。默认值为 1024，即源节点生成的每 1024 行数据打包并发送到目标节点。在大数据量方案中，行数越大，查询吞吐量越大，但在数据量小的场景下，查询时延可能会增加。此外，它可能会增加查询的内存开销。建议将 `batch_size` 设置在 1024 到 4096 之间。

### big_query_profile_second_threshold（3.1 及更高版本）

当会话变量 `enable_profile` 设置为 `false` 并且查询所花费的时间超过 `big_query_profile_second_threshold` 指定的阈值时，将为该查询生成配置文件。

### cbo_decimal_cast_string_strict（2.5.14 及更高版本）

控制 CBO 如何将数据从 DECIMAL 类型转换为 STRING 类型。如果该变量设置为 `true`，则以 v2.5.x 及之后版本内置的逻辑为准，系统实现严格转换（即系统截断生成的字符串，并根据刻度长度填充 0s）。如果此变量设置为 `false`，则以 v2.5.x 之前版本内置的逻辑为准，系统处理所有有效数字以生成字符串。缺省值为 `true`。

### cbo_enable_low_cardinality_optimize

是否启用低基数优化。开启该功能后，查询 STRING 列的性能提升了 3 倍左右。默认值：true。

### cbo_eq_base_type（2.5.14 及更高版本）

指定用于 DECIMAL 类型数据和 STRING 类型数据之间的数据比较的数据类型。默认值为 `VARCHAR`，DECIMAL 也是有效值。

### character_set_database（全局）

StarRocks 支持的字符集。仅支持 UTF8（`utf8`）。

### connector_io_tasks_per_scan_operator（2.5 及更高版本）

在外部表查询期间，扫描操作员可以发出的最大并发 I/O 任务数。该值为整数。默认值：16。

目前，StarRocks 在查询外部表时，可以自适应地调整并发 I/O 任务的数量。此功能由变量 `enable_connector_adaptive_io_tasks` 控制，该变量默认启用。

### count_distinct_column_buckets（2.5 及更高版本）

group-by-count-distinct 查询中 COUNT DISTINCT 列的存储桶数。仅当设置为 `enable_distinct_column_bucketization` 为 `true` 时，此变量才生效。默认值：1024。

### default_rowset_type（全局）

全局变量。用于设置计算节点的存储引擎使用的默认存储格式。当前支持的存储格式为 `alpha` 和 `beta`。

### default_table_compression（3.0 及更高版本）

表存储的默认压缩算法。支持的压缩算法为 `snappy, lz4, zlib, zstd`。默认值：lz4_frame。

请注意，如果在 CREATE TABLE 语句中指定了 `compression` 属性，则指定的压缩算法 `compression` 将生效。

### disable_colocate_join
用于控制是否启用共置联接。默认值为 `false`，表示该功能已启用。禁用此功能后，查询计划将不会尝试执行共置联接。

### disable_streaming_preaggregations

用于启用流式预聚合。默认值为 `false`，表示已启用。

### div_precision_increment

用于 MySQL 客户端兼容性。没有实际用途。

### enable_connector_adaptive_io_tasks（2.5 及更高版本）

查询外部表时，是否自适应调整并发 I/O 任务数。默认值：true。

如果未启用此功能，则可以使用变量 `connector_io_tasks_per_scan_operator` 手动设置并发 I/O 任务数。

### enable_distinct_column_bucketization（2.5 及更高版本）

是否在 group-by-count-distinct 查询中为 COUNT DISTINCT 列启用分桶化。使用查询 `select a, count(distinct b) from t group by a;` 作为示例。如果 GROUP BY `a` 列是低基数列，而 COUNT DISTINCT `b` 列是高基数列，并且存在严重的数据偏差，则会出现性能瓶颈。在这种情况下，您可以将 COUNT DISTINCT 列中的数据拆分为多个存储桶，以平衡数据并防止数据倾斜。

默认值：false，表示此功能已禁用。您必须将此变量与变量 `count_distinct_column_buckets` 一起使用。

您还可以通过向查询添加提示来为 COUNT DISTINCT 列启用分桶化，例如 `select a,count(distinct [skew] b) from t group by a;`。

### enable_group_level_query_queue（3.1.4 及更高版本）

是否开启资源组级 [查询队列](../administration/query_queues.md)。

默认值：false，表示此功能已禁用。

### enable_insert_strict

用于在使用 INSERT 语句加载数据时启用严格模式。默认值为 `true`，表示默认启用严格模式。有关详细信息，请参阅 [严格模式](../loading/load_concept/strict_mode.md)。

### enable_materialized_view_union_rewrite（2.5 及更高版本）

用于控制是否启用物化视图并集查询重写的布尔值。默认值： `true`。

### enable_rule_based_materialized_view_rewrite（2.5 及更高版本）

布尔值，用于控制是否启用基于规则的物化视图查询重写。该变量主要用于单表查询重写。默认值： `true`。

### enable_spill（3.0 及更高版本）

是否启用中间结果溢出。默认值： `false`。如果设置为 `true`，StarRocks 会将中间结果溢出到磁盘，以减少在查询中处理聚合、排序或 join 算子时的内存占用。

### enable_profile

指定是否发送查询的配置文件进行分析。默认值为 `false`，表示不需要配置文件。

默认情况下，只有当 BE 中发生查询错误时，才会向 FE 发送配置文件。配置文件发送会导致网络开销，从而影响高并发。

如果需要分析查询的配置文件，可以将此变量设置为 `true`。查询完成后，可以在当前连接的 FE（地址：`fe_host:fe_http_port/query`）的网页上查看配置文件。此页面显示最近 100 个已打开查询的配置文件，其中 `enable_profile` 已打开。

### enable_query_queue_load （全球）

布尔值，用于启用用于加载任务的查询队列。默认值： `false`。

### enable_query_queue_select （全球）

布尔值，用于为 SELECT 查询启用查询队列。默认值： `false`。

### enable_query_queue_statistic （全球）

布尔值，用于启用统计信息查询的查询队列。

### enable_query_tablet_affinity（2.5 及更高版本）

布尔值，用于控制是否将针对同一平板电脑的多个查询定向到固定副本。

在要查询的表有大量 tablet 的场景下，由于 tablet 的元信息和数据可以更快地缓存在内存中，因此可以显著提高查询性能。

但是，如果存在一些热点平板电脑，则该功能可能会降低查询性能，因为它将查询定向到同一个 BE，导致在高并发场景下无法充分利用多个 BE 的资源。

默认值： `false`，表示系统为每个查询选择一个副本。自 2.5.6、3.0.8、3.1.4 和 3.2.0 起支持此功能。

### enable_scan_datacache（2.5 及更高版本）

指定是否启用数据缓存功能。开启该功能后，StarRocks 会将从外部存储系统读取的热数据缓存到区块中，从而加快查询和分析速度。有关详细信息，请参阅 [数据缓存](../data_source/data_cache.md)。在 3.2 之前的版本中，此变量被命名为 `enable_scan_block_cache`。

### enable_populate_datacache（2.5 及更高版本）

指定是否在 StarRocks 中缓存从外部存储系统读取的数据块。如果不想缓存从外部存储系统读取的数据块，请将此变量设置为 `false`。默认值：true。从 2.5 开始支持此变量。在 3.2 之前的版本中，此变量被命名为 `enable_scan_block_cache`。

### enable_tablet_internal_parallel（2.3 及更高版本）

是否启用平板电脑的自适应并行扫描。开启该功能后，可以使用多个线程逐段扫描一台平板电脑，从而增加扫描并发。默认值：true。

### enable_query_cache（2.5 及更高版本）

指定是否启用查询缓存功能。取值范围：true、false。指定 `true` 启用此功能，并指定 `false` 禁用此功能。开启该功能后，仅对满足查询缓存应用场景指定条件的查询有效。

### enable_adaptive_sink_dop（2.5 及更高版本）

指定是否为数据加载启用自适应并行性。开启该功能后，系统会自动设置 INSERT INTO 和 Broker Load 作业的加载并行度，相当于 `pipeline_dop` 的机制。对于新部署的 v2.5 StarRocks 集群，该值为 `true` 默认值。对于从 v2.4 升级的 v2.5 群集，该值为 `false`。

### enable_pipeline_engine

指定是否启用管道执行引擎。表示 `true` 已启用，表示 `false` 相反。默认值：`true`。

### enable_sort_aggregate（2.5 及更高版本）

指定是否启用排序流式处理。`true` 表示已启用排序流式处理以对数据流中的数据进行排序。

### enable_global_runtime_filter

是否开启全局运行时过滤器（简称RF）。RF 在运行时过滤数据。数据筛选通常发生在联接阶段。在多表联接过程中，使用谓词下推等优化来过滤数据，以减少 Join 和 Shuffle 阶段的 I/O扫描行数，从而加快查询速度。

StarRocks 提供两种类型的 RF：Local RF 和 Global RF，Local RF 适用于 Broadcast Hash Join，Global RF 适用于 Shuffle Join。

默认值： `true`，表示启用全局 RF。如果禁用此功能，全局射频不会生效。本地射频仍然可以工作。

### enable_multicolumn_global_runtime_filter

是否启用多列全局运行时筛选器。默认值： `false`，表示禁用多列全局 RF。

如果联接（广播联接和复制联接除外）具有多个等联接条件：

* 如果禁用此功能，则只有本地 RF 有效。
* 开启该功能后，多列全局射频生效，并带入 `multi-column` 分区依据子句。

### ENABLE_WRITE_HIVE_EXTERNAL_TABLE（v3.2 及更高版本）

是否允许将数据下沉到 Hive 的外部表。默认值： `false`。

### event_scheduler

用于 MySQL 客户端兼容性。没有实际用途。

### enable_strict_type（v3.1 及更高版本）

是否允许对 WHERE 子句中的所有复合谓词和所有表达式进行隐式转换。默认值： `false`。

### force_streaming_aggregate

用于控制聚合节点是否开启流式聚合进行计算。默认值为 false，表示未启用该功能。

### forward_to_master

用于指定是否将某些命令转发给 leader FE 执行。默认值为 `false`，表示不转发到主 FE。一个 StarRocks 集群中有多个 FE，其中一个是 leader FE。通常，用户可以连接到任何 FE 进行全功能操作。但是，某些信息仅在主 FE 上可用。


例如，如果未将 SHOW BACKENDS 命令转发到 leader FE，则只能查看基本信息（例如节点是否处于活动状态）。转发到 leader FE 可以获取更详细的信息，包括节点启动时间和上次心跳时间。

受此变量影响的命令如下：

* SHOW FRONTENDS：转发到 leader FE 允许用户查看最后的心跳消息。

* SHOW BACKENDS：转发到 leader FE 允许用户查看启动时间、上次心跳信息和磁盘容量信息。

* SHOW BROKER：转发到 leader FE 允许用户查看启动时间和上次心跳信息。

* SHOW TABLET

* ADMIN SHOW REPLICA DISTRIBUTION

* ADMIN SHOW REPLICA STATUS：转发到 leader FE 允许用户查看存储在主 FE 元数据中的平板电脑信息。通常，不同 FE 的元数据中的平板电脑信息应该是相同的。如果发生错误，您可以使用此方法比较当前 FE 和 leader FE 的元数据。

* Show PROC：转发到 leader FE 允许用户查看存储在元数据中的 PROC 信息。这主要用于元数据比较。

### group_concat_max_len

[group_concat](../sql-reference/sql-functions/string-functions/group_concat.md) 函数返回的字符串的最大长度。默认值：1024。最小值：4。单位：字符。

### hash_join_push_down_right_table

用于控制是否可以通过在 Join 查询中对右表使用筛选条件来筛选左表的数据。如果可以，它可以减少查询期间需要处理的数据量。

`true` 表示允许该操作，系统会决定是否可以过滤左侧表。`false` 指示该操作已禁用。默认值为 `true`。

### init_connect（全局）

用于 MySQL 客户端兼容性。没有实际用途。

### interactive_timeout

用于 MySQL 客户端兼容性。没有实际用途。

### io_tasks_per_scan_operator（2.5 及更高版本）

扫描操作符可以发出的并发 I/O 任务数。如果要访问 HDFS 或 S3 等远程存储系统，但延迟较高，请增加此值。然而，较大的值会导致更多的内存消耗。

该值为整数。默认值：4。

### language（全局）

用于 MySQL 客户端兼容性。没有实际用途。

### license（全局）

显示 StarRocks 的许可证。

### load_mem_limit

指定导入操作的内存限制。默认值为 0，表示不使用此变量，而是使用 `query_mem_limit`。

此变量仅用于涉及查询和导入的 `INSERT` 操作。如果用户未设置此变量，则查询和导入的内存限制将设置为 `exec_mem_limit`。否则，查询的内存限制将设置为 `exec_mem_limit`，导入的内存限制将设置为 `load_mem_limit`。

其他导入方法（如 `BROKER LOAD`、`STREAM LOAD`）仍使用 `exec_mem_limit` 作为内存限制。

### log_rejected_record_num（v3.1 及更高版本）

指定可以记录的最大非限定数据行数。有效值：`0`、`-1` 和任何非零正整数。默认值：`0`。

* 值 `0` 指定不会记录筛选出的数据行。
* 值 `-1` 指定将记录筛选出的所有数据行。
* 非零正整数（如 `n`）指定每个 BE 上最多可以记录被筛选掉的数据行数为 `n`。

### lower_case_table_names（全局）

用于 MySQL 客户端兼容性。没有实际用途。StarRocks 中的表名区分大小写。

### materialized_view_rewrite_mode（v3.2 及更高版本）

指定异步物化视图的查询重写模式。有效值：

* `disable`：禁用异步物化视图的自动查询重写。
* `default`（默认值）：启用异步物化视图的自动查询重写，并允许优化器根据成本决定是否可以使用物化视图重写查询。如果无法重写查询，则直接扫描基表中的数据。
* `default_or_error`：启用异步物化视图的自动查询重写，并允许优化器根据成本决定是否可以使用物化视图重写查询。如果无法重写查询，则返回错误。
* `force`：启用异步物化视图的自动查询重写，优化器使用物化视图优先重写查询。如果无法重写查询，则直接扫描基表中的数据。
* `force_or_error`：启用异步物化视图的自动查询重写，优化器使用物化视图优先重写查询。如果无法重写查询，则返回错误。

### max_allowed_packet

用于与 JDBC 连接池 C3P0 兼容。此变量指定可在客户端和服务器之间传输的数据包的最大大小。默认值：32 MB。单位：字节。如果客户端报告“PacketTooBigException”，则可以增加此值。

### max_scan_key_num

每个查询分段的最大扫描密钥数。默认值：-1，表示使用 `be.conf` 文件中的值。如果此变量设置为大于 0 的值，则忽略 `be.conf` 中的值。

### max_pushdown_conditions_per_column

可以向下推一列的最大谓词数。默认值：-1，表示使用 `be.conf` 文件中的值。如果此变量设置为大于 0 的值，则忽略 `be.conf` 中的值。

### nested_mv_rewrite_max_level

可用于查询重写的嵌套物化视图的最大级别。类型：INT。范围：[1，+∞)。值为 `1` 表示只有在基表上创建的物化视图才能用于查询重写。默认值：`3`。

### net_buffer_length

用于 MySQL 客户端兼容性。没有实际用途。

### net_read_timeout

用于 MySQL 客户端兼容性。没有实际用途。

### net_write_timeout

用于 MySQL 客户端兼容性。没有实际用途。

### new_planner_optimize_timeout

查询优化器的超时持续时间。当优化器超时时，将返回错误并停止查询，从而影响查询性能。您可以根据自己的查询设置较大的值，也可以联系 StarRocks 技术支持进行排查。当查询具有过多联接时，通常会发生超时。

默认值：3000。单位：毫秒。

### parallel_exchange_instance_num

用于设置上层节点用于从执行计划中的下层节点接收数据的交换节点数。默认值为 -1，表示交换节点数等于较低级别节点的执行实例数。当此变量设置为大于 0 但小于较低级别节点的执行实例数时，交换节点数等于设置值。

在分布式查询执行计划中，上层节点通常有一个或多个交换节点，用于接收来自不同 BE 上下层节点执行实例的数据。通常，交换节点的数量等于较低级别节点的执行实例数量。

在一些聚合查询场景中，聚合后数据量急剧减少，可以尝试将该变量修改为较小的值，以降低资源开销。例如，使用重复键表运行聚合查询。

### parallel_fragment_exec_instance_num

用于设置用于扫描每个 BE 上的节点的实例数。默认值为 1。

查询计划通常会生成一组扫描范围。此数据分布在多个 BE 节点上。一个 BE 节点将具有一个或多个扫描范围，默认情况下，每个 BE 节点的一组扫描范围仅由一个执行实例处理。当计算机资源足够时，可以增加此变量以允许更多执行实例同时处理扫描范围以提高效率。

扫描实例的数量决定了上层其他执行节点的数量，如聚合节点、加入节点等。因此，它增加了整个查询计划执行的并发性。修改此变量将有助于提高效率，但较大的值将消耗更多的计算机资源，例如 CPU、内存和磁盘 IO。

### partial_update_mode（3.1 及更高版本）

用于控制部分更新的模式。有效值：

* `auto`（默认）：系统通过分析 UPDATE 语句和涉及的列，自动确定部分更新的模式。
* `column`：部分更新采用列模式，特别适用于列数少、行数大的部分更新。

有关详细信息，请参阅 [UPDATE](../sql-reference/sql-statements/data-manipulation/UPDATE.md#partial-updates-in-column-mode-since-v31)。

### performance_schema
用于与 MySQL JDBC 版本 8.0.16 及更高版本兼容。没有实际用途。

### prefer_compute_node

指定 FE 是否将查询执行计划分发给 CN 节点。有效取值：

* true：表示 FE 将查询执行计划分发给 CN 节点。
* false：表示 FE 不将查询执行计划分发给 CN 节点。

### pipeline_dop

管道实例的并行度，用于调整查询并发。默认值为 0，表示系统会自动调整每个管道实例的并行度。您也可以将此变量设置为大于 0 的值。通常情况下，将该值设置为物理 CPU 内核数的一半。

从 v3.0 开始，StarRocks会根据查询并行度自适应调整该变量。

### pipeline_profile_level

控制查询配置文件的级别。查询配置文件通常包括五个层级：Fragment、FragmentInstance、Pipeline、PipelineDriver 和 Operator。不同级别提供不同详细的配置文件信息：

* 0：StarRocks合并了配置文件的指标，并仅显示了少量核心指标。
* 1：默认值。StarRocks简化了配置文件，并合并了配置文件的指标，以减少配置文件的层级。
* 2：StarRocks保留了配置文件的所有层级。在复杂的 SQL 查询中，配置文件的大小会很大。不建议使用此值。

### query_cache_entry_max_bytes（2.5 及更高版本）

触发直通模式的阈值。有效取值范围为 0 到 9223372036854775807。当查询访问的特定平板计算结果的字节数或行数超过 `query_cache_entry_max_bytes` 或 `query_cache_entry_max_rows` 指定的阈值时，查询将切换到直通模式。

### query_cache_entry_max_rows（2.5 及更高版本）

可缓存的行数上限。详细信息请参见 `query_cache_entry_max_bytes` 中的说明。默认值为 409600。

### query_cache_agg_cardinality_limit（2.5 及更高版本）

查询缓存中 GROUP BY 的基数上限。如果 GROUP BY 生成的行数超过此值，则不会启用查询缓存。默认值为 5000000。如果 `query_cache_entry_max_bytes` 或 `query_cache_entry_max_rows` 设置为 0，则即使所涉及的平板电脑未生成任何计算结果，也会使用直通模式。

### query_cache_size（全局）

用于 MySQL 客户端兼容性。没有实际用途。

### query_cache_type

用于与 JDBC 连接池 C3P0 兼容。没有实际用途。

### query_mem_limit

用于设置每个后端节点上查询的内存限制。默认值为 0，表示没有限制。支持的单位包括 `B/K/KB/M/MB/G/GB/T/TB/P/PB`。

当出现 `Memory Exceed Limit` 错误时，您可以尝试增加此变量。

### query_queue_concurrency_limit（全局）

BE 上的并发查询上限。仅在设置大于 `0` 后生效。默认值为 `0`。

### query_queue_cpu_used_permille_limit（全局）

BE 上每千分之 CPU 使用率的上限（CPU 使用率 * 1000）。仅在设置大于 `0` 后生效。默认值为 `0`。取值范围：[0, 1000]

### query_queue_max_queued_queries（全局）

队列中查询的上限。当达到此阈值时，传入查询将被拒绝。仅在设置大于 `0` 后生效。默认值为 `1024`。

### query_queue_mem_used_pct_limit（全局）

BE 上的内存使用百分比上限。仅在设置大于 `0` 后生效。默认值为 `0`。取值范围：[0, 1]

### query_queue_pending_timeout_second（全局）

队列中挂起查询的最大超时时间。当达到此阈值时，相应的查询将被拒绝。单位：秒。默认值为 `300`。

### query_timeout

用于设置查询的超时时间（以“秒”为单位）。此变量将作用于当前连接中的所有查询语句以及 INSERT 语句。默认值为 300 秒。有效取值范围为 [1, 259200]。

### range_pruner_max_predicate（v3.0 及更高版本）

可用于范围分区修剪的 IN 谓词的最大数量。默认值为 100。如果超过 100，可能会导致系统扫描所有平板电脑，从而影响查询性能。

### rewrite_count_distinct_to_bitmap_hll

用于决定是否将计数不同的查询重写为 bitmap_union_count 和 hll_union_agg。

### runtime_filter_on_exchange_node

在 GRF 通过 Exchange 运算符向下推送到较低级别的运算符后，是否将 GRF 放置在 Exchange 节点上。默认值为 `false`，这意味着 GRF 在通过 Exchange 运算符向下推送到较低级别的运算符后，不会放置在 Exchange 节点上。这样可以防止重复使用 GRF 并减少计算时间。

然而，GRF的交付是一个“尽力而为”的过程。如果较低级别的运算符无法接收 GRF，但 GRF 未放置在 Exchange 节点上，则无法过滤数据，从而影响过滤性能。`true` 意味着 GRF 仍将放置在 Exchange 节点上，即使它被下推到 Exchange 运营商的较低级别的运营商。

### runtime_join_filter_push_down_limit

Hash 表允许的最大行数，基于该表生成 Bloom filter Local RF。如果超过此值，则不会生成本地射频。此变量可防止产生过长的本地射频。

该值为整数。默认值为 1024000。

### runtime_profile_report_interval

报告运行时配置文件的时间间隔。此变量从 v3.1.0 开始支持。

单位：秒，默认值为 `10`。

### spill_mode（3.0 及更高版本）

中间结果溢出的执行模式。有效取值：

* `auto`：当达到内存使用阈值时，会自动触发溢出。
* `force`：StarRocks 强制执行所有相关算子的溢出，无论内存占用情况如何。

仅当变量 `enable_spill` 设置为 `true` 时，此变量才生效。

### SQL_AUTO_IS_NULL

用于与 JDBC 连接池 C3P0 兼容。没有实际用途。

### sql_dialect（v3.0 及更高版本）

所使用的 SQL 方言。例如，您可以运行 `set sql_dialect = 'trino';` 命令将 SQL 方言设置为 Trino，以便您可以在查询中使用特定于 Trino 的 SQL 语法和函数。

> **注意**
>
> 配置 StarRocks 后，默认查询中的标识符不区分大小写。因此，在创建数据库和表时，必须以小写形式为数据库和表指定名称。如果以大写形式指定数据库和表名称，则针对这些数据库和表的查询将失败。

### sql_mode

用于指定适应某些 SQL 方言的 SQL 模式。

### sql_safe_updates

用于 MySQL 客户端兼容性。没有实际用途。

### sql_select_limit

用于 MySQL 客户端兼容性。没有实际用途。

### statistic_collect_parallel

用于调整可在 BE 上运行的统计信息采集任务的并行度。默认值为 1。您可以增加此值以加快收集任务。

### storage_engine

StarRocks 支持的引擎类型：

* olap：StarRocks 系统自有引擎。
* mysql：MySQL 外部表。
* broker：通过 broker 程序访问外部表。
* elasticsearch 或 es：Elasticsearch 外部表。
* hive：Hive 外部表。
* iceberg：Iceberg 外部表，从 v2.1 开始支持。
* hudi：Hudi 外部表，从 v2.2 开始支持。
* jdbc：JDBC 兼容数据库的外部表，从 v2.3 开始支持。

### streaming_preaggregation_mode

用于指定 GROUP BY 第一阶段的预聚合模式。如果第一阶段的预聚合效果不理想，可以使用流式处理模式，即在将数据流式传输到目标端之前，先进行简单的数据序列化。有效取值：

* `auto`：系统首先尝试本地预聚合。如果效果不理想，则切换到流媒体模式。这是默认值。
* `force_preaggregation`：系统直接执行本地预聚合。
* `force_streaming`：系统直接执行流式处理。

### system_time_zone

用于显示当前系统的时区。无法更改。

### time_zone

用于设置当前会话的时区。时区可能会影响某些时间函数的结果。

### tx_isolation
用于保持与 MySQL 客户端的兼容性。没有实际的使用场景。

### use_compute_nodes

可使用的最大 CN 节点数。当 `prefer_compute_node=true` 时，此变量有效。有效取值：

* `-1`：表示使用所有 CN 节点。
* `0`：表示不使用任何 CN 节点。

### use_v2_rollup

用于控制使用 segment v2 存储格式的汇总索引来获取数据的查询。此变量用于在使用 segment v2 时进行验证。不建议用于其他情况。

### vectorized_engine_enable（从 v2.4 版本开始弃用）

用于控制是否使用矢量化引擎执行查询。取值为 `true` 表示使用矢量化引擎，否则使用非矢量化引擎。默认值为 `true`。此功能从 v2.4 版本开始默认启用，因此已弃用。

### version（全局）

返回给客户端的 MySQL 服务器版本。

### version_comment（全局）

StarRocks 版本。不可更改。

### wait_timeout

用于设置空闲连接的连接超时时间。当空闲连接在设定的时间内未与 StarRocks 交互时，StarRocks 将主动断开该连接。默认值为 8 小时，单位为秒。