---
displayed_sidebar: "Chinese"
---

# 系统变量

StarRocks提供了许多可以设置和修改以满足您需求的系统变量。本节描述了StarRocks支持的变量。您可以通过在MySQL客户端上运行[SHOW VARIABLES](../sql-reference/sql-statements/Administration/SHOW_VARIABLES.md)命令来查看这些变量的设置。您也可以使用[SET](../sql-reference/sql-statements/Administration/SET.md)命令动态设置或修改变量。您可以使这些变量在整个系统中全局生效，仅在当前会话中生效，或仅在单个查询语句中生效。

StarRocks中的变量指的是MySQL中的变量集，但是**某些变量仅与MySQL客户端协议兼容，在MySQL数据库上不起作用**。

> **注意**
>
> 任何用户都有权利运行SHOW VARIABLES并使变量在会话级别生效。但是，只有拥有SYSTEM级别OPERATE权限的用户才能使变量全局生效。全局生效的变量将影响所有未来的会话（不包括当前会话）。
>
> 如果您想要对当前会话进行设置更改，并且让该设置变更应用于所有未来会话，您可以进行两次更改，一次不使用`GLOBAL`修饰符，一次使用`GLOBAL`修饰符。例如：
>
> ```SQL
> SET query_mem_limit = 137438953472; -- 应用于当前会话。
> SET GLOBAL query_mem_limit = 137438953472; -- 应用于所有未来会话。
> ```

## 变量层次和类型

StarRocks支持三种类型（级别）的变量：全局变量、会话变量和`SET_VAR`提示。它们的层次关系如下：

* 全局变量在全局级别生效，并可被会话变量和`SET_VAR`提示覆盖。
* 会话变量仅在当前会话中生效，并可被`SET_VAR`提示覆盖。
* `SET_VAR`提示仅在当前查询语句中生效。

## 查看变量

您可以使用`SHOW VARIABLES [LIKE 'xxx']`来查看所有或一些变量。示例：

```SQL
-- 显示系统中的所有变量。
SHOW VARIABLES;

-- 显示匹配特定模式的变量。
SHOW VARIABLES LIKE '%time_zone%';
```

## 设置变量

### 全局设置变量或单个会话设置变量

您可以设置变量以全局生效或仅在当前会话中生效。当设置为全局时，新值将用于所有未来会话，而当前会话仍然使用原始值。当设置为"仅当前会话"时，变量仅在当前会话中生效。

通过`SET <var_name> = xxx;`设置的变量仅对当前会话生效。示例：

```SQL
SET query_mem_limit = 137438953472;

SET forward_to_master = true;

SET time_zone = "Asia/Shanghai";
```

使用`SET GLOBAL <var_name> = xxx;`设置的变量会全局生效。示例：

```SQL
SET GLOBAL query_mem_limit = 137438953472;
```

以下变量仅全局生效。它们无法在单个会话中生效，这意味着您必须使用`SET GLOBAL <var_name> = xxx;`来设置这些变量。若您尝试为单个会话设置这样的变量（`SET <var_name> = xxx;`），会返回错误。

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

此外，变量设置也支持常量表达式，例如：

```SQL
SET query_mem_limit = 10 * 1024 * 1024 * 1024;
```

 ```SQL
SET forward_to_master = concat('tr', 'u', 'e');
```

### 在单个查询语句中设置变量

在某些场景下，您可能需要为特定查询专门设置变量。通过使用`SET_VAR`提示，您可以设置会话变量，该变量仅在单个语句中生效。示例：

```sql
SELECT /*+ SET_VAR(query_mem_limit = 8589934592) */ name FROM people ORDER BY name;

SELECT /*+ SET_VAR(query_timeout = 1) */ sleep(3);
```

> **注意**
>
> `SET_VAR`只能放在`SELECT`关键字后面，并用`/*+...*/`括起来。

您还可以在单个语句中设置多个变量。示例：

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

