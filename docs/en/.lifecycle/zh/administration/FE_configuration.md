---
displayed_sidebar: English
---

# FE 配置项

FE 参数分为动态参数和静态参数。

- 动态参数可以通过运行 SQL 命令进行配置和调整，非常方便。但是，如果重新启动 FE，配置将会失效。因此，我们建议您同时修改 `fe.conf` 文件中的配置项，以防止修改丢失。

- 静态参数只能在 FE 配置文件 **fe.conf** 中进行配置和调整。**修改此文件后，必须重新启动 FE 才能使更改生效。**

参数是否为动态参数由 [ADMIN SHOW CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SHOW_CONFIG.md) 输出中的 `IsMutable` 列指示。`TRUE` 表示动态参数。

请注意，动态和静态 FE 参数都可以在 **fe.conf** 文件中配置。

## 查看 FE 配置项

FE 启动后，您可以在 MySQL 客户端执行 ADMIN SHOW FRONTEND CONFIG 命令查看参数配置。如果要查询特定参数的配置，请执行以下命令：

```SQL
 ADMIN SHOW FRONTEND CONFIG [LIKE "pattern"];
 ```

 有关返回字段的详细说明，请参阅 [ADMIN SHOW CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SHOW_CONFIG.md)。

> **注意**
>
> 您必须具有管理员权限才能运行与集群管理相关的命令。

## 配置 FE 动态参数

您可以使用 [ADMIN SET FRONTEND CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md) 配置或修改 FE 动态参数的设置。

```SQL
ADMIN SET FRONTEND CONFIG ("key" = "value");
```

> **注意**
>
> FE 重启后，配置将恢复为 `fe.conf` 文件中的默认值。因此，我们建议您同时修改 `fe.conf` 中的配置项，以防止修改丢失。

### 日志记录

#### qe_slow_log_ms

- 单位：毫秒
- 默认值：5000
- 说明：用于确定查询是否为慢速查询的阈值。如果查询的响应时间超过此阈值，则会在 `fe.audit.log` 中将其记录为慢速查询。

### 元数据和集群管理

#### catalog_try_lock_timeout_ms

- **单位**：毫秒
- **默认值**：5000
- **描述**：获取全局锁的超时持续时间。

#### edit_log_roll_num

- **单位**： -
- **默认值**：50000
- **描述**：在为这些日志条目创建日志文件之前可以写入的元数据日志条目的最大数量。该参数用于控制日志文件的大小。新的日志文件将写入 BDBJE 数据库。

#### ignore_unknown_log_id

- **单位**： -
- **默认值**：FALSE
- **描述**：是否忽略未知日志 ID。回滚 FE 时，早期版本的 BE 可能无法识别部分日志 ID。如果值为 `TRUE`，则 FE 忽略未知的日志 ID。如果值为 `FALSE`，则 FE 退出。

#### ignore_materialized_view_error

- **单位**： -
- **默认值**：FALSE
- **描述**：FE 是否忽略物化视图错误导致的元数据异常。如果 FE 因物化视图错误导致元数据异常导致启动失败，可以设置该参数为 `true` 以允许 FE 忽略该异常。从 v2.5.10 开始支持此参数。

#### ignore_meta_check

- **单位**： -
- **默认值**：FALSE
- **描述**：非主 FE 是否忽略主 FE 的元数据间隙。如果该值为 TRUE，则非 leader FE 将忽略与 leader FE 的元数据差距，并继续提供数据读取服务。该参数保证了即使长时间停止 leader FE 也能提供连续的数据读取服务。如果该值为 FALSE，则非 leader FE 不会忽略 leader FE 的元数据差距，停止提供数据读取服务。

#### meta_delay_toleration_second

- **单位**：秒
- **默认值**：300
- **描述**：跟随者 FE 和观察者 FE 上的元数据滞后于领导者 FE 上的元数据的最长持续时间。如果超过此持续时间，非 Leader FE 将停止提供服务。

#### drop_backend_after_decommission

- **单位**： -
- **默认值**：TRUE
- **描述**：是否在 BE 停用后删除该 BE。`TRUE` 表示 BE 在停用后立即删除。`FALSE` 表示 BE 停用后不会被删除。

#### enable_collect_query_detail_info

- **单位**： -
- **默认值**：FALSE
- **描述**：是否采集查询的配置文件。如果此参数设置为 `TRUE`，系统将收集查询的配置文件。如果此参数设置为 `FALSE`，则系统不会收集查询的配置文件。

#### enable_background_refresh_connector_metadata

- **单位**： -
- **默认值**： `true` 在 v3.0 中，<br />`false` 在 v2.5 中
- **描述**：是否开启 Hive 元数据缓存周期性刷新。开启后，StarRocks 会轮询 Hive 集群的元存储（Hive Metastore 或 AWS Glue），并刷新频繁访问的 Hive 目录的缓存元数据，以感知数据变化。`true` 指示启用 Hive 元数据缓存刷新，并 `false` 指示禁用它。从 v2.5.5 开始支持此参数。

#### background_refresh_metadata_interval_millis

- **单位**：毫秒
- **默认值**：600000
- **说明**：连续两次 Hive 元数据缓存刷新之间的时间间隔。从 v2.5.5 开始支持此参数。

#### background_refresh_metadata_time_secs_since_last_access_secs

- **单位**：秒
- **默认值**：86400
- **描述**：Hive 元数据缓存刷新任务的过期时间。对于已访问的 Hive 目录，如果超过指定时间未访问，StarRocks 将停止刷新其缓存的元数据。对于尚未访问的 Hive 目录，StarRocks 不会刷新其缓存的元数据。从 v2.5.5 开始支持此参数。

#### enable_statistics_collect_profile

- **单位**：不适用
- **默认值**：false
- **描述**：是否为统计信息查询生成配置文件。您可以设置此项为 `true` 以允许 StarRocks 生成查询配置文件，用于查询系统统计信息。从 v3.1.5 开始支持此参数。

### 查询引擎

#### max_allowed_in_element_num_of_delete

- 单位： -
- 默认值：10000
- 说明：DELETE 语句中 IN 谓词允许的最大元素数。

#### enable_materialized_view

- 单位： -
- 默认值：TRUE
- 描述：是否启用物化视图的创建。

#### enable_decimal_v3

- 单位： -
- 默认值：TRUE
- 描述：是否支持 DECIMAL V3 数据类型。

#### enable_sql_blacklist

- 单位： -
- 默认值：FALSE
- 描述：是否开启 SQL 查询黑名单检查。开启该功能后，黑名单中的查询将无法执行。

#### dynamic_partition_check_interval_seconds

- 单位：秒
- 默认值：600
- 描述：检查新数据的时间间隔。如果检测到新数据，StarRocks 会自动为数据创建分区。

#### dynamic_partition_enable

- 单位： -
- 默认值：TRUE
- 描述：是否开启动态分区功能。开启该功能后，StarRocks 会为新数据动态创建分区，并自动删除过期的分区，保证数据的新鲜度。

#### http_slow_request_threshold_ms

- 单位：毫秒
- 默认值：5000
- 描述：如果 HTTP 请求的响应时间超过该参数指定的值，则会生成日志来跟踪该请求。
- 引入于：2.5.15，3.1.5

#### max_partitions_in_one_batch

