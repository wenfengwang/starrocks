---
displayed_sidebar: "English"
---

# 参数配置

本主题介绍了FE、BE和系统参数。还提供建议如何配置和调整这些参数。

## FE配置项

FE参数分为动态参数和静态参数。

- 可以通过运行SQL命令配置和调整动态参数，非常方便。但是如果重新启动FE，则配置将失效。因此，建议您也修改`fe.conf`文件中的配置项，以防修改丢失。

- 静态参数只能在FE配置文件**fe.conf**中配置和调整。**修改该文件后，必须重新启动FE才能生效。**

参数是否为动态参数在[ADMIN SHOW CONFIG](../zh-cn/sql-reference/sql-statements/Administration/ADMIN_SHOW_CONFIG.md)的输出中的`IsMutable`列中指示。`TRUE`表示动态参数。

请注意，动态和静态FE参数均可以在**fe.conf**文件中配置。

### 查看FE配置项

启动FE后，可以在MySQL客户端上运行ADMIN SHOW FRONTEND CONFIG命令来检查参数配置。如果要查询特定参数的配置，请运行以下命令：

```SQL
 ADMIN SHOW FRONTEND CONFIG [LIKE "pattern"];
 ```

有关返回字段的详细描述，请参见[ADMIN SHOW CONFIG](../zh-cn/sql-reference/sql-statements/Administration/ADMIN_SHOW_CONFIG.md)。

> **注意**
>
> 运行与集群管理相关的命令，需要具有管理员权限。

### 配置FE动态参数

您可以使用[ADMIN SET FRONTEND CONFIG](../zh-cn/sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md)来配置或修改FE动态参数的设置。

```SQL
ADMIN SET FRONTEND CONFIG ("key" = "value");
```

> **注意**
>
> 在FE重新启动后，配置将恢复为`fe.conf`文件中的默认值。因此，我们建议您也修改`fe.conf`中的配置项，以防修改丢失。

#### 日志

##### qe_slow_log_ms

- 单位：ms
- 默认值：5000
- 描述：用于确定查询是否为慢查询的阈值。如果查询的响应时间超过此阈值，则在`fe.audit.log`中将其记录为慢查询。

#### 元数据和集群管理

##### catalog_try_lock_timeout_ms

- **单位**：ms
- **默认值**：5000
- **描述**：获取全局锁的超时时长。

##### edit_log_roll_num

- **单位**：-
- **默认值**：50000
- **描述**：在创建用于控制日志文件大小的log文件之前可以写入的最大元数据日志条目数。新的日志文件将写入BDBJE数据库。

##### ignore_unknown_log_id

- **单位**：-
- **默认值**：FALSE
- **描述**：是否忽略未知的日志ID。当进行FE回滚时，较早版本的BE可能无法识别某些日志ID。如果值为`TRUE`，则FE会忽略未知的日志ID。如果值为`FALSE`，则FE会退出。

##### ignore_materialized_view_error

- **单位**：-
- **默认值**：FALSE
- **描述**：FE是否忽略由物化视图错误引起的元数据异常。如果由于物化视图错误引起的元数据异常导致FE无法启动，您可以将此参数设置为`true`以允许FE忽略异常。此参数从v2.5.10版本开始支持。

##### ignore_meta_check

- **单位**：-
- **默认值**：FALSE
- **描述**：非领导者FE是否忽略来自领导者FE的元数据间隙。如果该值为TRUE，则非领导者FE将忽略来自领导者FE的元数据间隙并继续提供数据读取服务。此参数可确保即使在长时间停止领导者FE时，也能持续提供数据读取服务。如果该值为FALSE，则非领导者FE不会忽略来自领导者FE的元数据间隙并停止提供数据读取服务。

##### meta_delay_toleration_second

- **单位**：s
- **默认值**：300
- **描述**：追随者和观察者FE的元数据可以滞后于领导者FE的最长持续时间。单位：秒。如果超过此持续时间，非领导者FE将停止提供服务。

##### drop_backend_after_decommission

- **单位**：-
- **默认值**：TRUE
- **描述**：BE是否在被停用后删除。`TRUE`表示BE在被停用后立即被删除。`FALSE`表示BE在被停用后不会被删除。

##### enable_collect_query_detail_info

- **单位**：-
- **默认值**：FALSE
- **描述**：是否收集查询的详细信息。如果此参数设置为`TRUE`，系统将收集查询的详细信息。如果此参数设置为`FALSE`，系统将不收集查询的详细信息。

##### enable_background_refresh_connector_metadata

- **单位**：-
- **默认值**：v3.0中为`true`<br />v2.5中为`false`
- **描述**：是否启用定期的Hive元数据缓存刷新。启用后，StarRocks轮询您的Hive集群的元数据存储库（Hive元数据存储库或AWS Glue），并刷新频繁访问的Hive目录的缓存的元数据，以感知数据更改。`true`表示启用Hive元数据缓存刷新，`false`表示禁用。此参数从v2.5.5版本开始支持。

##### background_refresh_metadata_interval_millis

- **单位**：ms
- **默认值**：600000
- **描述**：两次连续Hive元数据缓存刷新之间的间隔。此参数从v2.5.5版本开始支持。

##### background_refresh_metadata_time_secs_since_last_access_secs

- **单位**：s
- **默认值**：86400
- **描述**：Hive元数据缓存刷新任务的过期时间。对于已访问的Hive目录，如果超过指定时间未访问，则StarRocks将停止刷新其缓存的元数据。对于未访问的Hive目录，StarRocks将不刷新其缓存的元数据。此参数从v2.5.5版本开始支持。

##### enable_statistics_collect_profile

- **单位**：N/A
- **默认值**：false
- **描述**：是否为系统统计查询生成配置文件。您可以将此项设置为`true`，以允许StarRocks为系统统计查询生成查询配置文件。此参数从v3.1.5版本开始支持。

#### 查询引擎

##### max_allowed_in_element_num_of_delete

- 单位：-
- 默认值：10000
- 描述：DELETE语句中允许的IN谓词的最大元素数。

##### enable_materialized_view

- 单位：-
- 默认值：TRUE
- 描述：是否启用物化视图的创建。

##### enable_decimal_v3

- 单位：-
- 默认值：TRUE
- 描述：是否支持DECIMAL V3数据类型。

##### enable_sql_blacklist

- 单位：-
- 默认值：FALSE
- 描述：是否启用SQL查询的黑名单检查。启用此功能后，黑名单中的查询将无法执行。

##### dynamic_partition_check_interval_seconds

- 单位：s
- 默认值：600
- 描述：检查新数据的间隔。如果检测到新数据，StarRocks将自动为数据创建分区。

##### dynamic_partition_enable

- 单位：-
- 默认值：TRUE
- 描述：是否启用动态分区功能。启用此功能后，StarRocks将为新数据动态创建分区，并自动删除过期分区，以确保数据的新鲜度。

##### http_slow_request_threshold_ms

- 单位：ms
- 默认值：5000
- 描述：如果HTTP请求的响应时间超过此参数指定的值，系统将生成日志以跟踪此请求。
- 引入于：2.5.15，3.1.5

##### max_partitions_in_one_batch

- 单位：-
- 默认值：4096
- 描述：在批量创建分区时可创建的最大分区数。

##### max_query_retry_time

- 单位：-
- 默认值：2
- 描述：FE上查询的最大重试次数。

##### max_create_table_timeout_second

- 单位：s
- 默认值：600
- 描述：创建表的最大超时时长。

##### create_table_max_serial_replicas

- 单位：-
- 默认值：128
- 描述：顺序创建的最大副本数。如果实际副本数超过此值，则副本将并行创建。如果表的创建时间太长，请尝试减少此配置。

##### max_running_rollup_job_num_per_table

- 单位：-
- 默认值：1
- 描述：每个表可以并行运行的最大Rollup作业数。

##### max_planner_scalar_rewrite_num

- 单位：-
- 默认值：100000
- 描述：优化器可以重新编写标量运算符的最大次数。

##### enable_statistic_collect

- 单位：-
- 默认值：TRUE
- 描述：是否为 CBO 收集统计信息，默认情况下启用此功能。

##### enable_collect_full_statistic

- 单位：-
- 默认值：TRUE
- 描述：是否启用自动收集完整统计信息，默认情况下启用该功能。

##### statistic_auto_collect_ratio

- 单位：-
- 默认值：0.8
- 描述：用于确定自动收集的统计信息是否健康的阈值。如果统计信息健康度低于该阈值，则触发自动收集。

##### statistic_max_full_collect_data_size

- 单位：LONG
- 默认值：107374182400
- 描述：自动收集统计信息时最大分区的大小（以字节为单位）。如果某个分区超过此值，则执行抽样收集而非完整收集。

##### statistic_collect_max_row_count_per_query

- 单位：INT
- 默认值：5000000000
- 描述：用于单个分析任务查询的最大行数。如果超出此值，将把分析任务拆分为多个查询。

##### statistic_collect_interval_sec

- 单位：秒
- 默认值：300
- 描述：自动收集期间检查数据更新的间隔。

##### statistic_auto_analyze_start_time

- 单位：STRING
- 默认值：00:00:00
- 描述：自动收集的开始时间。取值范围：`00:00:00` - `23:59:59`。

##### statistic_auto_analyze_end_time

- 单位：STRING
- 默认值：23:59:59
- 描述：自动收集的结束时间。取值范围：`00:00:00` - `23:59:59`。

##### statistic_sample_collect_rows

- 单位：-
- 默认值：200000
- 描述：采样收集的最小行数。若参数值超过数据表中的实际行数，则执行完整收集。

##### histogram_buckets_size

- 单位：-
- 默认值：64
- 描述：直方图的默认桶数。

##### histogram_mcv_size

- 单位：-
- 默认值：100
- 描述：直方图中最常见值（MCV）的个数。

##### histogram_sample_ratio

- 单位：-
- 默认值：0.1
- 描述：直方图的采样比例。

##### histogram_max_sample_row_count

- 单位：-
- 默认值：10000000
- 描述：直方图的最大收集行数。

##### statistics_manager_sleep_time_sec

- 单位：秒
- 默认值：60
- 描述：元数据调度的间隔。系统根据此间隔执行以下操作：
  - 创建用于存储统计信息的表。
  - 删除已删除的统计信息。
  - 删除已过期的统计信息。

##### statistic_update_interval_sec