变量按**字母顺序**描述。具有`global`标签的变量仅能在全局级别生效。其他变量可以在全局或单个会话中生效。

### activate_all_roles_on_login (global）

是否在StarRocks用户连接到StarRocks集群时启用用户的所有角色（包括默认角色和授予角色）。该变量自v3.0开始支持。

* 如果启用（true），用户的所有角色在用户登录时激活。这将覆盖由[SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md)设置的角色。
* 如果禁用（false），由SET DEFAULT ROLE设置的角色会被激活。

默认值：false。

如果您想要在会话中激活分配给您的角色，请使用[SET ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md)命令。

### auto_increment_increment

用于MySQL客户端兼容性。没有实际用途。

### autocommit

用于MySQL客户端兼容性。没有实际用途。

### batch_size

用于指定每个节点在查询执行过程中传输的单个数据包的行数。默认值为1024，即源节点生成的每1024行数据打包传输到目标节点。更多行数将提高大数据量场景中的查询吞吐量，但可能增加小数据量场景中的查询延迟。同时，可能增加查询的内存开销。我们建议将`batch_size`设置在1024到4096之间。

### big_query_profile_second_threshold（3.1及更高版本）

当会话变量`enable_profile`设置为`false`并且查询所花时间超过变量`big_query_profile_second_threshold`指定的阈值时，为该查询生成一个性能概要。

### cbo_decimal_cast_string_strict（2.5.14及更高版本）

控制CBO如何将DECIMAL类型的数据转换为STRING类型。如果此变量设置为`true`，则v2.5.x及更高版本内建的逻辑生效，系统执行严格转换（即，系统根据比例长度截断生成的字符串并填充0）。如果此变量设置为`false`，则v2.5.x之前版本内建的逻辑生效，系统处理所有有效数字以生成字符串。默认值为`true`。

### cbo_enable_low_cardinality_optimize

是否启用低基数优化。启用此功能后，查询STRING列的性能提高约三倍。默认值为true。

### cbo_eq_base_type（2.5.14及更高版本）

指定用于DECIMAL类型数据与STRING类型数据比较的数据类型。默认值为`VARCHAR`，DECIMAL也是有效值。

### character_set_database (global）

StarRocks支持的字符集。仅支持UTF8（`utf8`）。

### connector_io_tasks_per_scan_operator（2.5及更高版本）

在查询外部表期间，扫描运算符发出的最大并发I/O任务数。该值为整数。默认值：16。

目前，StarRocks可以在查询外部表时自适应调整并发I/O任务的数量。此功能由变量`enable_connector_adaptive_io_tasks`控制，默认情况下已启用。

### count_distinct_column_buckets（2.5及更高版本）

分组计数唯一列查询中，COUNT DISTINCT列的桶数。仅当`enable_distinct_column_bucketization`设置为`true`时，此变量生效。默认值：1024。

### default_rowset_type（global）

全局变量。用于设置计算节点存储引擎使用的默认存储格式。当前支持的存储格式是`alpha`和`beta`。

### default_table_compression（3.0及更高版本）

表存储的默认压缩算法。支持的压缩算法有`snappy, lz4, zlib, zstd`。默认值：lz4_frame。

请注意，如果在CREATE TABLE语句中指定了`compression`属性，则`compression`指定的压缩算法生效。