- 单位： -
- 默认值：4096
- 描述：批量创建分区时可以创建的最大分区数。

#### max_query_retry_time

- 单位： -
- 默认值：2
- 描述：FE 上的最大查询重试次数。

#### max_create_table_timeout_second

- 单位：秒
- 默认值：600
- 描述：创建表的最长超时时间。

#### create_table_max_serial_replicas

- 单位： -
- 默认值：128
- 描述：要串行创建的最大副本数。如果实际副本计数超过此值，则将同时创建副本。如果表创建需要很长时间才能完成，请尝试减少此配置。

#### max_running_rollup_job_num_per_table

- 单位： -
- 默认值：1
- 描述：一个表可以并行运行的最大汇总作业数。

#### max_planner_scalar_rewrite_num

- 单位： -
- 默认值：100000
- 描述：优化器可以重写标量运算符的最大次数。

#### enable_statistic_collect

- 单位： -
- 默认值：TRUE
- 描述：是否采集 CBO 的统计信息。默认情况下，此功能处于启用状态。

#### enable_collect_full_statistic

- 单位： -
- 默认值：TRUE
- 描述：是否开启全量统计信息自动采集。默认情况下，此功能处于启用状态。

#### statistic_auto_collect_ratio

- 单位： -
- 默认值：0.8
- 说明：用于确定自动收集的统计信息是否正常的阈值。如果统计信息运行状况低于此阈值，则会触发自动收集。

#### statistic_max_full_collect_data_size

- 单位：长
- 默认值：107374182400
- 说明：用于自动收集统计信息的最大分区的大小（以字节为单位）。如果分区超过此值，则执行采样收集，而不是完整收集。

#### statistic_collect_max_row_count_per_query

- 单位：INT
- 默认值：5000000000
- 描述：单个分析任务查询的最大行数。如果超过此值，分析任务将被拆分为多个查询。

#### statistic_collect_interval_sec

- 单位：秒
- 默认值：300
- 描述：自动采集过程中检查数据更新的时间间隔。

#### statistic_auto_analyze_start_time

- 单位：字符串
- 默认值：00:00:00
- 描述：自动采集的开始时间。取值范围： `00:00:00` - `23:59:59`。

#### statistic_auto_analyze_end_time

- 单位：字符串
- 默认值：23:59:59
- 描述：自动采集的结束时间。取值范围： `00:00:00` - `23:59:59`。

#### statistic_sample_collect_rows

- 单位： -
- 默认值：200000
- 说明：采样收集的最小行数。如果参数值超过表中的实际行数，则执行完整收集。

#### histogram_buckets_size

- 单位： -
- 默认值：64
- 描述：直方图的默认存储桶编号。

#### histogram_mcv_size

- 单位： -
- 默认值：100

- 描述：直方图的最常见值（MCV）数量。

#### histogram_sample_ratio

- 单位：-
- 默认值：0.1
- 描述：直方图的采样比率。

#### histogram_max_sample_row_count

- 单位：-
- 默认值：10000000
- 描述：直方图收集的最大行数。

#### statistics_manager_sleep_time_sec

- 单位：秒
- 默认值：60
- 描述：元数据计划的间隔。系统根据此间隔执行以下操作：
  - 创建用于存储统计信息的表。
  - 删除已删除的统计信息。
  - 删除过期的统计信息。

#### statistic_update_interval_sec

- 单位：秒
- 默认值：`24 * 60 * 60`
- 描述：统计信息缓存更新的间隔。单位：秒。

#### statistic_analyze_status_keep_second

- 单位：秒
- 默认值：259200
- 描述：收集任务历史记录的保留时长。默认值为3天。

#### statistic_collect_concurrency

- 单位：-
- 默认值：3
- 描述：可并行运行的手动收集任务的最大数量。该值默认为3，这意味着最多可以并行运行三个手动收集任务。如果超过该值，传入任务将处于PENDING状态，等待调度。

#### statistic_auto_collect_small_table_rows

- 单位：-
- 默认值：10000000
- 描述：判断外部数据源（Hive、Iceberg、Hudi）中的表在自动收集时是否为小表的阈值。如果表的行数小于此值，则该表被视为小表。
- 引入版本：v3.2

#### enable_local_replica_selection

- 单位：-
- 默认值：FALSE
- 描述：是否选择本地副本进行查询。本地副本可降低网络传输成本。如果设置为TRUE，CBO会优先选择与当前FE具有相同IP地址的BE上的Tablet副本。如果该参数设置为`FALSE`，则可以同时选择本地副本和非本地副本。默认值为FALSE。

#### max_distribution_pruner_recursion_depth

- 单位：-
- 默认值：100
- 描述：分区修剪程序允许的最大递归深度。增加递归深度可以修剪更多元素，但也会增加CPU消耗。

#### enable_udf

- 单位：-
- 默认值：FALSE
- 描述：是否启用UDF。

### 加载和卸载

#### max_broker_load_job_concurrency

- **单位**：-
- **默认值**：5
- **描述**：StarRocks集群内允许的最大并发Broker Load作业数。此参数仅对Broker Load有效。此参数的值必须小于`max_running_txn_num_per_db`的值。从v2.5开始，默认值从`10`更改为`5`。此参数的别名为`async_load_task_pool_size`。

#### load_straggler_wait_second

- **单位**：秒
- **默认值**：300
- **描述**：BE副本可以容忍的最大加载延迟。如果超过此值，则执行克隆以从其他副本克隆数据。单位：秒。

#### desired_max_waiting_jobs

- **单位**：-
- **默认值**：1024
- **描述**：FE中待处理作业的最大数量。该数字是指所有作业，例如表创建、加载和架构更改作业。如果FE中的待处理作业数达到此值，FE将拒绝新的加载请求。此参数仅对异步加载生效。从v2.5开始，默认值从100更改为1024。

#### max_load_timeout_second

- **单位**：秒
- **默认值**：259200
- **描述**：加载作业允许的最长超时持续时间。如果超过此限制，加载作业将失败。此限制适用于所有类型的加载作业。

#### min_load_timeout_second

- **单位**：秒
- **默认值**：1
- **描述**：加载作业允许的最短超时持续时间。此限制适用于所有类型的加载作业。

#### max_running_txn_num_per_db

- **单位**：-
- **默认值**：100
- **描述**：StarRocks集群中每个数据库允许运行的最大加载事务数。默认值为`100`。当为数据库运行的实际加载事务数超过此参数的值时，将不会处理新的加载请求。对同步加载作业的新请求将被拒绝，对异步加载作业的新请求将被放入队列中。建议不要增加此参数的值，因为这会增加系统负载。

#### load_parallel_instance_num

- **单位**：-
- **默认值**：1
- **描述**：BE上每个加载作业的最大并发加载实例数。

#### disable_load_job

- **单位**：-
- **默认值**：FALSE
- **描述**：集群遇到错误时是否关闭加载。这样可以防止集群错误造成的任何损失。默认值为`FALSE`，表示未禁用加载。

#### history_job_keep_max_second

- **单位**：秒
- **默认值**：604800
- **描述**：历史作业（如架构更改作业）可以保留的最长持续时间（以秒为单位）。

#### label_keep_max_num

- **单位**：-
- **默认值**：1000
- **描述**：一段时间内可以保留的最大加载作业数。如果超过此数字，则将删除历史作业的信息。

#### label_keep_max_second