- 单位：秒
- 默认值：`24 * 60 * 60`
- 描述：统计信息缓存更新的间隔。单位：秒。

##### statistic_analyze_status_keep_second

- 单位：秒
- 默认值：259200
- 描述：保留收集任务历史记录的持续时间。默认值为3天。

##### statistic_collect_concurrency

- 单位：-
- 默认值：3
- 描述：允许并行运行的手动收集任务的最大数量。默认值为3，表示最多可以并行运行三个手动收集任务。若超出该值，新任务将处于待定状态，等待调度。

##### statistic_auto_collect_small_table_rows

- 单位：-
- 默认值：10000000
- 描述：自动收集期间确定外部数据源（Hive、Iceberg、Hudi）中表是否为小表的阈值。若表行数低于此值，则被视为小表。
- 引入版本：v3.2

##### enable_local_replica_selection

- 单位：-
- 默认值：FALSE
- 描述：是否为查询选择本地副本。本地副本可降低网络传输成本。若将此参数设置为 TRUE，则 CBO 会优先选择与当前 FE 具有相同 IP 地址的 BE 上的 Tablet 副本。若将此参数设置为 `FALSE`，则可以选择本地副本和非本地副本。默认值为 FALSE。

##### max_distribution_pruner_recursion_depth

- 单位：-
- 默认值：100
- 描述：分区修剪器允许的最大递归深度。增加递归深度可修剪更多元素，但也会增加 CPU 消耗。

##### enable_udf

- 单位：-
- 默认值：FALSE
- 描述：是否启用 UDF。

#### 载入和卸载

##### max_broker_load_job_concurrency

- **单位**：-
- **默认值**：5
- **描述**：StarRocks 集群内允许的最大并发 Broker Load 作业数。此参数仅对 Broker Load 有效。此参数的值必须小于 `max_running_txn_num_per_db` 的值。从 v2.5 开始，默认值从 `10` 更改为 `5`。此参数的别名为 `async_load_task_pool_size`。

##### load_straggler_wait_second

- **单位**：秒
- **默认值**：300
- **描述**：BE 副本可容忍的最大加载滞后时间。若超出此值，会执行复制操作，从其他副本克隆数据。单位：秒。

##### desired_max_waiting_jobs

- **单位**：-
- **默认值**：1024
- **描述**：FE 中的最大挂起作业数。该数字涵盖所有作业，如表创建、载入和模式更改作业。若 FE 中的挂起作业数达到此值，FE 将拒绝新的加载请求。此参数仅对异步加载有效。从 v2.5 开始，默认值从 100 更改为 1024。

##### max_load_timeout_second

- **单位**：秒
- **默认值**：259200
- **描述**：载入作业允许的最大超时持续时间。若超出此限制，载入作业将失败。此限制适用于所有类型的载入作业。

##### min_load_timeout_second

- **单位**：秒
- **默认值**：1
- **描述**：载入作业允许的最小超时持续时间。此限制适用于所有类型的载入作业。单位：秒。

##### max_running_txn_num_per_db

- **单位**：-
- **默认值**：100
- **描述**：StarRocks 集群内每个数据库允许运行的最大载入事务数。默认值为 `100`。若某个数据库运行的载入事务实际数量超过此参数的值，则不会处理新的载入请求。对同步载入的新请求将被拒绝，并将异步载入的新请求置于队列中。我们不建议增加此参数的值，因为会增加系统负载。

##### load_parallel_instance_num

- **单位**：-
- **默认值**：1
- **描述**：BE 上每个载入作业允许的最大并发载入数。

##### disable_load_job

- **单位**：-
- **默认值**：FALSE
- **描述**：集群遇到错误时是否禁用载入。这可以防止集群错误造成的任何损失。默认值为 `FALSE`，表示未禁用载入。

##### history_job_keep_max_second

- **单位**：秒
- **默认值**：604800
- **描述**：保留历史作业（如模式更改作业）的最大持续时间，以秒计。

##### label_keep_max_num

- **单位**：-
- **默认值**：1000
- **描述**：在一定时间段内可以保留的载入作业的最大数量。若超出此数量，将删除历史作业的信息。

##### label_keep_max_second

- **单位**：秒
- **默认值**：259200
- **描述**：对于已完成并处于 FINISHED 或 CANCELLED 状态的载入作业标签，可以保留的最大持续时间，以秒计。默认值为3天。超过此持续时间后，标签将被删除。此参数适用于所有类型的载入作业。过大的值会消耗大量内存。

##### max_routine_load_job_num

- **单位**：-
- **默认值**：100
- **描述**：StarRocks 集群中的最大 Routine Load 作业数。自 v3.1.0 起废弃该参数。

##### max_routine_load_task_concurrent_num

- **单位**：-
- **默认值**：5
- **描述**：每个 Routine Load 作业的最大并发任务数。

##### max_routine_load_task_num_per_be

- **单位**：-
- **默认值**：16
- **描述**：每个 BE 上的最大并发 Routine Load 任务数。自 v3.1.0 起，默认值从 5 调整为 16，并且不再需要小于或等于 BE 静态参数 `routine_load_thread_pool_size` 的值（已废弃）。

##### max_routine_load_batch_size

- **单位**：字节
- **默认值**：4294967296
- **描述**：每个 Routine Load 任务允许加载的最大数据量，以字节为单位。

##### routine_load_task_consume_second