### disable_colocate_join
```
将用于控制是否启用迁移连接的 Colocation Join。默认值为 `false`，表示该功能已启用。禁用此功能后，查询规划将不会尝试执行 Colocation Join。

### disable_streaming_preaggregations

用于启用流式预聚合。默认值为 `false`，表示已启用。

### div_precision_increment

用于 MySQL 客户端兼容性。无实际用途。

### enable_connector_adaptive_io_tasks（2.5及更高版本）

在查询外部表时，是否自适应调整并发 I/O 任务数。默认值为 true。

如果未启用此功能，则可以通过变量 `connector_io_tasks_per_scan_operator` 手动设置并发 I/O 任务数。

### enable_distinct_column_bucketization（2.5及更高版本）

是否为 GROUP BY - COUNT DISTINCT 查询中的 COUNT DISTINCT 列启用分桶。使用 `select a, count(distinct b) from t group by a;` 查询作为示例。如果 GROUP BY 列 `a` 是低基数列，而 COUNT DISTINCT 列 `b` 是具有严重数据倾斜的高基数列，则会出现性能瓶颈。在这种情况下，可以将 COUNT DISTINCT 列中的数据分割为多个桶，以平衡数据并防止数据倾斜。

默认值为 false，表示未启用此功能。必须同时使用变量 `count_distinct_column_buckets`。

还可以通过在查询中添加 `skew` 提示来为 COUNT DISTINCT 列启用分桶，例如 `select a,count(distinct [skew] b) from t group by a;`。

### enable_group_level_query_queue（3.1.4及更高版本）

是否启用资源组级别 [查询队列](../administration/query_queues.md)。

默认值为 false，表示未启用此功能。

### enable_insert_strict

用于在使用 INSERT 语句加载数据时启用严格模式。默认值为 `true`，表示默认情况下已启用严格模式。有关更多信息，请参阅[严格模式](../loading/load_concept/strict_mode.md)。

### enable_materialized_view_union_rewrite（2.5及更高版本）

布尔值，用于控制是否启用物化视图 Union 查询重写。默认值为 `true`。

### enable_rule_based_materialized_view_rewrite（2.5及更高版本）

布尔值，用于控制是否启用基于规则的物化视图查询重写。此变量主要用于单表查询重写。默认值为 `true`。

### enable_spill（3.0及更高版本）

是否启用中间结果溢出。默认值为 `false`。如果设置为 `true`，StarRocks 会将中间结果溢出到磁盘，以减少查询中的聚合、排序或连接操作时的内存使用。

### enable_profile

指定是否发送查询的概要信息以进行分析。默认值为 `false`，表示不需要概要信息。

默认情况下，在 BE 发生查询错误时，才将概要信息发送到 FE。发送概要信息会导致网络开销，因此会影响高并发性能。

如果需要分析查询的概要信息，可以将此变量设置为 `true`。查询完成后，可以在当前连接的 FE 网页上查看概要信息（地址： `fe_host:fe_http_port/query`）。此页面显示了最近 100 个启用了 `enable_profile` 的查询的概要信息。

### enable_query_queue_load（全局）

布尔值，用于启用加载任务的查询队列。默认值为 `false`。

### enable_query_queue_select（全局）

布尔值，用于启用 SELECT 查询的查询队列。默认值为 `false`。

### enable_query_queue_statistic（全局）

布尔值，用于启用统计查询的查询队列。

### enable_query_tablet_affinity（2.5及更高版本）

布尔值，用于控制是否将对同一 tablet 的多个查询定向到固定的副本。

在需要查询的表有大量 tablet 的情况下，此功能显著提高了查询性能，因为可以更快地将 tablet 的元信息和数据缓存在内存中。

但是，如果存在一些热点 tablet，此功能可能会降低查询性能，因为它会将查询定向到同一 BE，使其无法充分利用高并发场景中多个 BE 的资源。

默认值为 `false`，表示系统为每个查询选择一个副本。自 2.5.6、3.0.8、3.1.4 和 3.2.0 版开始支持此功能。

### enable_scan_datacache（2.5及更高版本）

指定是否启用数据缓存功能。启用此功能后，StarRocks 会将从外部存储系统读取的热数据缓存到块中，从而加快查询和分析速度。有关更多信息，请参阅[数据缓存](../data_source/data_cache.md)。在 3.2 版本之前，此变量的名称为 `enable_scan_block_cache`。

### enable_populate_datacache（2.5及更高版本）

指定是否将从外部存储系统读取的数据块缓存到 StarRocks 中。如果不希望缓存从外部存储系统读取的数据块，可将此变量设置为 `false`。默认值为 true。此变量自 2.5 版本起受支持。在 3.2 版本之前，此变量的名称为 `enable_scan_block_cache`。

### enable_tablet_internal_parallel（2.3及更高版本）

是否启用自适应并行扫描 tablet。启用此功能后，可以使用多个线程按段扫描一个 tablet，增加了扫描并发性。默认值为 true。

### enable_query_cache（2.5及更高版本）

指定是否启用查询缓存功能。有效值为 true 和 false。`true` 表示启用此功能，`false` 表示禁用此功能。启用此功能后，仅适用于[查询缓存](../using_starrocks/query_cache.md#application-scenarios)的应用场景中指定条件的查询。

### enable_adaptive_sink_dop（2.5及更高版本）

指定是否启用数据加载的自适应并行性。启用此功能后，系统会自动为 INSERT INTO 和 Broker Load 作业设置加载并行度，相当于 `pipeline_dop` 的机制。对于新部署的 v2.5 StarRocks 集群，默认值为 `true`。对于从 v2.4 升级到 v2.5 的 v2.5 集群，默认值为 `false`。

### enable_pipeline_engine

指定是否启用管道执行引擎。`true` 表示已启用，`false` 表示相反。默认值为 `true`。

### enable_sort_aggregate（2.5及更高版本）

指定是否启用排序流式处理。`true` 表示已启用排序流式处理以对数据流进行排序。

### enable_global_runtime_filter

是否启用全局运行时过滤器（RF）。RF 在运行时过滤数据。数据过滤通常发生在 Join 阶段。在多表连接中，会使用谓词下推等优化来过滤数据，以减少 Join 的扫描行数和 Shuffle 阶段的 I/O，从而加速查询。

StarRocks 提供两种 RF：局部 RF 和全局 RF。局部 RF 适用于广播哈希连接，全局 RF 适用于 Shuffle 连接。

默认值为 `true`，表示已启用全局 RF。如果禁用此功能，全局 RF 将不生效。局部 RF 仍可正常工作。

### enable_multicolumn_global_runtime_filter

是否启用多列全局运行时过滤器。默认值为 `false`，表示多列全局 RF 未启用。

如果一个 Join（不包括广播 Join 和复制 Join）具有多个等值连接条件：

* 如果禁用此功能，只有局部 RF 生效。
* 如果启用此功能，将启用多列全局 RF，并在分区 by 子句中带有 `multi-column`。

### ENABLE_WRITE_HIVE_EXTERNAL_TABLE（v3.2及更高版本）

是否允许将数据下沉到 Hive 的外部表。默认值为 `false`。

### event_scheduler

用于 MySQL 客户端兼容性。无实际用途。

### enable_strict_type（v3.1及更高版本）

是否允许对 WHERE 子句中的所有复合谓词和所有表达式进行隐式转换。默认值为 `false`。

### force_streaming_aggregate

用于控制聚合节点是否启用流式聚合计算。默认值为 false，表示未启用此功能。

### forward_to_master

用于指定是否将某些命令转发到主 FE 进行执行。默认值为 `false`，表示不转发到主 FE。StarRocks 集群中有多个 FE，其中一个是主 FE。通常情况下，用户可以连接任何 FE 进行全功能操作。但是，某些信息仅在主 FE 上可用。

例如，如果不将 SHOW BACKENDS 命令转发到主 FE，则只能查看基本信息（例如节点是否在线）。转发到主 FE 可以获取更详细的信息，包括节点启动时间和最后心跳时间。

受此变量影响的命令包括：

* SHOW FRONTENDS：转发到主 FE 允许用户查看最后心跳消息。

* SHOW BACKENDS：转发到主 FE 允许用户查看启动时间、最后心跳信息和磁盘容量信息。
```
* 显示经纪人：转发给主领导允许用户查看启动时间和最后心跳信息。