- **单位**：秒
- **默认值**：259200
- **描述**：保留已完成且处于“已完成”或“已取消”状态的加载作业标签的最长持续时间（以秒为单位）。默认值为3天。此持续时间到期后，标签将被删除。此参数适用于所有类型的加载作业。值过大会消耗大量内存。

#### max_routine_load_job_num

- **单位**：-
- **默认值**：100
- **描述**：StarRocks集群中Routine Load作业的最大数量。此参数自v3.1.0起已弃用。

#### max_routine_load_task_concurrent_num

- **单位**：-
- **默认值**：5
- **描述**：每个例程加载作业的最大并发任务数。

#### max_routine_load_task_num_per_be

- **单位**：-
- **默认值**：16
- **描述**：每个BE上并发例程加载任务的最大数量。从v3.1.0开始，该参数的默认值从5增加到16，不再需要小于等于BE静态参数的值`routine_load_thread_pool_size`（已弃用）。

#### max_routine_load_batch_size

- **单位**：字节
- **默认值**：4294967296
- **描述**：例程加载任务可以加载的最大数据量（以字节为单位）。

#### routine_load_task_consume_second

- **单位**：秒
- **默认值**：15
- **描述**：集群中每个例程加载任务消耗数据的最长时间。从v3.1.0开始，例程加载作业在job_properties中支持新参数`task_consume_second`[](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md#job_properties)。此参数适用于例程加载作业中的单个加载任务，该作业更加灵活。

#### routine_load_task_timeout_second

- **单位**：秒
- **默认值**：60
- **描述**：集群中每个例程加载任务的超时持续时间。从v3.1.0开始，例程加载作业在job_properties中支持新参数`task_timeout_second`[](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md#job_properties)。此参数适用于例程加载作业中的单个加载任务，该作业更加灵活。

#### routine_load_unstable_threshold_second
- **单位**：秒
- **默认值**：3600
- **描述**：如果例程加载作业中的任何任务滞后，则例程加载作业将设置为“不稳定”状态。具体来说，两者都是：
  - 正在使用的消息的时间戳与当前时间之间的差值超过此阈值。
  - 数据源中存在未使用的消息。

#### max_tolerable_backend_down_num

- **单位**：-
- **默认值**：0
- **描述**：允许的最大故障BE节点数。如果超过此数字，则无法自动恢复例程加载作业。

#### period_of_auto_resume_min

- **单位**：分钟
- **默认值**：5
- **描述**：自动恢复例程加载作业的时间间隔。

#### spark_load_default_timeout_second

- **单位**：秒
- **默认值**：86400
- **描述**：每个Spark Load作业的超时持续时间（以秒为单位）。

#### spark_home_default_dir

- **单位**：-
- **默认值**：StarRocksFE.STARROCKS_HOME_DIR + "/lib/spark2x"
- **描述**：Spark客户端的根目录。

#### stream_load_default_timeout_second

- **单位**：秒
- **默认值**：600
- **描述**：每个流加载作业的默认超时持续时间（以秒为单位）。

#### max_stream_load_timeout_second

- **单位**：秒
- **默认值**：259200
- **描述**：流加载作业允许的最长超时持续时间（以秒为单位）。

#### insert_load_default_timeout_second

- **单位**：秒
- **默认值**：3600
- **描述**：用于加载数据的INSERT INTO语句的超时持续时间（以秒为单位）。

#### broker_load_default_timeout_second

- **单位**：秒
- **默认值**：14400
- **描述**：Broker Load作业的超时持续时间（以秒为单位）。

#### min_bytes_per_broker_scanner

- **单位**：字节
- **默认值**：67108864
- **描述**：Broker Load实例允许处理的最小数据量（以字节为单位）。

#### max_broker_concurrency

- **单位**：-
- **默认值**：100
- **描述**：Broker Load任务的最大并发实例数。从v3.1开始，此参数已弃用。

#### export_max_bytes_per_be_per_task

- **单位**：字节
- **默认值**：268435456
- **描述**：单个数据卸载任务可以从单个BE导出的最大数据量，以字节为单位。

#### export_running_job_num_limit

- **单位**：-
- **默认值**：5
- **描述**：可并行运行的数据导出任务的最大数量。

#### export_task_default_timeout_second

- **单位**：秒
- **默认值**：7200
- **描述**：数据导出任务的超时时长，单位为秒。

#### empty_load_as_error

- **单位**：-
- **默认值**：TRUE
- **描述**：是否在未加载数据时返回错误消息“所有分区均无加载数据”。取值：
  - TRUE：如果未加载任何数据，系统将显示失败消息并返回错误“所有分区均无加载数据”。
  - FALSE：如果未加载任何数据，系统将显示成功消息并返回 OK，而不是错误。

#### external_table_commit_timeout_ms

- **单位**：毫秒
- **默认值**：10000
- **描述**：将写入事务提交（发布）到 StarRocks 外部表的超时时长。默认值 `10000` 表示 10 秒的超时持续时间。

#### enable_sync_publish

- **单位**： -
- **默认值**：TRUE
- **描述**：是否在加载事务的发布阶段同步执行应用任务。该参数仅适用于主键表。有效值：
  - `TRUE`（默认）：应用任务在加载事务的发布阶段同步执行。这意味着只有在应用任务完成后，加载事务才会上报成功，加载的数据才能真正被查询到。当任务单次加载大量数据或频繁加载数据时，设置该参数可以提高查询性能和稳定性，但可能会增加加载延迟。
  - `FALSE`：应用任务在加载事务的发布阶段异步执行。表示提交应用任务后，加载事务报告成功，但加载的数据无法立即查询到。在这种情况下，并发查询需要等待应用任务完成或超时，然后才能继续。当任务单次加载大量数据或频繁加载数据时，设置该参数可能会影响查询性能和稳定性。
- **引入于**：v3.2.0

### 存储

#### default_replication_num

- **单位**： -
- **默认值**：3
- **描述**：`default_replication_num` 设置在 StarRocks 中创建表时每个数据分区的默认副本数。创建表时，可以通过在 CREATE TABLE DDL 中指定 `replication_num=x` 来覆盖此设置。

#### enable_strict_storage_medium_check

- **单位**： -
- **默认值**：FALSE
- **描述**：FE在用户创建表时是否严格检查BE的存储介质。当该参数设置为 `TRUE` 时，FE会在用户创建表时检查BE的存储介质，如果BE的存储介质与 CREATE TABLE语句中指定的 `storage_medium` 参数不同，则返回错误。例如，CREATE TABLE语句中指定的存储介质是SSD，而BE的实际存储介质是HDD。因此，表创建失败。如果该参数为 `FALSE`，则 FE 在用户创建表时不会检查 BE 的存储介质。

#### enable_auto_tablet_distribution

- **单位**： -
- **默认值**：TRUE
- **描述**：是否自动设置桶数。
  - 如果该参数设置为 `TRUE`，则在创建表或添加分区时，无需指定存储桶的数量。StarRocks 会自动决定存储桶的数量。
  - 如果该参数设置为 `FALSE`，则需要在创建表或添加分区时手动指定存储桶数量。如果在向表添加新分区时未指定存储桶计数，则新分区将继承创建表时设置的存储桶计数。但是，您也可以手动指定新分区的存储桶数。从 2.5.7 版本开始，StarRocks 支持设置该参数。

#### storage_usage_soft_limit_percent

- **单位**：%
- **默认值**：90
- **描述**：如果 BE 存储目录的存储使用率（百分比）超过此值，并且剩余存储空间小于 `storage_usage_soft_limit_reserve_bytes`，则无法将 tablet 克隆到该目录。

#### storage_usage_soft_limit_reserve_bytes

- **单位**：字节
- **默认值**： `200 * 1024 * 1024 * 1024`
- **描述**：如果 BE 存储目录下的剩余存储空间小于此值，并且存储使用量（百分比）超过 `storage_usage_soft_limit_percent`，则无法将 tablet 克隆到该目录下。

#### catalog_trash_expire_second

- **单位**：秒
- **默认值**：86400
- **描述**：删除表或数据库后元数据可以保留的最长持续时间。如果此期限到期，数据将被删除且无法恢复。

#### alter_table_timeout_second

- **单位**：秒
- **默认值**：86400
- **描述**：架构更改操作（ALTER TABLE）的超时持续时间。

#### enable_fast_schema_evolution

- **默认值**：FALSE
- **描述**：是否开启 StarRocks 集群内所有表的快速 Schema 演进。有效值为 `TRUE` 和 `FALSE`（默认值）。启用快速架构演进可以提高架构更改的速度，并在添加或删除列时减少资源使用量。
  > **注意**
  >
  > - StarRocks 共享数据集群不支持该参数。
  > - 如果需要为特定表配置快速架构演进，例如禁用特定表的快速架构演进，则可以在创建表时设置表属性 [`fast_schema_evolution`](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md#set-fast-schema-evolution)。

- **引入于**：v3.2.0

#### recover_with_empty_tablet

- **单位**： -
- **默认值**：FALSE
- **描述**：是否将丢失或损坏的平板电脑副本替换为空副本。如果平板电脑副本丢失或损坏，则此平板电脑或其他正常平板电脑上的数据查询可能会失败。将丢失或损坏的平板电脑副本替换为空平板电脑可确保查询仍可执行。但是，结果可能不正确，因为数据丢失了。默认值为 `FALSE`，这意味着丢失或损坏的平板电脑副本不会替换为空副本，并且查询将失败。

#### tablet_create_timeout_second

- **单位**：秒
- **默认值**：10
- **描述**：创建平板电脑的超时持续时间。

#### tablet_delete_timeout_second

- **单位**：秒
- **默认值**：2
- **描述**：删除平板电脑的超时持续时间。

#### check_consistency_default_timeout_second

- **单位**：秒
- **默认值**：600
- **描述**：副本一致性检查的超时持续时间。您可以根据平板电脑的大小设置此参数。

#### tablet_sched_slot_num_per_path

- **单位**： -
- **默认值**：8
- **描述**：BE存储目录中可以并发运行的平板电脑相关任务的最大数量。别名为 `schedule_slot_num_per_path`。从 v2.5 开始，此参数的默认值从 `4` 更改为 `8`。

#### tablet_sched_max_scheduling_tablets

- **单位**： -
- **默认值**：10000
- **描述**：可以同时安排的最大平板电脑数。如果超过该值，将跳过平板电脑平衡和维修检查。

#### tablet_sched_disable_balance

- **单位**： -
- **默认值**：FALSE
- **描述**：是否禁用平板电脑平衡。`TRUE` 表示平板电脑平衡已禁用。`FALSE` 表示已启用平板电脑平衡。别名为 `disable_balance`。

#### tablet_sched_disable_colocate_balance

- **单位**： -
- **默认值**：FALSE
- **描述**：是否禁用共置表的副本平衡。`TRUE` 指示已禁用副本平衡。`FALSE` 指示已启用副本平衡。别名为 `disable_colocate_balance`。

#### tablet_sched_max_balancing_tablets

- **单位**： -
- **默认值**：500
- **描述**：可以同时平衡的最大片剂数。如果超过此值，将跳过平板电脑重新平衡。别名为 `max_balancing_tablets`。

#### tablet_sched_balance_load_disk_safe_threshold

- **单位**： -
- **默认值**：0.5
- **描述**：判断BE磁盘使用率是否均衡的阈值。仅当 `tablet_sched_balancer_strategy` 设置为 `disk_and_tablet` 时，此参数才生效。如果所有BE的磁盘使用率都低于 50%，则认为磁盘使用率是平衡的。对于 `disk_and_tablet` 策略，如果最高和最低BE磁盘使用率之间的差异大于 10%，则将磁盘使用率视为不平衡，并触发平板电脑重新平衡。别名为 `balance_load_disk_safe_threshold`。

#### tablet_sched_balance_load_score_threshold

- **单位**： -
- **默认值**：0.1
- **描述**：判断BE负载是否均衡的阈值。仅当 `tablet_sched_balancer_strategy` 设置为 `be_load_score` 时，此参数才生效。负载比平均负载低10%的BE处于低负载状态，负载比平均负载高10%的BE处于高负载状态。别名为 `balance_load_score_threshold`。

#### tablet_sched_repair_delay_factor_second

- **单位**：秒
- **默认值**：60
- **描述**：修复副本的时间间隔（以秒为单位）。别名为 `tablet_repair_delay_factor_second`。

#### tablet_sched_min_clone_task_timeout_sec

- **单位**：秒
- **默认值**：3 * 60
- **描述**：克隆平板电脑的最短超时持续时间（以秒为单位）。

#### tablet_sched_max_clone_task_timeout_sec

- **单位**：秒
- **默认值**：`2 * 60 * 60`
- **描述**：克隆平板电脑的最长超时持续时间（以秒为单位）。别名为 `max_clone_task_timeout_sec`。

#### tablet_sched_max_not_being_scheduled_interval_ms

- **单位**：毫秒
- **默认值**：`15 * 60 * 100`
- **描述**：在调度平板电脑克隆任务时，如果某个平板电脑在指定时间内未被调度，StarRocks 会优先调度该任务。

### 共享数据特定

#### lake_compaction_score_selector_min_score

- **默认值**：10.0
- **描述**：触发压缩操作的压缩分数阈值。当分区的压缩分数大于或等于此值时，系统将对该分区执行压缩。
- **引入于**：v3.1.0


压缩分数指示分区是否需要压缩，并根据分区中的文件数进行评分。文件过多会影响查询性能，因此系统会定期执行 Compaction 以合并小文件并减少文件计数。您可以根据运行 [SHOW PARTITIONS](../sql-reference/sql-statements/data-manipulation/SHOW_PARTITIONS.md) 返回的结果中的 `MaxCS` 列来检查分区的压缩分数。

#### lake_compaction_max_tasks

- **默认值**：-1
- **描述**：允许的最大并发压缩任务数。
- **引入于**：v3.1.0

系统根据分区中的平板电脑数量计算 Compaction 任务的数量。例如，如果一个分区有 10 个平板电脑，则在该分区上执行一次压缩会创建 10 个压缩任务。如果并发执行的 Compaction 任务数量超过此阈值，系统将不会创建新的 Compaction 任务。将此项设置为 `0` 禁用 Compaction，并将其设置为 `-1` 允许系统根据自适应策略自动计算此值。

#### lake_compaction_history_size

- **默认值**：12
- **描述**：要保存在 Leader FE 节点内存中的最近成功的 Compaction 任务记录数。您可以使用该命令查看最近成功的 Compaction 任务记录 `SHOW PROC '/compactions'` 。需要注意的是，Compaction 历史记录存储在 FE 进程内存中，如果重新启动 FE 进程，它将丢失。
- **引入于**：v3.1.0

#### lake_compaction_fail_history_size

- **默认值**：12
- **描述**：要保留在 Leader FE 节点内存中的最近失败的 Compaction 任务记录数。您可以使用该命令查看最近失败的 Compaction 任务记录 `SHOW PROC '/compactions'` 。需要注意的是，Compaction 历史记录存储在 FE 进程内存中，如果重新启动 FE 进程，它将丢失。
- **引入于**：v3.1.0

#### lake_publish_version_max_threads

- **默认值**：512
- **描述**：版本发布任务的最大线程数。
- **引入于**：v3.2.0

#### lake_autovacuum_parallel_partitions

- **默认值**：8
- **描述**：可以同时进行 AutoVacuum 的最大分区数。AutoVaccum 是压缩后的垃圾回收。
- **引入于**：v3.1.0

#### lake_autovacuum_partition_naptime_seconds

- **单位**：秒
- **默认值**：180
- **描述**：同一分区上 AutoVacuum 操作之间的最小间隔。
- **引入于**：v3.1.0

#### lake_autovacuum_grace_period_minutes

- **单位**：分钟
- **默认值**：5
- **描述**：保留历史数据版本的时间范围。此时间范围内的历史数据版本不会在压缩后通过 AutoVacuum 自动清理。您需要将此值设置为大于最大查询时间，以避免在查询完成之前删除通过运行查询访问的数据。
- **引入于**：v3.1.0

#### lake_autovacuum_stale_partition_threshold

- **单位**：小时
- **默认值**：12
- **描述**：如果分区在此时间范围内没有更新（loading、delete 或 Compactions），则系统不会在此分区上执行 AutoVavac。
- **引入于**：v3.1.0

#### lake_enable_ingest_slowdown

- **默认值**：false
- **描述**：是否开启数据引入速度减慢。启用数据引入速度减慢后，如果分区的压缩分数超过 `lake_ingest_slowdown_threshold`，则该分区上的加载任务将受到限制。
- **引入于**：v3.2.0

#### lake_ingest_slowdown_threshold

- **默认值**：100
- **描述**：触发数据引入速度减慢的压缩分数阈值。此配置仅在 `lake_enable_ingest_slowdown` 设置为 `true` 时生效。
- **引入于**：v3.2.0

> **注意**
>
> 当 `lake_ingest_slowdown_threshold` 小于 `lake_compaction_score_selector_min_score` 时，有效阈值为 `lake_compaction_score_selector_min_score`。

#### lake_ingest_slowdown_ratio

- **默认值**：0.1
- **描述**：触发数据引入速度减慢时加载速率减慢的比率。
- **引入于**：v3.2.0

数据加载任务包括两个阶段：数据写入和数据提交（COMMIT）。数据引入速度减慢是通过延迟数据提交来实现的。延迟率使用以下公式计算：`(compaction_score - lake_ingest_slowdown_threshold) * lake_ingest_slowdown_ratio`。例如，如果数据写入阶段需要 5 分钟，为 `lake_ingest_slowdown_ratio` 0.1，并且 Compaction 分数比 高 10`lake_ingest_slowdown_threshold`，则数据提交时间的延迟为 `5 * 10 * 0.1 = 5` 分钟，这意味着平均加载速度减半。

> **注意**
>
> - 如果一个加载任务同时写入多个分区，则使用所有分区中的最大 Compaction 分数来计算提交时间的延迟。
> - 提交时间的延迟是在第一次尝试提交期间计算的。一旦设置，它将不会改变。一旦延迟时间结束，只要 Compaction 分数不高于 `lake_compaction_score_upper_bound`，系统就会执行数据提交操作。
> - 如果提交时间的延迟超过加载任务的超时时间，则任务将直接失败。

#### lake_compaction_score_upper_bound

- **默认值**：0
- **描述**：分区的压缩分数上限。 `0` 表示没有上限。仅当设置为 时，此项才生效 `lake_enable_ingest_slowdown` `true`。当某个分区的压缩分数达到或超过此上限时，该分区上的所有加载任务将无限期延迟，直到压缩分数降至此值以下或任务超时。
- **引入于**：v3.2.0

### 其他 FE 动态参数

#### plugin_enable

- **单位**： -
- **默认值**：TRUE
- **描述**：插件是否可以安装在 FE 上。插件只能在 Leader FE 上安装或卸载。

#### max_small_file_number

- **单位**： -
- **默认值**：100
- **描述**：FE 目录下可以存储的最大小文件数。

#### max_small_file_size_bytes

- **单位**：字节
- **默认值**：1024 * 1024
- **描述**：小文件的最大大小（以字节为单位）。

#### agent_task_resend_wait_time_ms

- **单位**：ms
- **默认值**：5000
- **描述**：FE 在重新发送 Agent 任务之前必须等待的持续时间。只有当任务创建时间与当前时间的间隔超过该参数的值时，才能重发Agent任务。此参数用于防止重复发送代理任务。单位：毫秒

#### backup_job_default_timeout_ms

- **单位**：ms
- **默认值**：86400*1000
- **描述**：备份作业的超时时长。如果超过此值，备份作业将失败。

#### report_queue_size

- **单位**： -
- **默认值**：100
- **描述**：报表队列中可以等待的最大作业数。该报告是关于 BE 的磁盘、任务和平板电脑信息。如果队列中堆积了太多报表作业，则会发生 OOM。

#### enable_experimental_mv

- **单位**： -
- **默认值**：TRUE
- **描述**：是否开启异步物化视图功能。TRUE 表示已启用此功能。从 v2.5.2 开始，此功能默认启用。对于 v2.5.2 之前的版本，此功能默认处于关闭状态。

#### authentication_ldap_simple_bind_base_dn

- **单位**： -
- **默认值**：空字符串
- **描述**：基本 DN，这是 LDAP 服务器开始搜索用户身份验证信息的点。

#### authentication_ldap_simple_bind_root_dn

- **单位**： -
- **默认值**：空字符串
- **描述**：用于搜索用户身份验证信息的管理员 DN。

#### authentication_ldap_simple_bind_root_pwd

- **单位**： -
- **默认值**：空字符串
- **描述**：用于搜索用户认证信息的管理员密码。

#### authentication_ldap_simple_server_host

- **单位**： -
- **默认值**：空字符串
- **描述**：运行 LDAP 服务器的主机。

#### authentication_ldap_simple_server_port

- **单位**： -
- **默认值**：389
- **描述**：LDAP服务器的端口。

#### authentication_ldap_simple_user_search_attr

- **单位**： -
- **默认值**：uid
- **描述**：用于标识 LDAP 对象中的用户的属性的名称。

#### max_upload_task_per_be

- **单位**： -
- **默认值**：0
- **描述**：在每次 BACKUP 操作中，StarRocks 分配给一个 BE 节点的最大上传任务数。当此项设置为小于或等于 0 时，对任务编号没有限制。从 v3.1.0 开始支持此项。

#### max_download_task_per_be

- **单位**： -
- **默认值**：0
- **描述**：在每次 RESTORE 操作中，StarRocks 分配给一个 BE 节点的最大下载任务数。当此项设置为小于或等于 0 时，对任务编号没有限制。从 v3.1.0 开始支持此项。

#### allow_system_reserved_names

- **默认值**：FALSE
- **描述**：是否允许用户创建名称以 `__op` 和 `__row` 开头的列。要启用此功能，请将此参数设置为 `TRUE`。请注意，这些名称格式在 StarRocks 中是为特殊目的而保留的，创建此类列可能会导致未定义的行为。因此，默认情况下禁用此功能。从 v3.2.0 开始支持此项。

#### enable_backup_materialized_view

- **默认值**：TRUE
- **描述**：在备份或还原特定数据库时，启用异步实例化视图的 BACKUP 和 RESTORE。如果设置为 `false`，StarRocks 将跳过备份异步物化视图。从 v3.2.0 开始支持此项。

#### enable_colocate_mv_index

- **默认值**：TRUE
- **描述**：创建同步物化视图时，是否支持将同步物化视图索引与基表共置。如果设置为 `true`，tablet sink 将加快同步物化视图的写入性能。从 v3.2.0 开始支持此项。

#### enable_mv_automatic_active_check

- **默认值**：TRUE
- **描述**：是否启用系统自动检查和重新激活异步物化视图，因为它们的基表（视图）已经发生了模式更改或已被删除并重新创建。请注意，此功能不会重新激活用户手动设置为非活动状态的物化视图。此项目从v3.1.6版本开始支持。

## 配置FE静态参数

本节概述了您可以在FE配置文件**fe.conf**中配置的静态参数。在为FE重新配置这些参数后，您必须重新启动FE以使更改生效。

### 日志

#### log_roll_size_mb

- **默认值：** 1024 MB
- **描述：** 每个日志文件的大小。单位：MB。默认值`1024`指定每个日志文件的大小为1GB。

#### sys_log_dir

- **默认值：** StarRocksFE.STARROCKS_HOME_DIR + "/log"
- **描述：** 存储系统日志文件的目录。

#### sys_log_level

- **默认值：** INFO
- **描述：** 系统日志条目分类的严重级别。有效值：`INFO`、`WARN`、`ERROR`和`FATAL`。

#### sys_log_verbose_modules

- **默认值：** 空字符串
- **描述：** StarRocks生成系统日志的模块。如果将此参数设置为`org.apache.starrocks.catalog`，StarRocks仅为目录模块生成系统日志。

#### sys_log_roll_interval

- **默认值：** DAY
- **描述：** StarRocks轮换系统日志条目的时间间隔。有效值：`DAY`和`HOUR`。
  - 如果将此参数设置为`DAY`，则系统日志文件的名称将添加`yyyyMMdd`格式的后缀。
  - 如果将此参数设置为`HOUR`，则系统日志文件的名称将添加`yyyyMMddHH`格式的后缀。

#### sys_log_delete_age

- **默认值：** 7d
- **描述：** 系统日志文件的保留期。默认值`7d`指定每个系统日志文件可以保留7天。StarRocks会检查每个系统日志文件，并删除7天前生成的日志文件。

#### sys_log_roll_num

- **默认值：** 10
- **描述：** 可在`sys_log_roll_interval`参数指定的每个保留期内保留的最大系统日志文件数。

#### audit_log_dir

- **默认值：** StarRocksFE.STARROCKS_HOME_DIR + "/log"
- **描述：** 存储审核日志文件的目录。

#### audit_log_roll_num

- **默认值：** 90
- **描述：** 可在`audit_log_roll_interval`参数指定的每个保留期内保留的最大审核日志文件数。

#### audit_log_modules

- **默认值：** slow_query、query
- **描述：** StarRocks生成审核日志条目的模块。默认情况下，StarRocks为slow_query模块和query模块生成审核日志。使用逗号（,）和空格分隔模块名称。

#### audit_log_roll_interval

- **默认值：** DAY
- **描述：** StarRocks轮换审核日志条目的时间间隔。有效值：`DAY`和`HOUR`。
  - 如果将此参数设置为`DAY`，则审核日志文件的名称将添加`yyyyMMdd`格式的后缀。
  - 如果将此参数设置为`HOUR`，则审核日志文件的名称将添加`yyyyMMddHH`格式的后缀。

#### audit_log_delete_age

- **默认值：** 30d
- **描述：** 审核日志文件的保留期。默认值`30d`指定每个审核日志文件可以保留30天。StarRocks会检查每个审核日志文件，并删除30天前生成的日志文件。

#### dump_log_dir

- **默认值：** StarRocksFE.STARROCKS_HOME_DIR + "/log"
- **描述：** 存储转储日志文件的目录。

#### dump_log_modules

- **默认值：** query
- **描述：** StarRocks生成转储日志条目的模块。默认情况下，StarRocks为query模块生成转储日志。使用逗号（,）和空格分隔模块名称。

#### dump_log_roll_interval

- **默认值：** DAY
- **描述：** StarRocks轮换转储日志条目的时间间隔。有效值：`DAY`和`HOUR`。
  - 如果将此参数设置为`DAY`，则转储日志文件的名称将添加`yyyyMMdd`格式的后缀。
  - 如果将此参数设置为`HOUR`，则转储日志文件的名称将添加`yyyyMMddHH`格式的后缀。

#### dump_log_roll_num

- **默认值：** 10
- **描述：** 可在`dump_log_roll_interval`参数指定的每个保留期内保留的最大转储日志文件数。

#### dump_log_delete_age

- **默认值：** 7d
- **描述：** 转储日志文件的保留期。默认值`7d`指定每个转储日志文件可以保留7天。StarRocks会检查每个转储日志文件，并删除7天前生成的日志文件。

### 服务器

#### frontend_address

- **默认值：** 0.0.0.0
- **描述：** FE节点的IP地址。

#### priority_networks

- **默认值：** 空字符串
- **描述：** 声明具有多个IP地址的服务器的选择策略。请注意，最多一个IP地址必须与此参数指定的列表匹配。该参数的值是一个列表，其中包含多个条目，这些条目以CIDR表示法（;）分隔，例如10.10.10.0/24。如果没有IP地址与此列表中的条目匹配，则将随机选择一个IP地址。

#### http_port

- **默认值：** 8030
- **描述：** FE节点中HTTP服务器监听的端口。

#### http_backlog_num

- **默认值：** 1024
- **描述：** FE节点中HTTP服务器持有的积压队列长度。

#### cluster_name

- **默认：** StarRocks集群
- **描述：** FE所属的StarRocks集群名称。集群名称显示在`Title`网页上。

#### rpc_port

- **默认值：** 9020
- **描述：** FE节点中Thrift服务器监听的端口。

#### thrift_backlog_num

- **默认值：** 1024
- **描述：** FE节点中Thrift服务器持有的积压队列长度。

#### thrift_server_max_worker_threads

- **默认值：** 4096
- **描述：** FE节点中Thrift服务器支持的最大工作线程数。

#### thrift_client_timeout_ms

- **默认值：** 5000
- **描述：** 空闲客户端连接超时的时长。单位：毫秒

#### thrift_server_queue_size

- **默认值：** 4096
- **描述：** 挂起请求的队列长度。如果Thrift服务器中正在处理的线程数超过`thrift_server_max_worker_threads`中指定的值，则会将新请求添加到挂起队列中。

#### brpc_idle_wait_max_time

- **默认值：** 10000
- **描述：** bRPC客户端在空闲状态下等待的最长时长。单位：毫秒

#### query_port

- **默认值：** 9030
- **描述：** FE节点中MySQL服务器监听的端口。

#### mysql_service_nio_enabled

- **默认值：** TRUE
- **描述：** 指定是否为FE节点启用异步I/O。

#### mysql_service_io_threads_num

- **默认值：** 4
- **描述：** FE节点中MySQL服务器可以运行的最大线程数，用于处理I/O事件。

#### mysql_nio_backlog_num

- **默认值：** 1024
- **描述：** MySQL服务器在FE节点中持有的积压队列长度。

#### max_mysql_service_task_threads_num

- **默认值：** 4096
- **描述：** FE节点中MySQL服务器处理任务可以运行的最大线程数。

#### mysql_server_version

- **默认值：** 5.1.0
- **描述：** 返回给客户端的MySQL服务器版本。修改此参数将影响以下情况下的版本信息：
  1. `select version();`
  2. 握手数据包版本
  3. 全局变量`version`（`show variables like 'version';`）的值

#### max_connection_scheduler_threads_num

- **默认值：** 4096
- **描述：** 连接调度程序支持的最大线程数。

#### qe_max_connection

- **默认值：** 1024
- **描述：** 所有用户与FE节点可以建立的最大连接数。

#### check_java_version

- **默认值：** TRUE
- **描述：** 指定是否检查已执行和已编译的Java程序之间的版本兼容性。如果版本不兼容，StarRocks将报告错误并中止Java程序的启动。

### 元数据和集群管理

#### meta_dir

- **默认值：** StarRocksFE.STARROCKS_HOME_DIR + "/meta"
- **描述：** 存储元数据的目录。

#### heartbeat_mgr_threads_num

- **默认值：** 8
- **描述：** 心跳管理器可以运行的线程数，用于运行心跳任务。

#### heartbeat_mgr_blocking_queue_size

- **默认值：** 1024
- **描述：** 存储心跳管理器运行的心跳任务的阻塞队列的大小。

#### metadata_failure_recovery

- **默认值：** FALSE
- **描述：** 指定是否强制重置FE的元数据。设置此参数时要小心。

#### edit_log_port

- **默认值：** 9010
- **描述：** StarRocks集群中leader、follower和observer FE之间通信的端口。

#### edit_log_type

- **默认值：** BDB
- **描述：** 可生成的编辑日志类型。将值设置为`BDB`。

#### bdbje_heartbeat_timeout_second

- **默认值：** 30
- **描述：** StarRocks集群中leader、follower和observer FE心跳超时的时长。单位：秒。

#### bdbje_lock_timeout_second

- **默认值：** 1
- **描述：** 基于BDB JE的FE中的锁定超时的时长。单位：秒。

#### max_bdbje_clock_delta_ms

- **默认值：** 5000
- **描述：** StarRocks集群中leader FE与follower或observer FE之间允许的最大时钟偏移量。单位：毫秒。

#### txn_rollback_limit

- **默认值：** 100
- **描述：** 可以回滚的最大事务数。

#### bdbje_replica_ack_timeout_second

- **默认：** 10
- **描述：** 当元数据从主 FE 写入到 Follower FE 时，Leader FE 等待指定数量的 Follower FE 返回 ACK 消息的最长时间。单位：秒。如果正在写入大量元数据，跟随 FE 在返回 ACK 消息给 Leader FE 之前需要较长时间，从而导致 ACK 超时。在这种情况下，元数据写入失败，FE 进程退出。建议增加此参数的值，以防止出现这种情况。

#### master_sync_policy

- **默认：** SYNC
- **描述：** 主 FE 刷新日志到磁盘的策略。该参数仅在当前 FE 为主 FE 时有效。有效取值：
  - `SYNC`：提交事务时，生成日志条目并同时刷新到磁盘。
  - `NO_SYNC`：提交事务时，生成日志条目但不会同时刷新到磁盘。
  - `WRITE_NO_SYNC`：提交事务时，生成日志条目但不会刷新到磁盘。如果只部署了一个跟随 FE，建议将该参数设置为 `SYNC`。如果部署了三个或更多跟随 FE，建议将该参数和 `replica_sync_policy` 都设置为 `WRITE_NO_SYNC`。

#### replica_sync_policy

- **默认：** SYNC
- **描述：** 跟随 FE 刷新日志到磁盘的策略。该参数仅在当前 FE 为跟随 FE 时有效。有效取值：
  - `SYNC`：提交事务时，生成日志条目并同时刷新到磁盘。
  - `NO_SYNC`：提交事务时，生成日志条目但不会同时刷新到磁盘。
  - `WRITE_NO_SYNC`：提交事务时，生成日志条目但不会刷新到磁盘。

#### replica_ack_policy

- **默认：** SIMPLE_MAJORITY
- **描述：** 日志条目被视为有效的策略。默认值 `SIMPLE_MAJORITY` 指定如果大多数跟随 FE 返回 ACK 消息，则日志条目被视为有效。

#### cluster_id

- **默认：** -1
- **描述：** FE 所属的 StarRocks 集群的 ID。具有相同集群 ID 的 FE 或 BE 属于同一个 StarRocks 集群。有效取值：任意正整数。默认值 `-1` 指定 StarRocks 会在 StarRocks 集群的主 FE 首次启动时为该集群生成一个随机的集群 ID。

### 查询引擎

#### publish_version_interval_ms

- **默认：** 10
- **描述：** 发布验证任务下发的时间间隔。单位：毫秒。

#### statistic_cache_columns

- **默认：** 100000
- **描述：** 统计表可以缓存的行数。

#### statistic_cache_thread_pool_size

- **默认：** 10
- **描述：** 用于刷新统计缓存的线程池的大小。

### 装载和卸载

#### load_checker_interval_second

- **默认：** 5
- **描述：** 加载作业滚动处理的时间间隔。单位：秒。

#### transaction_clean_interval_second

- **默认：** 30
- **描述：** 清理已完成事务的时间间隔。单位：秒。建议指定较短的时间间隔，以确保能及时清理已完成的事务。

#### label_clean_interval_second

- **默认：** 14400
- **描述：** 清理标签的时间间隔。单位：秒。建议指定较短的时间间隔，以确保能及时清理历史标签。

#### spark_dpp_version

- **默认：** 1.0.0
- **描述：** 使用的 Spark 动态分区修剪（DPP）版本。

#### spark_resource_path

- **默认：** 空字符串
- **描述：** Spark 依赖包的根目录。

#### spark_launcher_log_dir

- **默认：** sys_log_dir + "/spark_launcher_log"
- **描述：** 存储 Spark 日志文件的目录。

#### yarn_client_path

- **默认：** StarRocksFE.STARROCKS_HOME_DIR + "/lib/yarn-client/hadoop/bin/yarn"
- **描述：** Yarn 客户端包的根目录。

#### yarn_config_dir

- **默认：** StarRocksFE.STARROCKS_HOME_DIR + "/lib/yarn-config"
- **描述：** 存储 Yarn 配置文件的目录。

#### export_checker_interval_second

- **默认：** 5
- **描述：** 定时调度加载作业的时间间隔。

#### export_task_pool_size

- **默认：** 5
- **描述：** 卸载任务线程池的大小。

### 存储

#### default_storage_medium

- **默认：** HDD
- **描述：** 在创建表或分区时，如果未指定存储介质，则用于表或分区的默认存储介质。有效取值：`HDD` 和 `SSD`。在创建表或分区时，如果未指定表或分区的存储介质类型，则使用该参数指定的默认存储介质。

#### tablet_sched_balancer_strategy

- **默认：** disk_and_tablet
- **描述：** 平板电脑之间实现负载均衡的策略。该参数的别名为 `tablet_balancer_strategy`。有效取值：`disk_and_tablet` 和 `be_load_score`。

#### tablet_sched_storage_cooldown_second

- **默认：** -1
- **描述：** 从创建表时开始自动冷却的延迟。该参数的别名为 `storage_cooldown_second`。单位：秒。默认值 `-1` 指定禁用自动冷却。如果要启用自动冷却，请将该参数设置为大于 `-1` 的值。

#### tablet_stat_update_interval_second

- **默认：** 300
- **描述：** FE 从每个 BE 中检索平板电脑统计信息的时间间隔。单位：秒。

### StarRocks 共享数据集群

#### run_mode

- **默认：** shared_nothing
- **描述：** StarRocks 集群的运行模式。有效取值：shared_data、shared_nothing（默认）。

  - shared_data 表示以共享数据模式运行 StarRocks。
  - shared_nothing 表示以无共享模式运行 StarRocks。
注意
StarRocks 集群不能同时采用 shared_data 和 shared_nothing 模式。不支持混合部署。
部署集群后，请勿更改 run_mode。否则，群集将无法重新启动。不支持从无共享集群转换为共享数据集群，反之亦然。

#### cloud_native_meta_port

- **默认：** 6090
- **描述：** 云原生元服务 RPC 端口。

#### cloud_native_storage_type

- **默认：** S3
- **描述：** 您使用的对象存储类型。在共享数据模式下，StarRocks 支持将数据存储在 Azure Blob 中（从 v3.1.1 开始支持），以及兼容 S3 协议的对象存储（如 AWS S3、Google GCP 和 MinIO）。有效取值：S3（默认）和 AZBLOB。如果将此参数指定为 S3，则必须添加以 aws_s3 为前缀的参数。如果将此参数指定为 AZBLOB，则必须添加以 azure_blob 为前缀的参数。

#### aws_s3_path

- **默认：** N/A
- **描述：** 用于存储数据的 S3 路径。由 S3 存储桶的名称及其下的子路径（如果有）组成，例如 `testbucket/subpath`。

#### aws_s3_endpoint

- **默认：** N/A
- **描述：** 用于访问 S3 存储桶的终端节点，例如 `https://s3.us-west-2.amazonaws.com`。

#### aws_s3_region

- **默认：** N/A
- **描述：** S3 存储桶所在的区域，例如 `us-west-2`。

#### aws_s3_use_aws_sdk_default_behavior

- **默认：** false
- **描述：** 是否使用 AWS 开发工具包的默认身份验证凭证。有效取值：true 和 false（默认）。

#### aws_s3_use_instance_profile

- **默认：** false
- **描述：** 是否使用实例配置文件和代入角色作为访问 S3 的凭证方法。有效取值：true 和 false（默认）。
  - 如果使用 IAM 用户凭证（Access Key 和 Secret Key）访问 S3，则必须将此项指定为 false，并指定 aws_s3_access_key 和 aws_s3_secret_key。
  - 如果使用实例配置文件访问 S3，则必须将此项指定为 true。
  - 如果使用 Assume Role 访问 S3，则必须将此项指定为 true，并指定 aws_s3_iam_role_arn。
  - 如果使用外部 AWS 账户，则还必须指定 aws_s3_external_id。

#### aws_s3_access_key

- **默认：** N/A
- **描述：** 用于访问 S3 存储桶的访问密钥 ID。

#### aws_s3_secret_key

- **默认：** N/A
- **描述：** 用于访问 S3 存储桶的秘密访问密钥。

#### aws_s3_iam_role_arn

- **默认：** N/A
- **描述：** 对存储数据文件的 S3 存储桶具有权限的 IAM 角色的 ARN。

#### aws_s3_external_id

- **默认：** N/A
- **描述：** 用于跨账户访问 S3 存储桶的 AWS 账户的外部 ID。

#### azure_blob_path

- **默认：** N/A
- **描述：** 用于存储数据的 Azure Blob 存储路径。由存储帐户中的容器名称和容器下的子路径（如果有）组成，例如 testcontainer/subpath。

#### azure_blob_endpoint

- **默认：** N/A
- **描述：** Azure Blob 存储帐户的终结点，例如 `https://test.blob.core.windows.net`。

#### azure_blob_shared_key

- **默认：** N/A
- **描述：** 用于授权 Azure Blob 存储请求的共享密钥。

#### azure_blob_sas_token

- **默认：** N/A
- **描述：** 用于授权 Azure Blob 存储请求的共享访问签名（SAS）。

### 其他 FE 静态参数

#### plugin_dir

- **默认：** STARROCKS_HOME_DIR/plugins
- **描述：** 存储插件安装包的目录。

#### small_file_dir
- **默认：** StarRocksFE.STARROCKS_HOME_DIR + "/small_files"
- **描述：** 小文件的根目录。

#### max_agent_task_threads_num

- **默认：** 4096
- **描述：** 代理任务线程池中允许的最大线程数。

#### auth_token

- **默认：** 空字符串
- **描述：** 用于标识认证 FE 所属的 StarRocks 集群的令牌。如果未指定该参数，StarRocks 会在集群的 leader FE 首次启动时为集群生成一个随机令牌。

#### tmp_dir

- **默认：** StarRocksFE.STARROCKS_HOME_DIR + "/temp_dir"
- **描述：** 存储临时文件的目录，例如在备份和还原过程中生成的文件。这些过程完成后，生成的临时文件将被删除。

#### locale

- **默认：** zh_CN.UTF-8
- **描述：** FE 使用的字符集。

#### hive_meta_load_concurrency

- **默认：** 4
- **描述：** Hive 元数据支持的最大并发线程数。

#### hive_meta_cache_refresh_interval_s

- **默认：** 7200
- **描述：** 缓存的 Hive 外部表元数据更新的时间间隔。单位：秒。

#### hive_meta_cache_ttl_s

- **默认：** 86400
- **描述：** 缓存的 Hive 外部表元数据过期时间。单位：秒。

#### hive_meta_store_timeout_s

- **默认：** 10
- **描述：** 与 Hive 元存储的连接超时时间。单位：秒。

#### es_state_sync_interval_second

- **默认：** 10
- **描述：** FE 获取 Elasticsearch 索引并同步 StarRocks 外部表元数据的时间间隔。单位：秒。

#### enable_auth_check

- **默认：** TRUE
- **描述：** 指定是否启用认证检查功能。有效值： `TRUE` 和 `FALSE`。 `TRUE` 指定启用此功能，`FALSE` 指定禁用此功能。

#### enable_metric_calculator

- **默认：** TRUE
- **描述：** 指定是否启用定期收集指标的功能。有效值： `TRUE` 和 `FALSE`。 `TRUE` 指定启用此功能，`FALSE` 指定禁用此功能。