- **单位**: s
- **默认**: 15
- **描述**: 集群中每个常规加载任务消耗数据的最长时间。自v3.1.0版本开始，常规加载作业支持[作业属性](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md#job_properties)中的新参数`task_consume_second`。该参数适用于常规加载作业中的单个加载任务，更加灵活。

##### routine_load_task_timeout_second

- **单位**: s
- **默认**: 60
- **描述**: 集群中每个常规加载任务的超时持续时间。自v3.1.0版本开始，常规加载作业支持[作业属性](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md#job_properties)中的新参数`task_timeout_second`。该参数适用于常规加载作业中的单个加载任务，更加灵活。

##### max_tolerable_backend_down_num

- **单位**: -
- **默认**: 0
- **描述**: 允许的最大故障BE节点数量。如果超过此数量，常规加载作业将无法自动恢复。

##### period_of_auto_resume_min

- **单位**: 分
- **默认**: 5
- **描述**: 常规加载作业自动恢复的时间间隔。

##### spark_load_default_timeout_second

- **单位**: s
- **默认**: 86400
- **描述**: Spark加载作业的超时持续时间，以秒为单位。

##### spark_home_default_dir

- **单位**: -
- **默认**: StarRocksFE.STARROCKS_HOME_DIR + "/lib/spark2x"
- **描述**: Spark客户端的根目录。

##### stream_load_default_timeout_second

- **单位**: s
- **默认**: 600
- **描述**: 流加载作业的默认超时持续时间，以秒为单位。

##### max_stream_load_timeout_second

- **单位**: s
- **默认**: 259200
- **描述**: 流加载作业的最大允许超时持续时间，以秒为单位。

##### insert_load_default_timeout_second

- **单位**: s
- **默认**: 3600
- **描述**: 用于加载数据的INSERT INTO语句的超时持续时间，以秒为单位。

##### broker_load_default_timeout_second

- **单位**: s
- **默认**: 14400
- **描述**: Broker加载作业的超时持续时间，以秒为单位。

##### min_bytes_per_broker_scanner

- **单位**: 字节
- **默认**: 67108864
- **描述**: 经纪人加载实例处理的最小数据量，以字节为单位。

##### max_broker_concurrency

- **单位**: -
- **默认**: 100
- **描述**: 经纪人加载任务的最大并发实例数。从v3.1版本开始，此参数已不建议使用。

##### export_max_bytes_per_be_per_task

- **单位**: 字节
- **默认**: 268435456
- **描述**: 单个数据卸载任务从单个BE导出的最大数据量，以字节为单位。

##### export_running_job_num_limit

- **单位**: -
- **默认**: 5
- **描述**: 可并行运行的数据导出任务的最大数量。

##### export_task_default_timeout_second

- **单位**: s
- **默认**: 7200
- **描述**: 数据导出任务的超时持续时间，以秒为单位。

##### empty_load_as_error

- **单位**: -
- **默认**: TRUE
- **描述**: 如果未加载任何数据，是否返回错误消息"所有分片均无加载数据"。取值:
  - TRUE: 如果未加载任何数据，系统显示失败消息并返回错误"所有分片均无加载数据"。
  - FALSE: 如果未加载任何数据，系统显示成功消息并返回OK，而非错误。

##### external_table_commit_timeout_ms

- **单位**: 毫秒
- **默认**: 10000
- **描述**: 提交（发布）写事务到StarRocks外部表的超时持续时间。默认值`10000`表示10秒的超时持续时间。

##### enable_sync_publish

- **单位**: -
- **默认**: TRUE
- **描述**: 是否在加载事务的发布阶段同步执行apply任务。此参数仅适用于主键表。有效值:
  - `TRUE`（默认）: apply任务在加载事务的发布阶段同步执行。这意味着只有在apply任务完成后，加载事务才报告为成功，并且加载的数据真正可被查询。当任务一次加载大量数据或频繁加载数据时，将此参数设置为`true`可以提高查询性能和稳定性，但可能会增加加载延迟。
  - `FALSE`: apply任务在加载事务的发布阶段异步执行。这意味着只有在提交apply任务后，加载事务才报告为成功，但加载的数据不能立即查询。在这种情况下，并发查询需要等待apply任务完成或超时后才能继续。当任务一次加载大量数据或频繁加载数据时，将此参数设置为`false`可能会影响查询性能和稳定性。
- **v3.2.0版本引入**

#### 存储

##### default_replication_num

- **单位**: -
- **默认**: 3
- **描述**: `default_replication_num`设置在StarRocks中创建表时每个数据分区的默认副本数。在创建表时，可以通过指定`replication_num=x`覆盖此设置。

##### enable_strict_storage_medium_check

- **单位**: -
- **默认**: FALSE
- **描述**: FE在用户创建表时是否严格检查BE的存储介质。如果此参数设置为`TRUE`，FE在用户创建表时检查BE的存储介质，如果BE的存储介质与CREATE TABLE语句中指定的`storage_medium`参数不同，则返回错误。例如，CREATE TABLE语句中指定的存储介质为SSD，但实际的BE存储介质是HDD。因此，表的创建将失败。如果此参数为`FALSE`，FE在用户创建表时不会检查BE的存储介质。

##### enable_auto_tablet_distribution

- **单位**: -
- **默认**: TRUE
- **描述**: 是否自动设置桶的数量。
  - 如果此参数设置为`TRUE`，在创建表或添加分区时无需指定桶的数量，StarRocks会自动确定桶的数量。
  - 如果此参数设置为`FALSE`，在创建表或添加分区时需要手动指定桶的数量。如果在为表添加新分区时未指定桶的数量，则新分区将继承在创建表时设置的桶的数量。但也可以为新分区手动指定桶的数量。从版本2.5.7开始，StarRocks支持设置此参数。

##### storage_usage_soft_limit_percent

- **单位**: %
- **默认**: 90
- **描述**: 如果BE存储目录的存储使用率（以百分比表示）超过此值并且剩余存储空间小于`storage_usage_soft_limit_reserve_bytes`，则无法克隆Tablet到此目录。

##### storage_usage_soft_limit_reserve_bytes

- **单位**: 字节
- **默认**: `200 * 1024 * 1024 * 1024`
- **描述**: 如果BE存储目录中的剩余存储空间小于此值并且存储使用率（以百分比表示）超过`storage_usage_soft_limit_percent`，则无法克隆Tablet到此目录。

##### catalog_trash_expire_second

- **单位**: s
- **默认**: 86400
- **描述**: 删除表或数据库后，元数据可以保留的最长时间。如果此持续时间到期，数据将被删除，无法恢复。单位：秒。

##### alter_table_timeout_second

- **单位**: s
- **默认**: 86400
- **描述**: 模式更改操作（ALTER TABLE）的超时持续时间。单位：秒。

#### 快速模式演化(快速模式演进)

- **默认**: TRUE
- **描述**: 是否为StarRocks集群中的所有表启用快速模式演化。有效值为`TRUE`（默认）和`FALSE`。启用快速模式演化可以增加模式更改的速度，并在添加或删除列时减少资源使用。
  > **注意**
  >
  > - StarRocks共享数据集群不支持此参数。
  > - 如果需要为特定表配置快速模式演化，例如禁用特定表的快速模式演化，可以在表创建时设置表属性[`fast_schema_evolution`](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md#set-fast-schema-evolution)。
- **v3.2.0版本引入**

##### recover_with_empty_tablet

- **单位**: -
- **默认**: FALSE
- **描述**：是否用空的副本替换丢失或损坏的平板副本。如果一个平板副本丢失或损坏，对该平板或其他健康平板的数据查询可能会失败。使用空的平板副本替换丢失或损坏的副本可确保查询仍然可以执行。但是，结果可能不正确，因为数据丢失。默认值是`FALSE`，表示不用空的副本替换丢失或损坏的平板副本，查询将失败。

##### tablet_create_timeout_second

- **单位**：秒
- **默认值**：10
- **描述**：创建平板的超时持续时间（秒）。

##### tablet_delete_timeout_second

- **单位**：秒
- **默认值**：2
- **描述**：删除平板的超时持续时间（秒）。

##### check_consistency_default_timeout_second

- **单位**：秒
- **默认值**：600
- **描述**：副本一致性检查的超时持续时间。您可以基于平板大小设置此参数。

##### tablet_sched_slot_num_per_path

- **单位**：-
- **默认值**：8
- **描述**：可以在BE存储目录中并行运行的与平板相关的任务的最大数量。别名为`schedule_slot_num_per_path`。从v2.5版本开始，此参数的默认值从`4`更改为`8`。

##### tablet_sched_max_scheduling_tablets

- **单位**：-
- **默认值**：10000
- **描述**：可以同时调度的平板最大数量。如果超过该值，将跳过平板平衡和修复检查。

##### tablet_sched_disable_balance

- **单位**：-
- **默认值**：FALSE
- **描述**：是否禁用平板平衡。`TRUE`表示已禁用平板平衡。`FALSE`表示已启用平板平衡。别名为`disable_balance`。

##### tablet_sched_disable_colocate_balance

- **单位**：-
- **默认值**：FALSE
- **描述**：是否禁用Colocate表的副本平衡。`TRUE`表示已禁用副本平衡。`FALSE`表示已启用副本平衡。别名为`disable_colocate_balance`。

##### tablet_sched_max_balancing_tablets

- **单位**：-
- **默认值**：500
- **描述**：可以同时平衡的平板最大数量。如果超过该值，将跳过平板重新平衡。别名为`max_balancing_tablets`。

##### tablet_sched_balance_load_disk_safe_threshold

- **单位**：-
- **默认值**：0.5
- **描述**：确定BE磁盘使用是否平衡的阈值。只有在`tablet_sched_balancer_strategy`设置为`disk_and_tablet`时，此参数才会生效。如果所有BE的磁盘使用低于50%，则认为磁盘使用是平衡的。对于`disk_and_tablet`策略，如果最高和最低BE磁盘使用之间的差异大于10%，则认为磁盘使用是不平衡的，并触发平板重新平衡。别名为`balance_load_disk_safe_threshold`。

##### tablet_sched_balance_load_score_threshold

- **单位**：-
- **默认值**：0.1
- **描述**：确定BE负载是否平衡的阈值。只有当`tablet_sched_balancer_strategy`设置为`be_load_score`时，此参数才会生效。负载比平均负载低10%的BE处于低负载状态，负载比平均负载高10%的BE处于高负载状态。别名为`balance_load_score_threshold`。

##### tablet_sched_repair_delay_factor_second

- **单位**：秒
- **默认值**：60
- **描述**：修复副本的间隔时间（秒）。别名为`tablet_repair_delay_factor_second`。

##### tablet_sched_min_clone_task_timeout_sec

- **单位**：秒
- **默认值**：3 * 60
- **描述**：克隆平板的最小超时持续时间（秒）。

##### tablet_sched_max_clone_task_timeout_sec

- **单位**：秒
- **默认值**：`2 * 60 * 60`
- **描述**：克隆平板的最大超时持续时间（秒）。别名为`max_clone_task_timeout_sec`。

##### tablet_sched_max_not_being_scheduled_interval_ms

- **单位**：毫秒
- **默认值**：`15 * 60 * 100`
- **描述**：在平板克隆任务正在被调度时，如果特定时间内某个平板未被调度，StarRocks将优先调度它。

#### 其他FE动态参数

##### plugin_enable

- **单位**：-
- **默认值**：TRUE
- **描述**：是否允许在FEs上安装插件。只能在Leader FE上安装或卸载插件。

##### max_small_file_number

- **单位**：-
- **默认值**：100
- **描述**：可以存储在FE目录上的小文件的最大数量。

##### max_small_file_size_bytes

- **单位**：字节
- **默认值**：1024 * 1024
- **描述**：小文件的最大大小（字节）。

##### agent_task_resend_wait_time_ms

- **单位**：毫秒
- **默认值**：5000
- **描述**：FE必须等待的持续时间，然后才能重发代理任务。只有当任务创建时间和当前时间之间的间隔大于此参数的值时，才可以重发代理任务。此参数用于防止重复发送代理任务。

##### backup_job_default_timeout_ms

- **单位**：毫秒
- **默认值**：86400*1000
- **描述**：备份作业的超时持续时间。如果超过此值，备份作业将失败。

##### report_queue_size

- **单位**：-
- **默认值**：100
- **描述**：可以在报告队列中等待的作业的最大数量。报告涉及BE的磁盘、任务和平板信息。如果报告作业在队列中堆积过多，将发生OOM。

##### enable_experimental_mv

- **单位**：-
- **默认值**：TRUE
- **描述**：是否启用异步物化视图功能。TRUE表示已启用此功能。从v2.5.2版本开始，默认情况下已启用此功能。对于早于v2.5.2版本的版本，默认情况下已禁用此功能。

##### authentication_ldap_simple_bind_base_dn

- **单位**：-
- **默认值**：空字符串
- **描述**：基本DN，即LDAP服务器开始搜索用户验证信息的起始点。

##### authentication_ldap_simple_bind_root_dn

- **单位**：-
- **默认值**：空字符串
- **描述**：用于搜索用户验证信息的管理员DN。

##### authentication_ldap_simple_bind_root_pwd

- **单位**：-
- **默认值**：空字符串
- **描述**：用于搜索用户验证信息的管理员密码。

##### authentication_ldap_simple_server_host

- **单位**：-
- **默认值**：空字符串
- **描述**：LDAP服务器运行的主机。

##### authentication_ldap_simple_server_port

- **单位**：-
- **默认值**：389
- **描述**：LDAP服务器的端口。

##### authentication_ldap_simple_user_search_attr

- **单位**：-
- **默认值**：uid
- **描述**：标识LDAP对象中用户的属性名称。

##### max_upload_task_per_be

- **单位**：-
- **默认值**：0
- **描述**：在每次备份操作中，StarRocks分配给BE节点的最大上传任务数。当此项设置为小于或等于0时，任务数量不受限制。从v3.1.0版本开始支持此项。

##### max_download_task_per_be

- **单位**：-
- **默认值**：0
- **描述**：在每次还原操作中，StarRocks分配给BE节点的最大下载任务数。当此项设置为小于或等于0时，任务数量不受限制。从v3.1.0版本开始支持此项。

##### allow_system_reserved_names

- **默认值**：FALSE
- **描述**：是否允许用户创建以`__op`和`__row`开头的列。要启用此功能，请将此参数设置为`TRUE`。请注意，这些名称格式在StarRocks中保留用于特殊目的，创建此类列可能导致未定义的行为。因此，默认情况下禁用此功能。从v3.2.0版本开始支持此项。

##### enable_backup_materialized_view

- **默认值**：TRUE
- **描述**：在备份或还原特定数据库时，是否启用异步物化视图的备份和还原。如果将此项设置为`false`，StarRocks将跳过备份异步物化视图。从v3.2.0版本开始支持此项。

##### enable_colocate_mv_index

- **默认值**：TRUE
- **描述**：在创建同步物化视图时是否支持将同步物化视图索引与基表合并。如果将此项设置为 `true`，则Tablet Sink将加速同步物化视图的写入性能。该项从v3.2.0版本开始受支持。

### 配置FE静态参数

本部分概述了FE配置文件**fe.conf**中可以配置的静态参数。重新为FE配置这些参数后，必须重新启动FE才能使更改生效。

#### 日志

##### log_roll_size_mb

- **默认值**：1024 MB
- **描述**：每个日志文件的大小。单位：MB。默认值 `1024` 指定每个日志文件的大小为1 GB。

##### sys_log_dir

- **默认值**：StarRocksFE.STARROCKS_HOME_DIR + "/log"
- **描述**：存储系统日志文件的目录。

##### sys_log_level

- **默认值**：INFO
- **描述**：系统日志条目分类的严重级别。有效值：`INFO`、`WARN`、`ERROR`和`FATAL`。

##### sys_log_verbose_modules

- **默认值**：空字符串
- **描述**：StarRocks生成系统日志的模块。如果将此参数设置为`org.apache.starrocks.catalog`，StarRocks仅为catalog模块生成系统日志。

##### sys_log_roll_interval

- **默认值**：DAY
- **描述**：StarRocks旋转系统日志条目的时间间隔。有效值：`DAY`和`HOUR`。
  - 如果将此参数设置为`DAY`，则在系统日志文件的名称后添加`yyyyMMdd`格式的后缀。
  - 如果将此参数设置为`HOUR`，则在系统日志文件的名称后添加`yyyyMMddHH`格式的后缀。

##### sys_log_delete_age

- **默认值**：7天
- **描述**：系统日志文件的保留期。默认值 `7天` 指定每个系统日志文件可以保留7天。StarRocks检查每个系统日志文件，并删除生成于7天前的日志文件。

##### sys_log_roll_num

- **默认值**：10
- **描述**：可在`sys_log_roll_interval`参数指定的每个保留期内保留的最大系统日志文件数。

##### audit_log_dir

- **默认值**：StarRocksFE.STARROCKS_HOME_DIR + "/log"
- **描述**：存储审计日志文件的目录。

##### audit_log_roll_num

- **默认值**：90
- **描述**：可在`audit_log_roll_interval`参数指定的每个保留期内保留的最大审计日志文件数。

##### audit_log_modules

- **默认值**：slow_query, query
- **描述**：StarRocks生成审计日志条目的模块。默认情况下，StarRocks为slow_query模块和query模块生成审计日志。使用逗号（,）和空格分隔模块名称。

##### audit_log_roll_interval

- **默认值**：DAY
- **描述**：StarRocks旋转审计日志条目的时间间隔。有效值：`DAY`和`HOUR`。
- 如果将此参数设置为`DAY`，则在审计日志文件的名称后添加`yyyyMMdd`格式的后缀。
- 如果将此参数设置为`HOUR`，则在审计日志文件的名称后添加`yyyyMMddHH`格式的后缀。

##### audit_log_delete_age

- **默认值**：30天
- **描述**：审计日志文件的保留期。默认值 `30天` 指定每个审计日志文件可以保留30天。StarRocks检查每个审计日志文件，并删除生成于30天前的日志文件。

##### dump_log_dir

- **默认值**：StarRocksFE.STARROCKS_HOME_DIR + "/log"
- **描述**：存储转储日志文件的目录。

##### dump_log_modules

- **默认值**：query
- **描述**：StarRocks生成转储日志条目的模块。默认情况下，StarRocks为query模块生成转储日志。使用逗号（,）和空格分隔模块名称。

##### dump_log_roll_interval

- **默认值**：DAY
- **描述**：StarRocks旋转转储日志条目的时间间隔。有效值：`DAY`和`HOUR`。
  - 如果将此参数设置为`DAY`，则在转储日志文件的名称后添加`yyyyMMdd`格式的后缀。
  - 如果将此参数设置为`HOUR`，则在转储日志文件的名称后添加`yyyyMMddHH`格式的后缀。

##### dump_log_roll_num

- **默认值**：10
- **描述**：可在`dump_log_roll_interval`参数指定的每个保留期内保留的最大转储日志文件数。

##### dump_log_delete_age

- **默认值**：7天
- **描述**：转储日志文件的保留期。默认值 `7天` 指定每个转储日志文件可以保留7天。StarRocks检查每个转储日志文件，并删除生成于7天前的日志文件。

#### 服务器

##### frontend_address

- **默认值**：0.0.0.0
- **描述**：FE节点的IP地址。

##### priority_networks

- **默认值**：空字符串
- **描述**：为具有多个IP地址的服务器声明选择策略。请注意，最多只能有一个IP地址与此参数指定的列表匹配。该参数的值是一个列表，其中包含用CIDR表示法以分号（;）分隔的条目，例如10.10.10.0/24。如果没有IP地址与此列表中的条目匹配，将随机选择一个IP地址。

##### http_port

- **默认值**：8030
- **描述**：FE节点中HTTP服务器侦听的端口。

##### http_backlog_num

- **默认值**：1024
- **描述**：FE节点中HTTP服务器持有的后备队列的长度。

##### cluster_name

- **默认值**：StarRocks Cluster
- **描述**：FE所属的StarRocks集群的名称。集群名称在网页上显示为`Title`。

##### rpc_port

- **默认值**：9020
- **描述**：FE节点中Thrift服务器侦听的端口。

##### thrift_backlog_num

- **默认值**：1024
- **描述**：FE节点中Thrift服务器持有的后备队列的长度。

##### thrift_server_max_worker_threads

- **默认值**：4096
- **描述**：Thrift服务器在FE节点中支持的最大工作线程数。

##### thrift_client_timeout_ms

- **默认值**：5000
- **描述**：空闲客户端连接超时时间长度。单位：毫秒。

##### thrift_server_queue_size

- **默认值**：4096
- **描述**：请求挂起的队列长度。如果在Thrift服务器中正在处理的线程数超过`thrift_server_max_worker_threads`指定的值，新请求将被加入待处理队列。

##### brpc_idle_wait_max_time

- **默认值**：10000
- **描述**：bRPC客户端等待空闲状态的最长时间长度。单位：毫秒。

##### query_port

- **默认值**：9030
- **描述**：FE节点中MySQL服务器侦听的端口。

##### mysql_service_nio_enabled

- **默认值**：TRUE
- **描述**：指定在FE节点中是否启用异步I/O。

##### mysql_service_io_threads_num

- **默认值**：4
- **描述**：可由FE节点中的MySQL服务器运行以处理I/O事件的最大线程数。

##### mysql_nio_backlog_num

- **默认值**：1024
- **描述**：FE节点中的MySQL服务器持有的后备队列的长度。

##### max_mysql_service_task_threads_num

- **默认值**：4096
- **描述**：FE节点中的MySQL服务器运行以处理任务的最大线程数。

##### mysql_server_version

- **默认值**：5.1.0
- **描述**：返回给客户端的MySQL服务器版本。修改此参数将影响以下情况中的版本信息：
  1. `select version();`
  2. 握手包版本
  3. 全局变量`version`的值（`show variables like 'version';`）

##### max_connection_scheduler_threads_num

- **默认值**：4096
- **描述**：连接调度器支持的最大线程数。

##### qe_max_connection

- **默认值**：1024
- **描述**：所有用户与FE节点建立的最大连接数。

##### check_java_version

- **默认值**：TRUE
- **描述**：指定是否检查执行程序和编译程序之间的版本兼容性。如果版本不兼容，StarRocks将报告错误并中止Java程序的启动。

#### 元数据和集群管理

##### meta_dir

- **默认值:** StarRocksFE.STARROCKS_HOME_DIR + "/meta"
- **描述:** 存储元数据的目录。

##### heartbeat_mgr_threads_num

- **默认值:** 8
- **描述:** 心跳管理器可运行心跳任务的线程数。

##### heartbeat_mgr_blocking_queue_size

- **默认值:** 1024
- **描述:** 存储心跳管理器运行的心跳任务的阻塞队列大小。

##### metadata_failure_recovery

- **默认值:** FALSE
- **描述:** 指定是否强制重置FE的元数据。设置此参数时请谨慎操作。

##### edit_log_port

- **默认值:** 9010
- **描述:** 用于在StarRocks集群的主、从和观察者FE之间进行通信的端口。

##### edit_log_type

- **默认值:** BDB
- **描述:** 可生成的编辑日志类型。将该值设置为 `BDB`。

##### bdbje_heartbeat_timeout_second

- **默认值:** 30
- **描述:** StarRocks集群的主、从和观察者FE之间发出的心跳超时时间。单位：秒。

##### bdbje_lock_timeout_second

- **默认值:** 1
- **描述:** BDB JE-based FE中锁超时的时间。单位：秒。

##### max_bdbje_clock_delta_ms

- **默认值:** 5000
- **描述:** 允许主FE和从或观察者FE之间的最大时钟偏差。单位：毫秒。

##### txn_rollback_limit

- **默认值:** 100
- **描述:** 可回滚的最大事务数。

##### bdbje_replica_ack_timeout_second

- **默认值:** 10
- **描述:** 在元数据从主FE写入到从FE时，主FE等待从FE返回指定数量的ACK消息的最长时间。单位：秒。如果正在写入大量的元数据，从FE需要较长时间才能返回ACK消息给主FE，会发生ACK超时。在这种情况下，元数据写入失败，FE进程退出。建议增加该参数的值以防止这种情况发生。

##### master_sync_policy

- **默认值:** SYNC
- **描述:** 主FE刷新日志到磁盘的策略。该参数仅在当前FE为主FE时有效。有效值：
  - `SYNC`：提交事务时生成的日志条目会同时被刷新到磁盘。
  - `NO_SYNC`：提交事务时生成的日志条目不会同时刷新到磁盘。
  - `WRITE_NO_SYNC`：提交事务时生成的日志条目会同时被生成但不会刷新到磁盘。若只部署一个从FE，建议将该参数设置为 `SYNC`。若部署了三个或更多从FE，则建议将该参数和 `replica_sync_policy` 都设置为 `WRITE_NO_SYNC`。

##### replica_sync_policy

- **默认值:** SYNC
- **描述:** 从FE刷新日志到磁盘的策略。该参数仅在当前FE为从FE时有效。有效值：
  - `SYNC`：提交事务时生成的日志条目会同时被刷新到磁盘。
  - `NO_SYNC`：提交事务时生成的日志条目不会同时刷新到磁盘。
  - `WRITE_NO_SYNC`：提交事务时生成的日志条目会同时被生成但不会刷新到磁盘。

##### replica_ack_policy

- **默认值:** SIMPLE_MAJORITY
- **描述:** 日志条目被视为有效的策略。默认值 `SIMPLE_MAJORITY` 指定如果大多数从FE返回ACK消息，则日志条目被视为有效。

##### cluster_id

- **默认值:** -1
- **描述:** FE所属的StarRocks集群的ID。具有相同集群ID的FE或BE属于同一个StarRocks集群。有效值：任何正整数。默认值 `-1` 指定StarRocks将在启动集群的领导FE时为StarRocks集群生成一个随机的集群ID。

#### 查询引擎

##### publish_version_interval_ms

- **默认值:** 10
- **描述:** 发布验证任务发出的时间间隔。单位：毫秒。

##### statistic_cache_columns

- **默认值:** 100000
- **描述:** 可缓存统计表的行数。

##### statistic_cache_thread_pool_size

- **默认值:** 10
- **描述:** 用于刷新统计缓存的线程池大小。

#### 加载和卸载

##### load_checker_interval_second

- **默认值:** 5
- **描述:** 滚动方式处理加载作业的时间间隔。单位：秒。

##### transaction_clean_interval_second

- **默认值:** 30
- **描述:** 清理完成事务的时间间隔。单位：秒。建议指定较短的时间间隔以确保完成事务能够及时清理。

##### label_clean_interval_second

- **默认值:** 14400
- **描述:** 清理标签的时间间隔。单位：秒。建议指定较短的时间间隔以确保历史标签能够及时清理。

##### spark_dpp_version

- **默认值:** 1.0.0
- **描述:** 使用的Spark动态分区修剪（DPP）版本。

##### spark_resource_path

- **默认值:** 空字符串
- **描述:** Spark依赖包的根目录。

##### spark_launcher_log_dir

- **默认值:** sys_log_dir + "/spark_launcher_log"
- **描述:** 存储Spark日志文件的目录。

##### yarn_client_path

- **默认值:** StarRocksFE.STARROCKS_HOME_DIR + "/lib/yarn-client/hadoop/bin/yarn"
- **描述:** Yarn客户端包的根目录。

##### yarn_config_dir

- **默认值:** StarRocksFE.STARROCKS_HOME_DIR + "/lib/yarn-config"
- **描述:** 存储Yarn配置文件的目录。

##### export_checker_interval_second

- **默认值:** 5
- **描述:** 调度加载作业的时间间隔。

##### export_task_pool_size

- **默认值:** 5
- **描述:** 卸载任务线程池的大小。

#### 存储

##### default_storage_medium

- **默认值:** HDD
- **描述:** 表或分区在创建时如果未指定存储介质，则使用的默认存储介质。有效值：`HDD` 和 `SSD`。在创建表或分区时，如果未指定存储介质类型，将使用该参数指定的默认存储介质。

##### tablet_sched_balancer_strategy

- **默认值:** disk_and_tablet
- **描述:** 在表片间实现的负载均衡策略。此参数的别名为 `tablet_balancer_strategy`。有效值：`disk_and_tablet` 和 `be_load_score`。

##### tablet_sched_storage_cooldown_second

- **默认值:** -1
- **描述:** 从表创建时间开始的自动冷却的延迟时间。此参数的别名为 `storage_cooldown_second`。单位：秒。默认值 `-1` 指定自动冷却被禁用。若要启用自动冷却，请将此参数设置为大于 `-1` 的值。

##### tablet_stat_update_interval_second

- **默认值:** 300
- **描述:** FE从每个BE检索片统计信息的时间间隔。单位：秒。

#### StarRocks 共享数据集群

##### run_mode

- **默认值:** shared_nothing
- **描述:** StarRocks集群的运行模式。有效值：shared_data 和 shared_nothing（默认）。

  - shared_data 表示在共享数据模式下运行StarRocks。
  - shared_nothing 表示在非共享数据模式下运行StarRocks。
**警告**
不要同时为StarRocks集群采用 shared_data 和 shared_nothing 模式。不支持混合部署。
部署集群后不要更改 run_mode。否则，集群将无法重启。不支持从非共享数据集群向共享数据集群或反向转换。

##### cloud_native_meta_port

- **默认值:** 6090
- **描述:** 云原生元服务RPC端口。

##### cloud_native_storage_type

- **默认值:** S3
- **描述**：您使用的对象存储类型。 在共享数据模式下，StarRocks支持在Azure Blob（从v3.1.1开始支持）和与S3协议兼容的对象存储（如AWS S3，Google GCP和MinIO）中存储数据。 有效值：S3（默认）和 AZBLOB。 如果将此参数指定为S3，则必须添加以aws_s3为前缀的参数。 如果将此参数指定为AZBLOB，则必须添加以azure_blob为前缀的参数。

##### aws_s3_path

- **默认值**：N/A
- **描述**：用于存储数据的S3路径。 它包括您的S3存储桶的名称和其下的子路径（如果有），例如`testbucket/subpath`。

##### aws_s3_endpoint

- **默认值**：N/A
- **描述**：用于访问您的S3存储桶的端点，例如`https://s3.us-west-2.amazonaws.com`。

##### aws_s3_region

- **默认值**：N/A
- **描述**：您的S3存储桶所在的区域，例如`us-west-2`。

##### aws_s3_use_aws_sdk_default_behavior

- **默认值**：false
- **描述**：是否使用AWS SDK的默认身份验证凭证。 有效值：true和false（默认）。

##### aws_s3_use_instance_profile

- **默认值**：false
- **描述**：是否使用实例配置文件和假定的角色作为访问S3的凭证方法。 有效值：true和false（默认）。
  - 如果您使用基于IAM用户的凭证（访问密钥和秘密密钥）访问S3，必须将此项指定为false，并指定aws_s3_access_key和aws_s3_secret_key。
  - 如果使用实例配置文件访问S3，必须将此项指定为true。
  - 如果使用假定角色访问S3，必须将此项指定为true，并指定aws_s3_iam_role_arn。
  - 如果使用外部AWS账号，还必须指定aws_s3_external_id。

##### aws_s3_access_key

- **默认值**：N/A
- **描述**：访问您的S3存储桶所使用的访问密钥ID。

##### aws_s3_secret_key

- **默认值**：N/A
- **描述**：访问您的S3存储桶所使用的秘密访问密钥。

##### aws_s3_iam_role_arn

- **默认值**：N/A
- **描述**：具有对存储数据文件的S3存储桶具有特权的IAM角色的ARN。

##### aws_s3_external_id

- **默认值**：N/A
- **描述**：用于跨账户访问您的S3存储桶的AWS账户的外部ID。

##### azure_blob_path

- **默认值**：N/A
- **描述**：用于存储数据的Azure Blob Storage路径。 它包括您存储账户中容器的名称和容器下的子路径（如果有），例如testcontainer/subpath。

##### azure_blob_endpoint

- **默认值**：N/A
- **描述**：您的Azure Blob Storage存储账户的终结点，例如`https://test.blob.core.windows.net`。

##### azure_blob_shared_key

- **默认值**：N/A
- **描述**：用于授权对您的Azure Blob Storage发出请求的共享密钥。

##### azure_blob_sas_token

- **默认值**：N/A
- **描述**：用于授权对您的Azure Blob Storage发出请求的共享访问签名（SAS）。

#### 其他FE静态参数

##### plugin_dir

- **默认值**：STARROCKS_HOME_DIR/plugins
- **描述**：存储插件安装包的目录。

##### small_file_dir

- **默认值**：StarRocksFE.STARROCKS_HOME_DIR + "/small_files"
- **描述**：小文件的根目录。

##### max_agent_task_threads_num

- **默认值**：4096
- **描述**：允许在代理任务线程池中的最大线程数。

##### auth_token

- **默认值**：空字符串
- **描述**：用于标识认证在StarRocks集群中FE所属的令牌。 如果未指定此参数，则StarRocks在集群中的领导FE首次启动时会生成一个随机令牌。

##### tmp_dir

- **默认值**：StarRocksFE.STARROCKS_HOME_DIR + "/temp_dir"
- **描述**：存储临时文件（例如在备份和还原过程中生成的文件）的目录。 这些过程完成后，生成的临时文件会被删除。

##### locale

- **默认值**：zh_CN.UTF-8
- **描述**：FE使用的字符集。

##### hive_meta_load_concurrency

- **默认值**：4
- **描述**：支持Hive元数据的最大并发线程数。

##### hive_meta_cache_refresh_interval_s

- **默认值**：7200
- **描述**：更新Hive外部表缓存的元数据的时间间隔。 单位：秒。

##### hive_meta_cache_ttl_s

- **默认值**：86400
- **描述**：Hive外部表缓存的元数据过期时间。 单位：秒。

##### hive_meta_store_timeout_s

- **默认值**：10
- **描述**：连接到Hive元存储的超时时间。 单位：秒。

##### es_state_sync_interval_second

- **默认值**：10
- **描述**：FE获取Elasticsearch索引并同步StarRocks外部表元数据的时间间隔。 单位：秒。

##### enable_auth_check

- **默认值**：TRUE
- **描述**：指定是否启用认证检查功能。 有效值：`TRUE`和`FALSE`。 `TRUE`指定启用此功能，`FALSE`指定禁用此功能。

##### enable_metric_calculator

- **默认值**：TRUE
- **描述**：指定是否启用定期收集指标的功能。 有效值：`TRUE`和`FALSE`。 `TRUE`指定启用此功能，`FALSE`指定禁用此功能。

## BE配置项

一些BE配置项是动态参数，您可以在BE节点仍在线时通过命令进行设置。其余的是静态参数。您只能通过更改相应的配置文件**be.conf**来设置BE节点的静态参数，并重新启动BE节点以使更改生效。

### 查看BE配置项

您可以使用以下命令查看BE配置项：

```shell
curl http://<BE_IP>:<BE_HTTP_PORT>/varz
```

### 配置BE动态参数

您可以使用`curl`命令配置BE节点的动态参数。

```Shell
curl -XPOST http://be_host:http_port/api/update_config?<configuration_item>=<value>
```

BE动态参数如下。

#### report_task_interval_seconds

- **默认值**：10秒
- **描述**：报告任务状态的时间间隔。 任务可以是创建表，删除表，加载数据或更改表模式。

#### report_disk_state_interval_seconds

- **默认值**：60秒
- **描述**：报告存储卷状态的时间间隔，其中包括存储卷中数据的大小。

#### report_tablet_interval_seconds

- **默认值**：60秒
- **描述**：报告所有表格最新版本的时间间隔。

#### report_workgroup_interval_seconds

- **默认值**：5秒
- **描述**：报告所有工作组最新版本的时间间隔。

#### max_download_speed_kbps

- **默认值**：50,000千字节/秒
- **描述**：每个HTTP请求的最大下载速度。 此值会影响BE节点之间数据复制同步的性能。

#### download_low_speed_limit_kbps

- **默认值**：50千字节/秒
- **描述**：每个HTTP请求的下载速度下限。 在低于此值的速度下持续运行的HTTP请求会在配置项`download_low_speed_time`中指定的时间段内中止。

#### download_low_speed_time

- **默认值**：300秒
- **描述**：HTTP请求可以在低于`download_low_speed_limit_kbps`的速度下持续运行的最长时间。 在此配置项中指定的时间段内持续运行低于`download_low_speed_limit_kbps`值速度的HTTP请求会中止。

#### status_report_interval

- **默认值**：5秒
- **描述**：查询报告其配置的时间间隔，可用于FE进行查询统计数据收集。

#### scanner_thread_pool_thread_num

- **默认值**：48（线程数）
- **描述**：存储引擎用于并发存储卷扫描的线程数。 所有线程都在线程池中管理。

#### thrift_client_retry_interval_ms

- **默认值**：100毫秒
- **描述**：thrift客户端重试的时间间隔。

#### scanner_thread_pool_queue_size

- **默认值**：102,400
- **描述**：存储引擎支持的扫描任务数。

#### scanner_row_num

- **默认值：** 16,384
- **描述：** 每个扫描线程返回的最大行数。

#### max_scan_key_num

- **默认值：** 1,024（每个查询的最大扫描键数）
- **描述：** 每个查询划分的最大扫描键数。

#### max_pushdown_conditions_per_column

- **默认值：** 1,024
- **描述：** 每个列允许下推的最大条件数。如果条件数超过此限制，谓词将不会被下推到存储层。

#### exchg_node_buffer_size_bytes

- **默认值：** 10,485,760 字节
- **描述：** 每个查询交换节点接收端的最大缓冲区大小。此配置项是一个软限制。当以过高的速度向接收端发送数据时，将触发背压。

#### memory_limitation_per_thread_for_schema_change

- **默认值：** 2 GB
- **描述：** 每个模式更改任务允许的最大内存大小。

#### update_cache_expire_sec

- **默认值：** 360 秒
- **描述：** 更新缓存的过期时间。

#### file_descriptor_cache_clean_interval

- **默认值：** 3,600 秒
- **描述：** 清理长时间未使用的文件描述符的时间间隔。

#### disk_stat_monitor_interval

- **默认值：** 5 秒
- **描述：** 监控磁盘健康状态的时间间隔。

#### unused_rowset_monitor_interval

- **默认值：** 30 秒
- **描述：** 清理过期行集的时间间隔。

#### max_percentage_of_error_disk

- **默认值：** 0%
- **描述：** 存储卷中可容忍的最大错误百分比，对应的BE节点将退出。

#### default_num_rows_per_column_file_block

- **默认值：** 1,024
- **描述：** 每个行块中可存储的最大行数。

#### pending_data_expire_time_sec

- **默认值：** 1,800 秒
- **描述：** 存储引擎中待处理数据的过期时间。

#### inc_rowset_expired_sec

- **默认值：** 1,800 秒
- **描述：** 进行增量克隆时，传入数据的过期时间。此配置项用于增量克隆。

#### tablet_rowset_stale_sweep_time_sec

- **默认值：** 1,800 秒
- **描述：** 在表中清除陈旧行集的时间间隔。

#### snapshot_expire_time_sec

- **默认值：** 172,800 秒
- **描述：** 快照文件的过期时间。

#### trash_file_expire_time_sec

- **默认值：** 259,200 秒
- **描述：** 清理垃圾文件的时间间隔。

#### base_compaction_check_interval_seconds

- **默认值：** 60 秒
- **描述：** 基础压缩线程轮询的时间间隔。

#### min_base_compaction_num_singleton_deltas

- **默认值：** 5
- **描述：** 触发基础压缩的最小段数。

#### max_base_compaction_num_singleton_deltas

- **默认值：** 100
- **描述：** 每次基础压缩可压缩的最大段数。

#### base_compaction_interval_seconds_since_last_operation

- **默认值：** 86,400 秒
- **描述：** 距上次基础压缩操作的时间间隔。此配置项是触发基础压缩的条件之一。

#### cumulative_compaction_check_interval_seconds

- **默认值：** 1 秒
- **描述：** 累积压缩线程轮询的时间间隔。

#### update_compaction_check_interval_seconds

- **默认值：** 60 秒
- **描述：** 检查主键表更新压缩的时间间隔。

#### min_compaction_failure_interval_sec

- **默认值：** 120 秒
- **描述：** 自上次压缩失败后可以调度Tablet压缩的最短时间间隔。

#### max_compaction_concurrency

- **默认值：** -1
- **描述：** 压缩的最大并发数（包括基础压缩和累积压缩）。值为-1表示不对并发数施加限制。

#### periodic_counter_update_period_ms

- **默认值：** 500 毫秒
- **描述：** 收集计数器统计信息的时间间隔。

#### load_error_log_reserve_hours

- **默认值：** 48 小时
- **描述：** 数据加载日志保留时间。

#### streaming_load_max_mb

- **默认值：** 10,240 MB
- **描述：** 可以流式传输到StarRocks的文件的最大大小。

#### streaming_load_max_batch_size_mb

- **默认值：** 100 MB
- **描述：** 可以流式传输到StarRocks的JSON文件的最大大小。

#### memory_maintenance_sleep_time_s

- **默认值：** 10 秒
- **描述：** 触发ColumnPool GC的时间间隔。StarRocks周期性地执行GC，并将释放的内存返还给操作系统。

#### write_buffer_size

- **默认值：** 104,857,600 字节
- **描述：** 存储在内存中MemTable的缓冲区大小。此配置项是触发刷写的阈值。

#### tablet_stat_cache_update_interval_second

- **默认值：** 300 秒
- **描述：** 更新Tablet Stat Cache的时间间隔。

#### result_buffer_cancelled_interval_time

- **默认值：** 300 秒
- **描述：** BufferControlBlock释放数据前的等待时间。

#### thrift_rpc_timeout_ms

- **默认值：** 5,000 毫秒
- **描述：** thrift RPC的超时时间。

#### max_consumer_num_per_group

- **默认值：** 3（常规加载中消费者组的最大消费者数）
- **描述：** 消费者组中的最大消费者数。

#### max_memory_sink_batch_count

- **默认值：** 20（扫描缓存批次的最大数量）
- **描述：** 扫描缓存批次的最大数量。

#### scan_context_gc_interval_min

- **默认值：** 5 分钟
- **描述：** 清理扫描上下文的时间间隔。

#### path_gc_check_step

- **默认值：** 1,000（每次连续扫描的最大文件数）
- **描述：** 每次连续扫描的最大文件数。

#### path_gc_check_step_interval_ms

- **默认值：** 10 毫秒
- **描述：** 文件扫描之间的时间间隔。

#### path_scan_interval_second

- **默认值：** 86,400 秒
- **描述：** GC清理过期数据的时间间隔。

#### storage_flood_stage_usage_percent

- **默认值：** 95
- **描述：** 如果BE存储目录的使用率（以百分比表示）超过此值，并且剩余存储空间小于`storage_flood_stage_left_capacity_bytes`，则拒绝加载和恢复作业。

#### storage_flood_stage_left_capacity_bytes

- **默认值：** 107,374,182,400 字节（加载和恢复作业拒绝的剩余存储空间阈值）
- **描述：** 如果BE存储目录的剩余存储空间小于此值，并且使用率（以百分比表示）超过`storage_flood_stage_usage_percent`，则拒绝加载和恢复作业。

#### tablet_meta_checkpoint_min_new_rowsets_num

- **默认值：** 10（触发TabletMeta Checkpoint的最小新行集数量）
- **描述：** 自上次TabletMeta Checkpoint以来创建的最小行集数。

#### tablet_meta_checkpoint_min_interval_secs

- **默认值：** 600 秒
- **描述：** 线程轮询TabletMeta Checkpoint的时间间隔。

#### max_runnings_transactions_per_txn_map

- **默认值：** 100（每个分区并发事务的最大数目）
- **描述：** 每个分区中可以同时运行的最大事务数。

#### tablet_max_pending_versions

- **默认值：** 1,000（主键表中可容忍的最大待处理版本数）
- **描述：** 主键表中可容忍的最大待处理版本数。待处理版本指已提交但尚未应用的版本。

#### tablet_max_versions

- **默认值：** 1,000（表中允许的最大版本数）
- **描述：** 表中允许的最大版本数量。如果版本数超过此值，新的写请求将失败。

#### max_hdfs_file_handle

- **默认值：** 1,000（HDFS文件描述符的最大数目）
- **描述：** 可打开的HDFS文件描述符的最大数目。

#### be_exit_after_disk_write_hang_second

- **默认值：** 60 秒
- **描述：** 磁盘挂起后BE等待退出的时间。

#### min_cumulative_compaction_failure_interval_sec

- **默认值：** 30 秒
- **描述：** 累积压缩在失败后重试的最小时间间隔。

#### size_tiered_level_num

- **默认值：** 7（大小分层压缩的层级数）
- **描述：** Size-tiered合并策略的层数。每个层级最多保留一个数据行集。因此，在稳定状态下，数据行集的数量最多与此配置项中指定的层级号相同。

#### size_tiered_level_multiple

- **默认值：** 5 (相邻层级之间的数据大小倍数，在Size-tiered合并中)
- **描述：** Size-tiered合并策略中，相邻两个层级之间的数据大小倍数。

#### size_tiered_min_level_size

- **默认值：** 131,072 字节
- **描述：** Size-tiered合并策略中最小层级的数据大小。小于该值的数据行集会立即触发数据合并。

#### storage_page_cache_limit

- **默认值：** 20%
- **描述：** PageCache的大小。可以指定大小，例如`20G`、`20,480M`、`20,971,520K`或`21,474,836,480B`。也可以指定为内存大小的比率（百分比），例如`20%`。仅当`disable_storage_page_cache`设置为`false`时生效。

#### internal_service_async_thread_num

- **默认值：** 10（线程数）
- **描述：** 每个BE允许用于与Kafka交互的线程池大小。目前，负责处理常规加载请求的FE依赖于BE与Kafka进行交互，StarRocks中的每个BE都有自己用于与Kafka交互的线程池。如果向BE分发大量常规加载任务，则BE与Kafka进行交互的线程池可能因处理所有任务而过于繁忙。在这种情况下，您可以根据需要调整此参数的值。

#### update_compaction_ratio_threshold

- **默认值：** 0.5
- **描述：** 共享数据集群中用于合并主键表的数据最大比例。如果单个Tablet过大，建议缩小此值。此参数从v3.1.5版本开始支持。

### 配置BE静态参数

只能通过更改相应的配置文件**be.conf**来设置BE的静态参数，并重新启动BE使更改生效。

BE的静态参数如下。

#### hdfs_client_enable_hedged_read

- **默认值：** false
- **单位：** 无
- **描述：** 指定是否启用hedged read功能。此参数从v3.0版本开始支持。

#### hdfs_client_hedged_read_threadpool_size

- **默认值：** 128
- **单位：** 无
- **描述：** 指定您的HDFS客户端上Hedged Read线程池的大小。线程池大小限制了在HDFS客户端中用于运行hedged read的线程数量。此参数从v3.0版本开始支持。相当于您HDFS集群的hdfs-site.xml文件中的dfs.client.hedged.read.threadpool.size参数。

#### hdfs_client_hedged_read_threshold_millis

- **默认值：** 2500
- **单位：** 毫秒
- **描述：** 指定启动hedged read前要等待的毫秒数。例如，您将此参数设置为30。在这种情况下，如果从块中读取的数据未在30毫秒内返回，您的HDFS客户端会立即开始针对不同的块副本进行新的读取。此参数从v3.0版本开始支持。相当于您HDFS集群的hdfs-site.xml文件中的dfs.client.hedged.read.threshold.millis参数。

#### be_port

- **默认值：** 9060
- **单位：** 无
- **描述：** BE thrift服务器端口，用于接收来自FE的请求。

#### brpc_port

- **默认值：** 8060
- **单位：** 无
- **描述：** BE bRPC端口，用于查看bRPC的网络统计信息。

#### brpc_num_threads

- **默认值：** -1
- **单位：** 无
- **描述：** bRPC的线程数。值为-1表示与CPU线程数相同。

#### priority_networks

- **默认值：** 空字符串
- **单位：** 无
- **描述：** 用于指定BE节点的优先IP地址的CIDR格式IP地址，如果托管BE节点的机器有多个IP地址。

#### heartbeat_service_port

- **默认值：** 9050
- **单位：** 无
- **描述：** BE心跳服务端口，用于接收来自FE的心跳信息。

#### starlet_port

- **默认值：** 9070
- **单位：** 无
- **描述：** 共享数据集群中CN（v3.0中的BE）的额外代理服务端口。

#### heartbeat_service_thread_count

- **默认值：** 1
- **单位：** 无
- **描述：** BE心跳服务的线程计数。

#### create_tablet_worker_count

- **默认值：** 3
- **单位：** 无
- **描述：** 用于创建Tablet的线程数。

#### drop_tablet_worker_count

- **默认值：** 3
- **单位：** 无
- **描述：** 用于删除Tablet的线程数。

#### push_worker_count_normal_priority

- **默认值：** 3
- **单位：** 无
- **描述：** 用于处理NORMAL优先级加载任务的线程数。

#### push_worker_count_high_priority

- **默认值：** 3
- **单位：** 无
- **描述：** 用于处理HIGH优先级加载任务的线程数。

#### transaction_publish_version_worker_count

- **默认值：** 0
- **单位：** 无
- **描述：** 用于发布版本的最大线程数。当此值设置为小于或等于0时，系统会将CPU核心数的一半作为值，以避免在导入并发度高但仅使用固定数量线程时出现线程资源不足的情况。从v2.5开始，默认值已从8更改为0。

#### clear_transaction_task_worker_count

- **默认值：** 1
- **单位：** 无
- **描述：** 用于清除事务的线程数。

#### alter_tablet_worker_count

- **默认值：** 3
- **单位：** 无
- **描述：** 用于模式更改的线程数。

#### clone_worker_count

- **默认值：** 3
- **单位：** 无
- **描述：** 用于克隆的线程数。

#### storage_medium_migrate_count

- **默认值：** 1
- **单位：** 无
- **描述：** 用于存储介质迁移（从SATA到SSD）的线程数。

#### check_consistency_worker_count

- **默认值：** 1
- **单位：** 无
- **描述：** 用于检查Tablet一致性的线程数。

#### sys_log_dir

- **默认值：** `${STARROCKS_HOME}/log`
- **单位：** 无
- **描述：** 存储系统日志（包括INFO、WARNING、ERROR和FATAL）的目录。

#### user_function_dir

- **默认值：** `${STARROCKS_HOME}/lib/udf`
- **单位：** 无
- **描述：** 用于存储用户定义函数（UDF）的目录。

#### small_file_dir

- **默认值：** `${STARROCKS_HOME}/lib/small_file`
- **单位：** 无
- **描述：** 用于存储文件管理器下载的文件的目录。

#### sys_log_level

- **默认值：** INFO
- **单位：** 无
- **描述：** 系统日志条目分类的严重级别。有效值：INFO、WARN、ERROR和FATAL。

#### sys_log_roll_mode

- **默认值：** SIZE-MB-1024
- **单位：** 无
- **描述：** 系统日志分段模式。有效值包括`TIME-DAY`、`TIME-HOUR`和`SIZE-MB-`大小。默认值表示日志被分割成每个1GB的段。

#### sys_log_roll_num

- **默认值：** 10
- **单位：** 无
- **描述：** 保留的日志分段数。

#### sys_log_verbose_modules

- **默认值：** 空字符串
- **单位：** 无
- **描述：** 要打印的日志模块。例如，如果将此配置项设置为OLAP，StarRocks仅打印OLAP模块的日志。有效值是BE中的命名空间，包括starrocks、starrocks::debug、starrocks::fs、starrocks::io、starrocks::lake、starrocks::pipeline、starrocks::query_cache、starrocks::stream和starrocks::workgroup。

#### sys_log_verbose_level

- **默认值：** 10
- **单位：** 无
- **描述**：要打印的日志级别。该配置项用于控制代码中使用VLOG初始化的日志输出。

#### log_buffer_level

- **默认**：空字符串
- **单位**：N/A
- **描述**：刷新日志的策略。默认值表示日志在内存中被缓冲。有效值为-1和0。-1表示日志不在内存中被缓冲。

#### num_threads_per_core

- **默认**：3
- **单位**：N/A
- **描述**：在每个CPU核心上启动的线程数。

#### compress_rowbatches

- **默认**：TRUE
- **单位**：N/A
- **描述**：一个布尔值，控制是否在BE之间的RPC中压缩行批次。TRUE表示压缩行批次，FALSE表示不压缩行批次。

#### serialize_batch

- **默认**：FALSE
- **单位**：N/A
- **描述**：一个布尔值，控制是否在BE之间的RPC中序列化行批次。TRUE表示序列化行批次，FALSE表示不序列化行批次。

#### storage_root_path

- **默认**：`${STARROCKS_HOME}/storage`
- **单位**：N/A
- **描述**：存储卷的目录和介质。
  - 多个卷用分号(`;`)隔开。
  - 如果存储介质为SSD，在目录末尾添加`medium:ssd`。
  - 如果存储介质为HDD，在目录末尾添加,medium:hdd。

#### max_length_for_bitmap_function

- **默认**：1000000
- **单位**：字节
- **描述**：位图函数输入值的最大长度。

#### max_length_for_to_base64

- **默认**：200000
- **单位**：字节
- **描述**：to_base64()函数输入值的最大长度。

#### max_tablet_num_per_shard

- **默认**：1024
- **单位**：N/A
- **描述**：每个分片中的最大tablet数。该配置项用于限制每个存储目录下的tablet子目录数。

#### max_garbage_sweep_interval

- **默认**：3600
- **单位**：秒
- **描述**：存储卷垃圾回收的最大时间间隔。

#### min_garbage_sweep_interval

- **默认**：180
- **单位**：秒
- **描述**：存储卷垃圾回收的最小时间间隔。

#### file_descriptor_cache_capacity

- **默认**：16384
- **单位**：N/A
- **描述**：可缓存的文件描述符数。

#### min_file_descriptor_number

- **默认**：60000
- **单位**：N/A
- **描述**：BE进程中的最小文件描述符数。

#### index_stream_cache_capacity

- **默认**：10737418240
- **单位**：字节
- **描述**：BloomFilter、Min和Max的统计信息的缓存容量。

#### disable_storage_page_cache

- **默认**：FALSE
- **单位**：N/A
- **描述**：一个布尔值，控制是否禁用PageCache。
  - 启用PageCache时，StarRocks会缓存最近扫描的数据。
  - 当频繁重复类似查询时，PageCache可以显著提升查询性能。
  - TRUE表示禁用PageCache。
  - 该配置项的默认值已自StarRocks v2.4起从TRUE更改为FALSE。

#### base_compaction_num_threads_per_disk

- **默认**：1
- **单位**：N/A
- **描述**：每个存储卷上用于基本压实的线程数。

#### base_cumulative_delta_ratio

- **默认**：0.3
- **单位**：N/A
- **描述**：累积文件大小与基本文件大小的比率。达到此值的比率是触发基本压实的条件之一。

#### compaction_trace_threshold

- **默认**：60
- **单位**：秒
- **描述**：每次压实的时间阈值。如果一次压实的时间超过了时间阈值，StarRocks会打印相应的跟踪信息。

#### be_http_port

- **默认**：8040
- **单位**：N/A
- **描述**：HTTP服务器端口。

#### be_http_num_workers

- **默认**：48
- **单位**：N/A
- **描述**：HTTP服务器使用的线程数。

#### load_data_reserve_hours

- **默认**：4
- **单位**：小时
- **描述**：小规模加载生成的文件的保留时间。

#### number_tablet_writer_threads

- **默认**：16
- **单位**：N/A
- **描述**：流加载使用的线程数。

#### streaming_load_rpc_max_alive_time_sec

- **默认**：1200
- **单位**：秒
- **描述**：流加载的RPC超时时间。

#### fragment_pool_thread_num_min

- **默认**：64
- **单位**：N/A
- **描述**：查询使用的最小线程数。

#### fragment_pool_thread_num_max

- **默认**：4096
- **单位**：N/A
- **描述**：查询使用的最大线程数。

#### fragment_pool_queue_size

- **默认**：2048
- **单位**：N/A
- **描述**：每个BE节点可处理的查询数上限。

#### enable_token_check

- **默认**：TRUE
- **单位**：N/A
- **描述**：一个布尔值，控制是否启用令牌检查。TRUE表示启用令牌检查，FALSE表示禁用。

#### enable_prefetch

- **默认**：TRUE
- **单位**：N/A
- **描述**：一个布尔值，控制是否启用查询预取。TRUE表示启用预取，FALSE表示禁用。

#### load_process_max_memory_limit_bytes

- **默认**：107374182400
- **单位**：字节
- **描述**：BE节点上所有加载进程占用的内存资源的最大大小限制。

#### load_process_max_memory_limit_percent

- **默认**：30
- **单位**：%
- **描述**：BE节点上所有加载进程占用的内存资源的最大百分比限制。

#### sync_tablet_meta

- **默认**：FALSE
- **单位**：N/A
- **描述**：一个布尔值，控制是否启用表tlet元数据的同步。TRUE表示启用同步，FALSE表示禁用。

#### routine_load_thread_pool_size

- **默认**：10
- **单位**：N/A
- **描述**：每个BE上常规加载的线程池大小。自v3.1.0起，此参数已弃用。现在每个BE上常规加载的线程池大小由FE动态参数max_routine_load_task_num_per_be控制。

#### brpc_max_body_size

- **默认**：2147483648
- **单位**：字节
- **描述**：bRPC的最大body大小。

#### tablet_map_shard_size

- **默认**：32
- **单位**：N/A
- **描述**：tablet map分片大小。该值必须为二的幂。

#### enable_bitmap_union_disk_format_with_set

- **默认**：FALSE
- **单位**：N/A
- **描述**：一个布尔值，控制是否启用BITMAP类型的新存储格式，可以提高bitmap_union的性能。TRUE表示启用新存储格式，FALSE表示禁用。

#### mem_limit

- **默认**：90%
- **单位**：N/A
- **描述**：BE进程内存上限。可以将其设置为百分比（“80%”）或物理限制（“100GB”）。

#### flush_thread_num_per_store

- **默认**：2
- **单位**：N/A
- **描述**：每个存储中用于刷新MemTable的线程数。

#### datacache_enable

- **默认**：false
- **单位**：N/A
- **描述**：是否启用数据缓存。TRUE表示启用数据缓存，FALSE表示禁用数据缓存。

#### datacache_disk_path

- **默认**：N/A
- **单位**：N/A
- **描述**：磁盘路径。我们建议为该参数配置与BE机器上的磁盘数量相同的路径。多个路径需要用分号（;）分隔。

#### datacache_meta_path

- **默认**：N/A
- **单位**：N/A
- **描述**：块元数据的存储路径。您可以自定义存储路径。我们建议将元数据存储在$STARROCKS_HOME路径下。

#### datacache_mem_size

- **默认**：10%
- **单位**：N/A
- **描述**: 可以缓存在内存中的数据的最大量。您可以将其设置为百分比（例如，`10%`）或物理限制（例如，`10G`，`21474836480`）。默认值为`10%`。我们建议将此参数的值设置为至少10GB。

#### datacache_disk_size

- **默认值**: 0
- **单位**: 无
- **描述**: 可以缓存在单个磁盘上的数据的最大量。您可以将其设置为百分比（例如，`80%`）或物理限制（例如，`2T`，`500G`）。例如，如果对`datacache_disk_path`参数配置了两个磁盘路径，并将`datacache_disk_size`参数的值设置为`21474836480`（20GB），则这两个磁盘上最多可以缓存40GB的数据。默认值为`0`，表示只使用内存来缓存数据。单位：字节。

#### jdbc_connection_pool_size

- **默认值**: 8
- **单位**: 无
- **描述**: JDBC连接池大小。在每个BE节点上，访问具有相同jdbc_url的外部表的查询共享相同的连接池。

#### jdbc_minimum_idle_connections

- **默认值**: 1
- **单位**: 无
- **描述**: JDBC连接池中空闲连接的最小数量。

#### jdbc_connection_idle_timeout_ms

- **默认值**: 600000
- **单位**: 无
- **描述**: 在此值之后，JDBC连接池中的空闲连接将过期。如果JDBC连接池中的连接空闲时间超过此值，则连接池将关闭超出配置项`jdbc_minimum_idle_connections`中指定数量的空闲连接。

#### query_cache_capacity

- **默认值**: 536870912
- **单位**: 无
- **描述**: BE中查询缓存的大小。单位：字节。默认大小为512MB。大小不能少于4MB。如果BE的内存容量不足以提供您期望的查询缓存大小，则可以增加BE的内存容量。

#### enable_event_based_compaction_framework

- **默认值**: TRUE
- **单位**: 无
- **描述**: 是否启用基于事件的压实框架。TRUE表示启用了基于事件的压实框架，FALSE表示禁用。启用基于事件的压实框架可以极大地减少在存在许多tablet或单个tablet具有大量数据的情况下进行压实的开销。

#### enable_size_tiered_compaction_strategy

- **默认值**: TRUE
- **单位**: 无
- **描述**: 是否启用大小分层压实策略。TRUE表示启用了大小分层压实策略，FALSE表示禁用。

#### lake_service_max_concurrency

- **默认值**: 0
- **单位**: 无
- **描述**: 共享数据集群中RPC请求的最大并发数。当达到此阈值时，将拒绝传入请求。当此项设置为0时，不限制并发。
| null_encoding | 0 | N/A | |
| num_cores | 0 | N/A | |
| object_storage_access_key_id |  | N/A | |
| object_storage_endpoint |  | N/A | |
| object_storage_endpoint_path_style_access | 0 | N/A | |
| object_storage_endpoint_use_https | 0 | N/A | |
| object_storage_max_connection | 102400 | N/A | |
| object_storage_region | | N/A | |
| object_storage_secret_access_key |  | N/A | |
| orc_coalesce_read_enable | 1 | N/A | |
| orc_file_cache_max_size | 8388608 | N/A | |
| orc_natural_read_size | 8388608 | N/A | |
| parallel_clone_task_per_path | 2 | N/A | |
| parquet_coalesce_read_enable | 1 | N/A | |
| parquet_header_max_size | 16384 | N/A | |
| parquet_late_materialization_enable | 1 | N/A | |
| path_gc_check | 1 | N/A | |
| path_gc_check_interval_second | 86400 | N/A | |
| pipeline_exec_thread_pool_thread_num | 0 | N/A | |
| pipeline_connector_scan_thread_num_per_cpu | 8 | N/A | 用于连接器扫描的每个CPU的线程数  |
| pipeline_max_num_drivers_per_exec_thread | 10240 | N/A | |
| pipeline_prepare_thread_pool_queue_size | 102400 | N/A | |
| pipeline_prepare_thread_pool_thread_num | 0 | N/A | |
| pipeline_print_profile | 0 | N/A | |
| pipeline_scan_thread_pool_queue_size | 102400 | N/A | |
| pipeline_scan_thread_pool_thread_num | 0 | N/A | |
| pipeline_sink_brpc_dop | 64 | N/A | |
| pipeline_sink_buffer_size | 64 | N/A | |
| pipeline_sink_io_thread_pool_queue_size | 102400 | N/A | |
| pipeline_sink_io_thread_pool_thread_num | 0 | N/A | |
| plugin_path |  | N/A | |
| port | 20001 | N/A | |
| pprof_profile_dir |  | N/A | |
| pre_aggregate_factor | 80 | N/A | |
| priority_queue_remaining_tasks_increased_frequency | 512 | N/A | |
| profile_report_interval | 30 | N/A | |
| pull_load_task_dir |  | N/A | |
| query_cache_capacity | 536870912 | N/A | |
| query_debug_trace_dir |  | N/A | |
| query_scratch_dirs |  | N/A | |
| release_snapshot_worker_count | 5 | N/A | |
| repair_compaction_interval_seconds | 600 | N/A | |
| rewrite_partial_segment | 1 | N/A | |
| routine_load_kafka_timeout_second | 10 | N/A | |
| routine_load_pulsar_timeout_second | 10 | N/A | |
| rpc_compress_ratio_threshold | 1.1 | N/A | |
| scan_use_query_mem_ratio | 0.25 | N/A | |
| scratch_dirs | /tmp | N/A | |
| send_rpc_runtime_filter_timeout_ms | 1000 | N/A | |
| sleep_five_seconds | 5 | N/A | |
| sleep_one_second | 1 | N/A | |
| sorter_block_size | 8388608 | N/A | |
| storage_format_version | 2 | N/A | |
| sys_minidump_dir |  | N/A | |
| sys_minidump_enable | 0 | N/A | |
| sys_minidump_interval | 600 | N/A | |
| sys_minidump_limit | 20480 | N/A | |
| sys_minidump_max_files | 16 | N/A | |
| tablet_internal_parallel_max_splitted_scan_bytes | 536870912 | N/A | |
| tablet_internal_parallel_max_splitted_scan_rows | 1048576 | N/A | |
| tablet_internal_parallel_min_scan_dop | 4 | N/A | |
| tablet_internal_parallel_min_splitted_scan_rows | 16384 | N/A | |
| tablet_max_versions | 15000 | N/A | |
| tablet_writer_open_rpc_timeout_sec | 60 | N/A | |
| tc_max_total_thread_cache_bytes | 1073741824 | N/A | |
| thrift_connect_timeout_seconds | 3 | N/A | |
| txn_map_shard_size | 128 | N/A | |
| txn_shard_size | 1024 | N/A | |
| udf_thread_pool_size | 1 | N/A | |
| update_compaction_num_threads_per_disk | 1 | N/A | |
| update_compaction_per_tablet_min_interval_seconds | 120 | N/A | |
| update_memory_limit_percent | 60 | N/A | |
| upload_worker_count | 1 | N/A | |
| use_mmap_allocate_chunk | 0 | N/A | |
| vector_chunk_size | 4096 | N/A | |
| vertical_compaction_max_columns_per_group | 5 | N/A | |
| web_log_bytes | 1048576 | N/A | |