* 显示平板电脑

* ADMIN SHOW REPLICA DISTRIBUTION

* ADMIN SHOW REPLICA STATUS：转发给主领导允许用户查看存储在主领导元数据中的平板信息。通常情况下，不同前端服务器的元数据中的平板信息应该是相同的。如果出现错误，您可以使用此方法比较当前前端服务器和主领导的元数据。

* 显示 PROC：转发到主领导允许用户查看存储在元数据中的 PROC 信息。这主要用于元数据比较。

### group_concat_max_len

由 [group_concat](../sql-reference/sql-functions/string-functions/group_concat.md) 函数返回的字符串的最大长度。默认值：1024。最小值：4。单位：字符。

### hash_join_push_down_right_table

用于控制是否可以使用加入查询中右表的过滤条件来过滤左表的数据。如果可以，可以减少查询时需要处理的数据量。

`true` 表示允许该操作，并且系统决定是否可以对左表进行过滤。`false` 表示该操作被禁用。默认值为 `true`。

### init_connect (global)

用于 MySQL 客户端兼容性。无实际用途。

### interactive_timeout

用于 MySQL 客户端兼容性。无实际用途。

### io_tasks_per_scan_operator (2.5 及更高)

扫描操作符可以发出的并发 I/O 任务数。如果要访问诸如 HDFS 或 S3 等远程存储系统，但延迟很高，可以增加该值。但更大的值会导致更多的内存消耗。

该值为整数。默认值：4。

### language (global)

用于 MySQL 客户端兼容性。无实际用途。

### 执照（全局）

显示 StarRocks 的许可证。

### load_mem_limit

指定导入操作的内存限制。默认值为 0，意味着此变量未被使用，而是使用 `query_mem_limit`。

此变量仅用于涉及查询和导入两者的 `INSERT` 操作。如果用户没有设置此变量，将会为查询和导入的内存限制分别设置为 `exec_mem_limit`。否则，将为查询的内存限制设置为 `exec_mem_limit`，将导入的内存限制设置为 `load_mem_limit`。

其他导入方法，例如 `BROKER LOAD`、`STREAM LOAD` 仍然使用 `exec_mem_limit` 作为内存限制。

### log_rejected_record_num（v3.1 及更高）

指定可以记录的不合格数据行的最大数量。有效值：`0`、`-1` 和任何非零正整数。默认值：`0`。

* 值 `0` 指定被过滤掉的数据行不会被记录。
* 值 `-1` 指定所有被过滤掉的数据行都会被记录。
* 诸如 `n` 等非零正整数指定每个后端服务器可以记录的最多 `n` 个被过滤掉的数据行。

### lower_case_table_names (global)

用于 MySQL 客户端兼容性。在 StarRocks 中，表名区分大小写。

### materialized_view_rewrite_mode (v3.2 及更高)

指定异步物化视图的查询重写模式。有效值：

* `disable`：禁用异步物化视图的自动查询重写。
* `default`（默认值）：启用异步物化视图的自动查询重写，并允许优化器根据成本来决定是否可以使用物化视图重写查询。如果查询无法被重写，它将直接扫描基表中的数据。
* `default_or_error`：启用异步物化视图的自动查询重写，并允许优化器根据成本来决定是否可以使用物化视图重写查询。如果查询无法被重写，将返回错误。
* `force`：启用异步物化视图的自动查询重写，并使优化器优先使用物化视图重写查询。如果查询无法被重写，它将直接扫描基表中的数据。
* `force_or_error`：启用异步物化视图的自动查询重写，并使优化器优先使用物化视图重写查询。如果查询无法被重写，将返回错误。

### max_allowed_packet

用于与 JDBC 连接池 C3P0 的兼容性。此变量指定客户端和服务器之间可传输的数据包的最大大小。默认值：32 MB。单位：字节。如果客户端报告“PacketTooBigException”，可以提高此值。

### max_scan_key_num

每个查询分段的最大扫描键数。默认值：-1，表示使用 `be.conf` 文件中的值。如果此变量设置为大于 0 的值，则会忽略 `be.conf` 中的值。

### max_pushdown_conditions_per_column

可为列推送的谓词的最大数量。默认值：-1，表示使用 `be.conf` 文件中的值。如果此变量设置为大于 0 的值，则会忽略 `be.conf` 中的值。

### nested_mv_rewrite_max_level

可以用于查询重写的嵌套物化视图的最大层数。类型：INT。范围：[1, +∞)。值 `1` 表示仅可以用于查询重写的物化视图是在基表上创建的物化视图。默认值：`3`。

### net_buffer_length

用于 MySQL 客户端兼容性。无实际用途。

### net_read_timeout

用于 MySQL 客户端兼容性。无实际用途。 

### net_write_timeout

用于 MySQL 客户端兼容性。无实际用途。

### new_planner_optimize_timeout

查询优化器的超时时长。当优化器超时时，将返回错误并停止查询，从而影响查询性能。可以根据查询设置此变量为较大的值，或者联系 StarRocks 技术支持进行故障排除。当查询涉及太多连接时，会经常出现超时。

默认值：3000。单位：毫秒。

### parallel_exchange_instance_num

用于设置执行计划中上层节点用于从下层节点接收数据的交换节点数。默认值为 -1，表示交换节点的数量与下层节点的执行实例数量相等。当此变量设置为大于 0 但小于下层节点的执行实例数量时，交换节点的数量等于设置的值。

在分布式查询执行计划中，上层节点通常有一个或多个交换节点接收来自不同前端服务器上下层节点的数据。通常交换节点的数量与下层节点的执行实例数量相等。

在某些聚合查询场景中，如在聚合后数据量急剧减少情况下，您可以尝试将此变量修改为较小的值，以减少资源开销。例如，在使用重复键表运行聚合查询时。

### parallel_fragment_exec_instance_num

用于设置在每个 BE 上用于扫描节点的实例数。默认值为 1。

查询计划通常会生成一组扫描范围。这些数据分布在多个 BE 节点上。每个 BE 节点将有一个或多个扫描范围，而默认情况下，每个 BE 节点的一组扫描范围都将由一个执行实例处理。当机器资源充足时，您可以增加此变量，以允许更多执行实例同时处理扫描范围，以提高效率。

扫描实例的数量决定了上层的其他执行节点的数量，例如聚合节点和连接节点。因此，它增加了整个查询计划执行的并发性。修改此变量将有助于提高效率，但更大的值将消耗更多的机器资源，如 CPU、内存和磁盘 IO。

### partial_update_mode（3.1 及更高）

用于控制部分更新的模式。有效值：

* `auto`（默认）：系统通过分析 UPDATE 语句和涉及的列自动确定部分更新的模式。
* `column`：用于部分更新的列模式，特别适用于涉及少量列和大量行的部分更新。

有关更多信息，请参阅 [UPDATE](../sql-reference/sql-statements/data-manipulation/UPDATE.md#partial-updates-in-column-mode-since-v31)。

### performance_schema

用于与 MySQL JDBC 版本 8.0.16 及以上版本的兼容性。无实际用途。

### prefer_compute_node

指定 FEs 是否将查询执行计划分发给 CN 节点。有效值：

* true：表示 FEs 将查询执行计划分发给 CN 节点。
* false：表示 FEs 不将查询执行计划分发给 CN 节点。

### pipeline_dop

管道实例的并行度，用于调整查询并发性。默认值：0，表示系统自动调整每个管道实例的并行度。您还可以将此变量设置为大于 0 的值。通常，将该值设置为物理 CPU 核心数的一半。

从 v3.0 起，StarRocks会根据查询并发度自适应调整此变量。

### pipeline_profile_level

控制查询配置文件的级别。查询配置文件通常有五个级别: Fragment、FragmentInstance、Pipeline、PipelineDriver 和 Operator。不同的级别提供配置文件的不同详细信息：

* 0：StarRocks结合配置文件的指标，仅显示少量核心指标。
* 1：默认值。StarRocks简化配置文件并合并配置文件的指标，以减少配置文件级别。
* 2：StarRocks保留配置文件的所有级别。在SQL查询复杂时，配置文件的大小会较大。不推荐设置此值。

### query_cache_entry_max_bytes (v2.5及更高版本)

触发穿越模式的阈值。有效值: 0 到 9223372036854775807。当由查询访问的特定table的计算结果的字节数或行数超过`query_cache_entry_max_bytes`或`query_cache_entry_max_rows`指定的阈值时，查询将切换到穿越模式。

### query_cache_entry_max_rows (v2.5及更高版本)

可缓存的行数上限。请参阅`query_cache_entry_max_bytes`中的描述。默认值: 409600。

### query_cache_agg_cardinality_limit (v2.5及更高版本)

在Query Cache中用于GROUP BY的基数上限。如果由GROUP BY生成的行数超过此值，将不启用Query Cache。默认值: 5000000。如果`query_cache_entry_max_bytes`或`query_cache_entry_max_rows`设置为0，则即使参与的table未生成任何计算结果，也将使用穿越模式。

### query_cache_size (全局)

用于MySQL客户端兼容性。没有实际用途。

### query_cache_type

用于与JDBC连接池C3P0兼容。没有实际用途。

### query_mem_limit

用于设置每个后端节点的查询内存限制。默认值为0，表示无限制。支持的单位包括`B/K/KB/M/MB/G/GB/T/TB/P/PB`。

当出现`Memory Exceed Limit`错误时，可以尝试增加此变量的值。

### query_queue_concurrency_limit (全局)

后端节点上并发查询的上限。仅在设置大于`0`后生效。默认值: `0`。

### query_queue_cpu_used_permille_limit (全局)

后端节点上CPU使用量千分比的上限（CPU使用量 * 1000）。仅在设置大于`0`后生效。默认值: `0`。范围: [0, 1000]

### query_queue_max_queued_queries (全局)

队列中的查询上限。达到此阈值后，将拒绝新的查询。仅在设置大于`0`后生效。默认值: `1024`。

### query_queue_mem_used_pct_limit (全局)

后端节点上内存使用率的上限。仅在设置大于`0`后生效。默认值: `0`。范围: [0, 1]

### query_queue_pending_timeout_second (全局)

队列中挂起查询的最大超时时间。达到此阈值后，对应的查询将被拒绝。单位: 秒。默认值: `300`。

### query_timeout

用于设置查询的超时时间，单位为"秒"。此变量将应用于当前连接中的所有查询语句，以及INSERT语句。默认值为300秒。值范围: [1, 259200]。

### range_pruner_max_predicate (v3.0及更高版本)

用于Range分区修剪的最大IN预算数。默认值: 100。如果超过100，系统将扫描所有table，从而影响查询性能。

### rewrite_count_distinct_to_bitmap_hll

用于决定是否重写count distinct查询为bitmap_union_count和hll_union_agg。

### runtime_filter_on_exchange_node

在GRF被推送至Exchange operator的下级operator后，是否将GRF放置在Exchange Node上。默认值为`false`，这意味着GRF被推送至Exchange operator的下级operator后不会放置在Exchange Node上。这可以防止GRF的重复使用并减少计算时间。

但是，GRF的传送是一个“最佳尝试”过程。如果下级operator未能接收到GRF，但GRF未放置在Exchange Node上，则无法过滤数据，这会影响过滤性能。`true`表示GRF仍将被放置在Exchange Node上，即使它已被推送至Exchange operator的下级operator。

### runtime_join_filter_push_down_limit

基于构建Bloom filter本地RF的Hash表所允许的最大行数。如果超出此值，则不会生成本地RF。此变量防止生成过长的本地RF。

该值为整数。默认值: 1024000。

### runtime_profile_report_interval

运行时配置文件上报间隔。此变量从v3.1.0开始支持。

单位: 秒，默认值: `10`。

### spill_mode (v3.0及更高版本)

中间结果溢出的执行模式。有效值:

* `auto`：当内存使用量阈值达到时，将自动触发溢出。
* `force`：无论内存使用情况如何，StarRocks都会强制对所有相关operator执行溢出。

此变量仅在变量`enable_spill`设置为`true`时生效。

### SQL_AUTO_IS_NULL

用于与JDBC连接池C3P0兼容。没有实际用途。

### sql_dialect (v3.0及更高版本)

使用的SQL方言。例如，可以运行`set sql_dialect = 'trino';`命令将SQL方言设置为Trino，从而可以在查询中使用Trino特定的SQL语法和函数。

> **注意**
>
> 在配置StarRocks使用Trino方言后，默认情况下查询中的标识符不区分大小写。因此，在数据库和表的创建时必须以小写指定名称。如果指定的数据库和表名称为大写，则对这些数据库和表的查询将失败。

### sql_mode

用于指定适应特定SQL方言的SQL模式。

### sql_safe_updates

用于与MySQL客户端兼容性。没有实际用途。

### sql_select_limit

用于与MySQL客户端兼容性。没有实际用途。

### statistic_collect_parallel

用于调整可以在BE上运行的统计信息收集任务的并行性。默认值: 1。可以增加此值以加快收集任务的速度。

### storage_engine

StarRocks支持的引擎类型:

* olap: StarRocks系统内建引擎。
* mysql: MySQL外部table。
* broker: 通过broker程序访问外部table。
* elasticsearch或es: Elasticsearch外部table。
* hive: Hive外部table。
* iceberg: Iceberg外部table，从v2.1开始支持。
* hudi: Hudi外部table，从v2.2开始支持。
* jdbc: 用于与兼容JDBC的数据库的外部table，从v2.3开始支持。

### streaming_preaggregation_mode

用于指定GROUP BY首阶段的预聚合模式。如果第一阶段的预聚合效果不理想，可以使用streaming模式，在流式数据发送到目标前对数据进行简单序列化。有效值:

* `auto`：系统首先尝试本地预聚合。如果效果不理想，则切换到streaming模式。这是默认值。
* `force_preaggregation`：系统直接执行本地预聚合。
* `force_streaming`：系统直接执行streaming。

### system_time_zone

用于显示当前系统的时区。不可更改。

### time_zone

用于设置当前会话的时区。时区可以影响某些时间函数的结果。

### tx_isolation

用于与MySQL客户端兼容性。没有实际用途。

### use_compute_nodes

可使用的CN节点的最大数量。当`prefer_compute_node=true`时，此变量有效。有效值:

* `-1`：表示使用所有CN节点。
* `0`：表示不使用CN节点。

### use_v2_rollup

用于控制使用v2段存储格式的rollup索引提取数据的查询。此变量用于验证上线v2段时。不建议在其他情况下使用。

### vectorized_engine_enable (v2.4版本弃用)

用于控制是否使用矢量化引擎执行查询。值为`true`表示使用矢量化引擎，否则使用非矢量化引擎。默认情况下为`true`。从v2.4及更高版本开始，此功能默认启用，因此已弃用。

### version (全局)

MySQL服务器版本返回给客户端。

### version_comment (全局)

StarRocks版本。不可更改。

### wait_timeout

用于设置空闲连接的连接超时时间。当空闲连接在此时间内不与StarRocks交互时，StarRocks将主动断开连接。默认值为8小时，以秒为单位。