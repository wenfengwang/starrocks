---
displayed_sidebar: "Chinese"
---

# Configuration Parameters

This article introduces how to configure StarRocks FE nodes, BE nodes, Brokers, and system parameters, and introduces related parameters.

## FE Configuration Items

FE parameters are divided into dynamic parameters and static parameters. Dynamic parameters can be configured and adjusted online through SQL commands, which is convenient and efficient. **It should be noted that the dynamic settings made through SQL commands will be invalid after restarting the FE. If you want the settings to take effect permanently, it is recommended to modify the fe.conf file at the same time.**

Static parameters must be configured and adjusted in the FE configuration file **fe.conf**. **After the adjustments are completed, the FE needs to be restarted to make the changes take effect.**

Whether a parameter is a dynamic parameter can be viewed through the `IsMutable` column in the results returned by [ADMIN SHOW CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SHOW_CONFIG.md). `TRUE` indicates a dynamic parameter.

Both static and dynamic parameters can be modified through the **fe.conf** file.

### View FE Configuration Items

After the FE is started, you can execute the ADMIN SHOW FRONTEND CONFIG command in the MySQL client to view the parameter configuration. If you want to view the configuration of specific parameters, execute the following command:

```SQL
 ADMIN SHOW FRONTEND CONFIG [LIKE "pattern"];
 ```

For a detailed explanation of the command return fields, refer to [ADMIN SHOW CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SHOW_CONFIG.md).

> **Note**
>
> Only users with the `cluster_admin` role can execute cluster management related commands.

### Configure FE Dynamic Parameters

You can use the [ADMIN SET FRONTEND CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md) command to dynamically modify FE parameters online.

```sql
ADMIN SET FRONTEND CONFIG ("key" = "value");
```

> **Note**
>
> The settings made dynamically will be restored to the configuration in the **fe.conf** file or the default value after the FE is restarted. If you need the settings to take effect permanently, it is recommended to modify the **fe.conf** file after setting to prevent the modifications from becoming invalid after a restart.

This section classifies the dynamic parameters of FE as follows:

- [Log](#log)
- [Metadata and Cluster Management](#metadata-and-cluster-management)
- [Query engine](#query-engine)
- [Import and Export](#import-and-export)
- [Storage](#storage)
- [Other Dynamic Parameters](#other-dynamic-parameters)

#### Log

##### qe_slow_log_ms

- Meaning: The duration for identifying slow queries. If the response time of a query exceeds this threshold, it will be recorded as a slow query in the audit log `fe.audit.log`.
- Unit: milliseconds
- Default value: 5000

#### Metadata and Cluster Management

##### catalog_try_lock_timeout_ms

- Meaning: The timeout duration for acquiring global locks.
- Unit: milliseconds
- Default value: 5000

##### edit_log_roll_num

- Meaning: This parameter controls the size of the log files, specifying how many metadata logs are written before performing a log rolling operation to generate new log files. The new log files will be written to the BDBJE Database.
- Default value: 50000

##### ignore_unknown_log_id

- Meaning: Whether to ignore unknown log IDs. When the FE rolls back to a lower version, there may be log IDs that the lower version BE cannot recognize.<br />If set to TRUE, the FE will ignore these log IDs; otherwise, the FE will exit.
- Default value: FALSE

##### ignore_materialized_view_error

- Meaning: Whether to ignore metadata anomalies caused by materialized view errors. If the FE cannot start due to metadata anomalies caused by materialized view errors, you can set this parameter to `true` to ignore the errors.
- Default value: FALSE
- Introduced in version: 2.5.10

##### ignore_meta_check

- Meaning: Whether to ignore cases where the metadata falls behind. If set to true, the non-master FE will ignore the metadata delay gap between the master FE and itself, even if the metadata delay gap exceeds meta_delay_toleration_second, the non-master FE will still provide read services.<br />This parameter will be very helpful when you try to stop the Master FE for a long time, but still want the non-Master FE to provide read services.
- Default value: FALSE

##### meta_delay_toleration_second

- Meaning: The maximum time that a non-Leader FE can tolerate the metadata lagging behind the Leader FE in the StarRocks cluster where the FE is located.<br />If the delay time between the metadata on the non-Leader FE and the metadata on the Leader FE exceeds the value of this parameter, the non-Leader FE will stop serving.
- Unit: seconds
- Default value: 300

##### drop_backend_after_decommission

- Meaning: Whether to delete the BE after it has been decommissioned. true means the BE will be immediately deleted after being decommissioned. False means the BE will not be deleted after the decommissioning is completed.
- Default value: TRUE

##### enable_collect_query_detail_info

- Meaning: Whether to collect query profile information. When set to true, the system will collect query profiles. When set to false, the system will not collect query profiles.
- Default value: FALSE

##### enable_background_refresh_connector_metadata

- Meaning: Whether to enable periodic refreshing of Hive metadata cache. When enabled, StarRocks will poll the metadata service of the Hive cluster (Hive Metastore or AWS Glue) and refresh the metadata cache of frequently accessed Hive external data directories to detect data updates. `true` means enabled, `false` means disabled.
- Default value: TRUE for v3.0; FALSE for v2.5

##### background_refresh_metadata_interval_millis

- Meaning: The interval between two consecutive refreshes of the Hive metadata cache.
- Unit: milliseconds
- Default value: 600000
- Introduced in version: 2.5.5

##### background_refresh_metadata_time_secs_since_last_access_secs

- Meaning: The expiration time for the Hive metadata cache refresh task. For Hive Catalogs that have been accessed, if they have not been accessed for longer than this time, the refresh of their metadata cache will be stopped. For Hive Catalogs that have not been accessed, StarRocks will not refresh their metadata cache.
- Default value: 86400
- Unit: seconds
- Introduced in version: 2.5.5

##### enable_statistics_collect_profile

- Meaning: Whether to generate Profile for statistical information queries. You can set this to `true` to allow StarRocks to generate Profile for system statistic queries.
- Default value: false
- Introduced in version: 3.1.5

#### Query engine

##### max_allowed_in_element_num_of_delete

- Meaning: The maximum number of elements allowed in the IN predicate of a DELETE statement.
- Default value: 10000

##### enable_materialized_view

- Meaning: Whether to allow the creation of materialized views.
- Default value: TRUE

##### enable_decimal_v3

- Meaning: Whether to enable Decimal V3.
- Default value: TRUE

##### enable_sql_blacklist

- Meaning: Whether to enable SQL Query blacklist validation. If enabled, Query in the blacklist cannot be executed.
- Default value: FALSE

##### dynamic_partition_check_interval_seconds

- Meaning: The time period for dynamic partition check. If new data is generated, partitions will be generated automatically.
- Unit: seconds
- Default value: 600

##### dynamic_partition_enable

- Meaning: Whether to enable dynamic partitioning. After enabling, you can dynamically create partitions for new data as needed, and StarRocks will automatically delete expired partitions to ensure the timeliness of the data.
- 默认值：TRUE

##### http_slow_request_threshold_ms

- 含义：If the time for an HTTP request exceeds the duration specified by this parameter, a log will be generated to track the request.
- 单位：毫秒
- 默认值：5000
- 引入版本：2.5.15，3.1.5

##### max_partitions_in_one_batch

- 含义：The maximum number of partitions when creating partitions in batches.
- 默认值：4096

##### max_query_retry_time

- 含义：The maximum number of query retries on the FE.
- 默认值：2

##### max_create_table_timeout_second

- 含义：The maximum timeout for creating tables.
- 单位：秒
- 默认值：600

##### max_running_rollup_job_num_per_table

- 含义：The maximum concurrency of rollup tasks executed on each table.
- 默认值：1

##### max_planner_scalar_rewrite_num

- 含义：The maximum number of times the optimizer can rewrite ScalarOperator.
- 默认值：100000

##### enable_statistic_collect

- 含义：Whether to collect statistics, this switch is on by default.
- 默认值：TRUE

##### enable_collect_full_statistic

- 含义：Whether to enable automatic full statistics collection, this switch is on by default.
- 默认值：TRUE

##### statistic_auto_collect_ratio

- 含义：The health threshold for automatic statistics. If the health of statistics is lower than this threshold, automatic collection is triggered.
- 默认值：0.8

##### statistic_max_full_collect_data_size

- 含义：The maximum partition size for automatic statistics collection.<br />If it exceeds this value, full collection is abandoned and the table is sampled instead.
- 单位：GB
- 默认值：100

##### statistic_collect_interval_sec

- 含义：In the automatic periodic collection task, the interval for detecting data updates, default is 5 minutes.
- 单位：秒
- 默认值：300

##### statistic_auto_analyze_start_time

- 含义：Used to configure the start time of automatic full collection. Value range: `00:00:00` to `23:59:59`.
- 类型：STRING
- 默认值：00:00:00
- 引入版本：2.5.0

##### statistic_auto_analyze_end_time

- 含义：Used to configure the end time of automatic full collection. Value range: `00:00:00` to `23:59:59`.
- 类型：STRING
- 默认值：23:59:59
- 引入版本：2.5.0

##### statistic_sample_collect_rows

- 含义：Minimum number of sampled rows. If the collection type is set to sampling (SAMPLE), this parameter needs to be set.<br />If the parameter value exceeds the actual number of table rows, a full collection is performed by default.
- 默认值：200000

##### histogram_buckets_size

- 含义：Default number of buckets in the histogram.
- 默认值：64

##### histogram_mcv_size

- 含义：Default number of most common values in the histogram.
- 默认值：100

##### histogram_sample_ratio

- 含义：Default sampling ratio in the histogram.
- 默认值：0.1

##### histogram_max_sample_row_count

- 含义：Maximum sampling row count in the histogram.
- 默认值：10000000

##### statistics_manager_sleep_time_sec

- 含义：Metadata scheduling interval for statistics. Based on this interval, the system performs the following operations:<ul><li>Create statistics tables;</li><li>Delete statistics of deleted tables;</li><li>Delete expired statistics history records.</li></ul>
- 单位：秒
- 默认值：60

##### statistic_update_interval_sec

- 含义：Memory Cache expiration time for statistics information.
- 单位：秒
- 默认值：24 \* 60 \* 60

##### statistic_analyze_status_keep_second

- 含义：The retention time of statistics collection task records, default is 3 days.
- 单位：秒
- 默认值：259200

##### statistic_collect_concurrency

- 含义：The maximum concurrency of manual collection tasks, default is 3, which means that up to 3 manual collection tasks can run concurrently. Excessive tasks are in PENDING state, waiting for scheduling.
- 默认值：3

##### statistic_auto_collect_small_table_rows

- 含义：In automatic collection, the threshold for judging the number of rows of tables under external data sources (Hive, Iceberg, Hudi) as small tables.
- 默认值：10000000
- 引入版本：v3.2

##### enable_local_replica_selection

- 含义：Whether to select local replicas for queries. Local replicas can reduce the network latency for data transmission.<br />If set to true, the optimizer will prioritize tablet replicas on BE nodes with the same IP as the current FE. Setting it to false means that local or non-local replicas can be selected for queries.
- 默认值：FALSE

##### max_distribution_pruner_recursion_depth

- 含义：The maximum recursion depth allowed for partition pruning. Increasing the recursion depth can prune more elements but also increase CPU resource consumption.
- 默认值：100

##### enable_udf

- 含义：Whether to enable UDF.
- 默认值：FALSE

#### Import and Export

##### max_broker_load_job_concurrency

- 含义：The maximum number of Broker Load jobs that can be executed in parallel in a StarRocks cluster. This parameter is only applicable to Broker Load. The default value of this parameter has changed from `10` to `5` since version 2.5. The parameter alias is `async_load_task_pool_size`.
- 默认值：5

##### load_straggler_wait_second

- 含义：Controls the maximum tolerated import lag of BE replicas. Cloning will be performed if it exceeds this duration.
- 单位：秒
- 默认值：300

##### desired_max_waiting_jobs

- 含义：The maximum number of waiting tasks, applicable to all tasks, including table creation, import, and schema change.<br />If the number of jobs in PENDING state in FE reaches this value, the FE will reject new import requests. This parameter configuration is only valid for asynchronous imports. The default value of this parameter has changed from 100 to 1024 since version 2.5.
- 默认值：1024

##### max_load_timeout_second

- 含义：The maximum timeout for import jobs, applicable to all imports.
- 单位：秒
- 默认值：259200

##### min_load_timeout_second

- 含义：The minimum timeout for import jobs, applicable to all imports.
- 单位：秒
- 默认值：1

##### max_running_txn_num_per_db

- 含义：The maximum number of import-related transactions running in a StarRocks cluster database, with a default value of `1000`. Since version 3.1, the default value has been changed from 100 to 1000.<br />When the maximum number of import-related transactions running in the database exceeds the limit, subsequent imports will not be executed. If it is a synchronous import job request, the job will be rejected; if it is an asynchronous import job request, the job will wait in the queue. It is not recommended to increase this value as it will increase the system load.
- 默认值：1000

##### load_parallel_instance_num

- 含义：The maximum number of concurrent instances allowed for each job on a single BE. Deprecated since version 3.1.
- 默认值：1

##### disable_load_job

- 意义: 是否禁用任何导入任务，用作集群问题时的防止损措施。
- 默认值: FALSE

##### history_job_keep_max_second

- 意义: 历史任务最大的保留时长，例如 schema change 任务。
- 单位: 秒
- 默认值: 604800

##### label_keep_max_num

- 意义: 一定时间内保留的导入任务的最大数量。超过该数量后，历史导入作业的信息会被删除。
- 默认值: 1000

##### label_keep_max_second

- 意义: 已完成且处于 FINISHED 或 CANCELLED 状态的导入作业记录在 StarRocks 系统标签中的保留时长，默认为3天。<br />该参数配置适用于所有模式的导入作业。- 单位: 秒。设定过大将会消耗大量内存。
- 默认值: 259200

##### max_routine_load_task_concurrent_num

- 意义: 每个常规加载作业最大并发执行的任务数。
- 默认值: 5

##### max_routine_load_task_num_per_be

- 意义: 每个 BE 并发执行的常规加载导入任务数量上限。从3.1.0版本开始，默认值从5变为16，并且不再需要小于等于BE的配置项 `routine_load_thread_pool_size`（已废弃）。
- 默认值: 16

##### max_routine_load_batch_size

- 意义: 每个常规加载任务导入的最大数据量。
- 单位: 字节
- 默认值: 4294967296

##### routine_load_task_consume_second

- 意义: 集群内每个常规加载导入任务消费数据的最大时间。<br />自v3.1.0起，常规加载导入作业 [job_properties](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md#job_properties) 新增参数 `task_consume_second`，作用于单个常规加载导入作业内的导入任务，更加灵活。
- 单位: 秒
- 默认值: 15

##### routine_load_task_timeout_second

- 意义: 集群内每个常规加载导入任务超时时间，单位: 秒。<br />自v3.1.0起，常规加载导入作业 [job_properties](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md#job_properties) 新增参数 `task_timeout_second`，作用于单个常规加载导入作业内的任务，更加灵活。
- 单位: 秒
- 默认值: 60

##### max_tolerable_backend_down_num

- 意义: 允许的最大故障BE节点数。如果故障的BE节点数超过该阈值，则不能自动恢复常规加载作业。
- 默认值: 0

##### period_of_auto_resume_min

- 意义: 自动恢复常规加载的时间间隔。
- 单位: 分钟
- 默认值: 5

##### spark_load_default_timeout_second

- 意义: Spark导入的超时时间。
- 单位: 秒
- 默认值: 86400

##### spark_home_default_dir

- 意义: Spark客户端根目录。
- 默认值: `StarRocksFE.STARROCKS_HOME_DIR + "/lib/spark2x"`

##### stream_load_default_timeout_second

- 意义: Stream加载的默认超时时间。
- 单位: 秒
- 默认值: 600

##### max_stream_load_timeout_second

- 意义: Stream加载的最大超时时间。
- 单位: 秒
- 默认值: 259200

##### insert_load_default_timeout_second

- 意义: Insert Into语句的超时时间。
- 单位: 秒
- 默认值: 3600

##### broker_load_default_timeout_second

- 意义: Broker加载的超时时间。
- 单位: 秒
- 默认值: 14400

##### min_bytes_per_broker_scanner

- 意义: 单个Broker加载任务最大并发实例数。
- 单位: 字节
- 默认值: 67108864

##### max_broker_concurrency

- 意义: 单个Broker加载任务最大并发实例数。从3.1版本起，StarRocks不再支持该参数。
- 默认值: 100

##### export_max_bytes_per_be_per_task

- 意义: 单个导出任务在单个BE上导出的最大数据量。
- 单位: 字节
- 默认值: 268435456

##### export_running_job_num_limit

- 意义: 导出作业最大的运行数目。
- 默认值: 5

##### export_task_default_timeout_second

- 意义: 导出作业的超时时长。
- 单位: 秒。
- 默认值: 7200

##### empty_load_as_error

- 意义: 导入数据为空时，是否返回报错提示 `all partitions have no load data`。取值：<br /> - **TRUE**：当导入数据为空时，则显示导入失败，并返回报错提示 `all partitions have no load data`。<br /> - **FALSE**：当导入数据为空时，则显示导入成功，并返回 `OK`，不返回报错提示。
- 默认值: TRUE

##### enable_sync_publish

- 意义: 是否在导入事务publish阶段同步执行apply任务，仅适用于主键模型表。取值：
  - `TRUE`（默认）：导入事务publish阶段同步执行apply任务，即apply任务完成后才会返回导入事务publish成功，此时所导入数据真正可查。因此当导入任务一次导入的数据量比较大，或者导入频率较高时，开启该参数可以提升查询性能和稳定性，但是会增加导入耗时。
  - `FALSE`：在导入事务publish阶段异步执行apply任务，即在导入事务publish阶段apply任务提交之后立即返回导入事务publish成功，然而此时导入数据并不真正可查。这时并发的查询需要等到apply任务完成或者超时，才能继续执行。因此当导入任务一次导入的数据量比较大，或者导入频率较高时，关闭该参数会影响查询性能和稳定性。
- 默认值: TRUE
- 引入版本: v3.2.0

##### external_table_commit_timeout_ms

- 意义: 发布写事务到StarRocks外表的超时时长，单位为毫秒。默认值 `10000` 表示超时时长为10秒。
- 单位: 毫秒
- 默认值: 10000

#### 存储

##### default_replication_num

- 意义: 用于配置分区默认的副本数。如果建表时指定了 `replication_num` 属性，则该属性优先生效；如果建表时未指定 `replication_num`，则配置的 `default_replication_num` 生效。建议该参数的取值不要超过集群内BE节点数。
- 默认值: 3

##### enable_strict_storage_medium_check

- 意义: 建表时，是否严格校验存储介质类型。<br />为true时表示在建表时，会严格校验BE上的存储介质。比如建表时指定 `storage_medium = HDD`，而BE上只配置了SSD，那么建表失败。<br />为FALSE时则忽略介质匹配，建表成功。
- 默认值: FALSE

##### enable_auto_tablet_distribution

- Meaning: Whether to enable the automatic bucketing feature.<ul><li>Set to `true` to enable, you do not need to specify the number of buckets when creating a table or adding a partition, StarRocks automatically determines the number of buckets. For the strategy of automatically setting the number of buckets, please refer to [Determine the number of buckets)](../table_design/Data_distribution.md#Determine the number of buckets).</li><li>Set to `false` to disable, you need to manually specify the number of buckets when creating a table.<br />For adding a partition, if you do not specify the number of buckets, the number of buckets for the new partition inherits the number of buckets when the table was created. Of course, you can also manually specify the number of buckets for the new partition.</li></ul>
- Default value: TRUE
- Introduced Version: 2.5.6

##### storage_usage_soft_limit_percent

- Meaning: If the storage directory usage rate of BE exceeds this value and the remaining space is less than `storage_usage_soft_limit_reserve_bytes`, then cloning tablets to this path cannot continue.
- Default value: 90

##### storage_usage_soft_limit_reserve_bytes

- Meaning: Default 200 GB, in bytes. If the remaining space in the BE storage directory is less than this value and the space usage rate exceeds `storage_usage_soft_limit_percent`, then cloning tablets to this path cannot continue.
- Unit: bytes
- Default value: 200 \* 1024 \* 1024 \* 1024

##### catalog_trash_expire_second

- Meaning: After deleting a table/database, the length of time metadata is retained in the recycle bin, beyond this time, the data cannot be recovered.
- Unit: seconds
- Default value: 86400

##### alter_table_timeout_second

- Meaning: Schema change timeout duration.
- Unit: seconds
- Default value: 86400

##### fast_schema_evolution

- Meaning: Whether to enable fast schema evolution for all tables in the cluster, with values: `TRUE` (default) or `FALSE`. When enabled, adding or deleting columns can improve schema change speed and reduce resource usage.
  > **NOTE**
  >
  > - StarRocks storage-computing-separated clusters do not support this parameter.
  > - If you need to set this configuration for a table, such as disabling fast schema evolution for the table, you can set the table property [`fast_schema_evolution`](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md#Set-fast-schema-evolution) when creating the table.
- Default value: TRUE
- Introduced Version: 3.2.0

##### recover_with_empty_tablet

- Meaning: When a tablet replica is lost/damaged, whether to use an empty tablet as a replacement.<br />This ensures that queries can still be executed when a tablet replica is lost/damaged (although the results may be incorrect due to missing data). The default is false, no replacement is made, and the query fails.
- Default value: FALSE

##### tablet_create_timeout_second

- Meaning: Timeout duration for creating tablets. Starting from version 3.1, the default value has been changed from 1 to 10.
- Unit: seconds
- Default value: 10

##### tablet_delete_timeout_second

- Meaning: Timeout duration for deleting tablets.
- Unit: seconds
- Default value: 2

##### check_consistency_default_timeout_second

- Meaning: Timeout duration for replica consistency check.
- Unit: seconds
- Default value: 600

##### tablet_sched_slot_num_per_path

- Meaning: The number of tablet-related tasks that can be executed simultaneously in a BE storage directory. Also known as `schedule_slot_num_per_path`. Starting from version 2.5, the default value of this parameter has been changed from `4` in version 2.4 to `8`.
- Default value: 8

##### tablet_sched_max_scheduling_tablets

- Meaning: The number of tablets that can be scheduled simultaneously. If the number of tablets being scheduled exceeds this value, skipped tablets will not undergo tablet balancing and repair checks.
- Default value: 10000

##### tablet_sched_disable_balance

- Meaning: Whether to disable Tablet balancing scheduling. Also known as `disable_balance`.
- Default value: FALSE

##### tablet_sched_disable_colocate_balance

- Meaning: Whether to disable replica balancing for Colocate Tables. Also known as `disable_colocate_balance`.
- Default value: FALSE

##### tablet_sched_max_balancing_tablets

- Meaning: The maximum number of tablets being balanced. If the number of tablets being balanced exceeds this value, tablet rebalancing will be skipped. Also known as `max_balancing_tablets`.
- Default value: 500

##### tablet_sched_balance_load_disk_safe_threshold

- Meaning: Threshold for determining whether the BE disk usage rate is balanced. This parameter is only effective when `tablet_sched_balancer_strategy` is set to `disk_and_tablet`.<br />If the disk usage rate of all BEs is below 50%, it is considered balanced.<br />For the disk_and_tablet strategy, if the difference between the maximum and minimum BE disk usage rates is greater than 10%, it is considered unbalanced, and will trigger tablet rebalancing. Also known as `balance_load_disk_safe_threshold`.
- Default value: 0.5

##### tablet_sched_balance_load_score_threshold

- Meaning: Used to determine whether BE load is balanced. This parameter is only effective when `tablet_sched_balancer_strategy` is set to `be_load_score`.<br />BEs with a load 10% lower than the average load are in a low load state, while BEs with a load 10% higher than the average load are in a high load state. Also known as `balance_load_score_threshold`.
- Default value: 0.1

##### tablet_sched_repair_delay_factor_second

- Meaning: Interval for FE to repair replicas. Also known as `tablet_repair_delay_factor_second`.
- Unit: seconds
- Default value: 60

##### tablet_sched_min_clone_task_timeout_sec

- Meaning: Minimum timeout duration for cloning tablets.
- Unit: seconds
- Default value: 3 \* 60

##### tablet_sched_max_clone_task_timeout_sec

- Meaning: Maximum timeout duration for cloning tablets. Also known as `max_clone_task_timeout_sec`.
- Unit: seconds
- Default value: 2 \* 60 \* 60

##### tablet_sched_max_not_being_scheduled_interval_ms

- Meaning: When scheduling tablet clones, if a tablet has not been scheduled for this duration, its scheduling priority will be raised to schedule it as soon as possible.
- Unit: milliseconds
- Default value: 15 \* 60 \* 100

#### Other Dynamic Parameters

##### plugin_enable

- Meaning: Whether the plugin function is enabled. Can only be installed/uninstalled on Leader FE.
- Default value: TRUE

##### max_small_file_number

- Meaning: The maximum number of small files allowed for storage.
- Default value: 100

##### max_small_file_size_bytes

- Meaning: The upper limit for the size of stored files.
- Unit: bytes
- Default value: 1024 \* 1024

##### agent_task_resend_wait_time_ms

- Meaning: Wait time for resending Agent tasks. The creation time of the agent task must have been set and it must have exceeded this value before the task can be resent.<br />This parameter prevents overly frequent sending of agent tasks.
- Unit: milliseconds
- Default value: 5000

##### backup_job_default_timeout_ms

- Meaning: Timeout duration for Backup jobs.
- Unit: milliseconds
- Default value: 86400*1000

##### enable_experimental_mv

- 含义：是否开启异步物化视图功能。`TRUE` 表示开启。从 2.5.2 版本开始，该功能默认开启。2.5.2 版本之前默认值为 `FALSE`。
- 默认值：TRUE

##### authentication_ldap_simple_bind_base_dn

- 含义：检索用户时，使用的 Base DN，用于指定 LDAP 服务器检索用户鉴权信息的起始点。
- 默认值：空字符串

##### authentication_ldap_simple_bind_root_dn

- 含义：检索用户时，使用的管理员账号的 DN。
- 默认值：空字符串

##### authentication_ldap_simple_bind_root_pwd

- 含义：检索用户时，使用的管理员账号的密码。
- 默认值：空字符串

##### authentication_ldap_simple_server_host

- 含义：LDAP 服务器所在主机的主机名。
- 默认值：空字符串

##### authentication_ldap_simple_server_port

- 含义：LDAP 服务器的端口。
- 默认值：389

##### authentication_ldap_simple_user_search_attr

- 含义：LDAP 对象中标识用户的属性名称。
- 默认值：uid

##### max_upload_task_per_be

- 含义：单次 BACKUP 操作下，系统向单个 BE 节点下发的最大上传任务数。设置为小于或等于 0 时表示不限制任务数。该参数自 v3.1.0 起新增。
- 默认值：0

##### max_download_task_per_be

- 含义：单次 RESTORE 操作下，系统向单个 BE 节点下发的最大下载任务数。设置为小于或等于 0 时表示不限制任务数。该参数自 v3.1.0 起新增。
- 默认值：0

##### allow_system_reserved_names

- 含义：是否允许用户创建以 `__op` 或 `__row` 开头命名的列。TRUE 表示启用此功能。请注意，在 StarRocks 中，这样的列名被保留用于特殊目的，创建这样的列可能导致未知行为，因此系统默认禁止使用这类名字。该参数自 v3.2.0 起新增。
- 默认值: FALSE

##### enable_backup_materialized_view

- 含义：在数据库的备份操作中，是否对数据库中的异步物化视图进行备份。如果设置为 `false`，将跳过对异步物化视图的备份。该参数自 v3.2.0 起新增。
- 默认值: TRUE

##### enable_colocate_mv_index

- 含义：在创建同步物化视图时，是否将同步物化视图的索引与基表加入到相同的 Colocate Group。如果设置为 `true`，TabletSink 将加速同步物化视图的写入性能。该参数自 v3.2.0 起新增。
- 默认值: TRUE

### 配置 FE 静态参数

以下 FE 配置项为静态参数，不支持在线修改，您需要在 `fe.conf` 中修改并重启 FE。

本节对 FE 静态参数做了如下分类：

- [Log](#log-fe-静态)
- [Server](#server-fe-静态)
- [元数据与集群管理](#元数据与集群管理fe-静态)
- [Query engine](#query-enginefe-静态)
- [导入和导出](#导入和导出fe-静态)
- [存储](#存储fe-静态)
- [其他动态参数](#其他动态参数)

#### Log (FE 静态)

##### log_roll_size_mb

- 含义：日志文件的大小。
- 单位：MB
- 默认值：1024，表示每个日志文件的大小为 1 GB。

##### sys_log_dir

- 含义：系统日志文件的保存目录。
- 默认值：`StarRocksFE.STARROCKS_HOME_DIR + "/log"`

##### sys_log_level

- 含义：系统日志的级别，从低到高依次为 `INFO`、`WARN`、`ERROR`、`FATAL`。
- 默认值：INFO

##### sys_log_verbose_modules

- 含义：打印系统日志的模块。如果设置参数取值为 `org.apache.starrocks.catalog`，则表示只打印 Catalog 模块下的日志。
- 默认值：空字符串

##### sys_log_roll_interval

- 含义：系统日志滚动的时间间隔。取值范围：`DAY` 和 `HOUR`。<br />取值为 `DAY` 时，日志文件名的后缀为 `yyyyMMdd`。取值为 `HOUR` 时，日志文件名的后缀为 `yyyyMMddHH`。
- 默认值：DAY

##### sys_log_delete_age

- 含义：系统日志文件的保留时长。默认值 `7d` 表示系统日志文件可以保留 7 天，保留时长超过 7 天的系统日志文件会被删除。
- 默认值：`7d`

##### sys_log_roll_num

- 含义：每个 `sys_log_roll_interval` 时间段内，允许保留的系统日志文件的最大数目。
- 默认值：10

##### audit_log_dir

- 含义：审计日志文件的保存目录。
- 默认值：`StarRocksFE.STARROCKS_HOME_DIR + "/log"`

##### audit_log_roll_num

- 含义：每个 `audit_log_roll_interval` 时间段内，允许保留的审计日志文件的最大数目。
- 默认值：90

##### audit_log_modules

- 含义：打印审计日志的模块。默认打印 slow_query 和 query 模块的日志。可以指定多个模块，模块名称之间用英文逗号加一个空格分隔。
- 默认值：slow_query, query

##### audit_log_roll_interval

- 含义：审计日志滚动的时间间隔。取值范围：`DAY` 和 `HOUR`。<br />取值为 `DAY` 时，日志文件名的后缀为 `yyyyMMdd`。取值为 `HOUR` 时，日志文件名的后缀为 `yyyyMMddHH`。
- 默认值：DAY

##### audit_log_delete_age

- 含义：审计日志文件的保留时长。- 默认值 `30d` 表示审计日志文件可以保留 30 天，保留时长超过 30 天的审计日志文件会被删除。
- 默认值：`30d`

##### dump_log_dir

- 含义：Dump 日志文件的保存目录。
- 默认值：`StarRocksFE.STARROCKS_HOME_DIR + "/log"`

##### dump_log_modules

- 含义：打印 Dump 日志的模块。默认打印 query 模块的日志。可以指定多个模块，模块名称之间用英文逗号加一个空格分隔。
- 默认值：query

##### dump_log_roll_interval

- 含义：Dump 日志滚动的时间间隔。取值范围：`DAY` 和 `HOUR`。<br />取值为 `DAY` 时，日志文件名的后缀为 `yyyyMMdd`。取值为 `HOUR` 时，日志文件名的后缀为 `yyyyMMddHH`。
- 默认值：DAY

##### dump_log_roll_num

- 含义：每个 `dump_log_roll_interval` 时间内，允许保留的 Dump 日志文件的最大数目。
- 默认值：10

##### dump_log_delete_age

- 含义：Dump 日志文件的保留时长。- 默认值 `7d` 表示 Dump 日志文件可以保留 7 天，保留时长超过 7 天的 Dump 日志文件会被删除。
- 默认值：`7d`

#### Server (FE 静态)

##### frontend_address

- 含义：FE 节点的 IP 地址。
- 默认值：0.0.0.0

##### priority_networks

- 含义：为那些有多个 IP 地址的服务器声明一个选择策略。 <br />请注意，最多应该有一个 IP 地址与此列表匹配。这是一个以分号分隔格式的列表，用 CIDR 表示法，例如 `10.10.10.0/24`。 如果没有匹配这条规则的ip，会随机选择一个。
- 默认值：空字符串

##### http_port

- 含义：FE 节点上 HTTP 服务器的端口。
- 默认值：8030

##### http_backlog_num

- 含义：HTTP 服务器支持的 Backlog 队列长度。
- 默认值：1024

##### cluster_name

- 含义：FE 所在 StarRocks 集群的名称，显示为网页标题。
- 默认值：StarRocks Cluster

##### rpc_port

- 含义：FE 节点上 Thrift 服务器的端口。
- 默认值：9020

##### thrift_backlog_num

- 含义：Thrift 服务器支持的 Backlog 队列长度。
- 默认值：1024

##### thrift_server_max_worker_threads

- 含义：Thrift 服务器支持的最大工作线程数。
- 默认值：4096

##### thrift_client_timeout_ms

- 含义：Thrift 客户端链接的空闲超时时间，即链接超过该时间无新请求后则将链接断开。
- 单位：毫秒。
- 默认值：5000

##### thrift_server_queue_size

- 含义：Thrift 服务器 pending 队列长度。如果当前处理线程数量超过了配置项 `thrift_server_max_worker_threads` 的值，则将超出的线程加入 pending 队列。
- 默认值：4096

##### brpc_idle_wait_max_time

- 含义：bRPC 的空闲等待时间。单位：毫秒。
- 默认值：10000

##### query_port

- 含义：FE 节点上 MySQL 服务器的端口。
- 默认值：9030

##### mysql_service_nio_enabled  

- 含义：是否开启 MySQL 服务器的异步 I/O 选项。
- 默认值：TRUE

##### mysql_service_io_threads_num

- 含义：MySQL 服务器中用于处理 I/O 事件的最大线程数。
- 默认值：4

##### mysql_nio_backlog_num

- 含义：MySQL 服务器支持的 Backlog 队列长度。
- 默认值：1024

##### max_mysql_service_task_threads_num

- 含义：MySQL 服务器中用于处理任务的最大线程数。
- 默认值：4096

##### mysql_server_version

- 含义：MySQL 服务器的版本。修改该参数配置会影响以下场景中返回的版本号：1. `select version();` 2. Handshake packet 版本 3. 全局变量 `version` 的取值 (`show variables like 'version';`)
- 默认值：5.1.0

##### max_connection_scheduler_threads_num

- 含义：连接调度器支持的最大线程数。
- 默认值：4096

##### qe_max_connection

- 含义：FE 支持的最大连接数，包括所有用户发起的连接。
- 默认值：1024

##### check_java_version

- 含义：检查已编译的 Java 版本与运行的 Java 版本是否兼容。<br />如果不兼容，则上报 Java 版本不匹配的异常信息，并终止启动。
- 默认值：TRUE

#### 元数据与集群管理（FE 静态）

##### meta_dir

- 含义：元数据的保存目录。
- 默认值：`StarRocksFE.STARROCKS_HOME_DIR + "/meta"`

##### heartbeat_mgr_threads_num

- 含义：Heartbeat Manager 中用于发送心跳任务的最大线程数。
- 默认值：8

##### heartbeat_mgr_blocking_queue_size

- 含义：Heartbeat Manager 中存储心跳任务的阻塞队列大小。
- 默认值：1024

##### metadata_failure_recovery

- 含义：是否强制重置 FE 的元数据。请谨慎使用该配置项。
- 默认值：FALSE

##### edit_log_port

- 含义：FE 所在 StarRocks 集群中各 Leader FE、Follower FE、Observer FE 之间通信用的端口。
- 默认值：9010

##### edit_log_type

- 含义：编辑日志的类型。取值只能为 `BDB`。
- 默认值：BDB

##### bdbje_heartbeat_timeout_second

- 含义：FE 所在 StarRocks 集群中 Leader FE 和 Follower FE 之间的 BDB JE 心跳超时时间。
- 单位：秒。
- 默认值：30

##### bdbje_lock_timeout_second

- 含义：BDB JE 操作的锁超时时间。
- 单位：秒。
- 默认值：1

##### max_bdbje_clock_delta_ms

- 含义：FE 所在 StarRocks 集群中 Leader FE 与非 Leader FE 之间能够容忍的最大时钟偏移。
- 单位：毫秒。
- 默认值：5000

##### txn_rollback_limit

- 含义：允许回滚的最大事务数。
- 默认值：100

##### bdbje_replica_ack_timeout_second  

- 含义：FE 所在 StarRocks 集群中，元数据从 Leader FE 写入到多个 Follower FE 时，Leader FE 等待足够多的 Follower FE 发送 ACK 消息的超时时间。当写入的元数据较多时，可能返回 ACK 的时间较长，进而导致等待超时。如果超时，会导致写元数据失败，FE 进程退出，此时可以适当地调大该参数取值。
- 单位：秒
- 默认值：10

##### master_sync_policy

- 含义：FE 所在 StarRocks 集群中，Leader FE 上的日志刷盘方式。该参数仅在当前 FE 为 Leader 时有效。取值范围： <ul><li>`SYNC`：事务提交时同步写日志并刷盘。</li><li> `NO_SYNC`：事务提交时不同步写日志。</li><li> `WRITE_NO_SYNC`：事务提交时同步写日志，但是不刷盘。 </li></ul>如果您只部署了一个 Follower FE，建议将其设置为 `SYNC`。 如果您部署了 3 个及以上 Follower FE，建议将其与下面的 `replica_sync_policy` 均设置为 `WRITE_NO_SYNC`。
- 默认值：SYNC

##### replica_sync_policy

- 含义：FE 所在 StarRocks 集群中，Follower FE 上的日志刷盘方式。取值范围： <ul><li>`SYNC`：事务提交时同步写日志并刷盘。</li><li> `NO_SYNC`：事务提交时不同步写日志。</li><li> `WRITE_NO_SYNC`：事务提交时同步写日志，但是不刷盘。</li></ul>
- 默认值：SYNC

##### replica_ack_policy

- 含义：判定日志是否有效的策略，默认是多数 Follower FE 返回确认消息，就认为生效。
- 默认值：SIMPLE_MAJORITY

##### cluster_id

- 含义：FE 所在 StarRocks 集群的 ID。具有相同集群 ID 的 FE 或 BE 属于同一个 StarRocks 集群。取值范围：正整数。- 默认值 `-1` 表示在 Leader FE 首次启动时随机生成一个。
- 默认值：-1

#### Query engine（FE 静态）

##### publish_version_interval_ms

- 含义：两个版本发布操作之间的时间间隔。
- 单位：毫秒
- 默认值：10

##### statistic_cache_columns

- 含义：缓存统计信息表的最大行数。
- 默认值：100000

#### Import and Export (FE Static)

##### load_checker_interval_second

- 含义：Import job polling interval.
- 单位：second.
- 默认值：5  

##### transaction_clean_interval_second

- 含义：Cleaning interval for completed transactions. It is recommended to keep the cleaning interval as short as possible to ensure that completed transactions are cleaned in a timely manner.
- 单位：second.
- 默认值：30

##### label_clean_interval_second

- 含义：Cleaning interval for job labels. It is recommended to keep the cleaning interval as short as possible to ensure that labels of historical jobs are cleaned in a timely manner.
- 单位：second.
- 默认值：14400

##### spark_dpp_version

- 含义：Version of Spark DPP feature.
- 默认值：1.0.0

##### spark_resource_path

- 含义：Root directory of Spark dependencies.
- 默认值：Empty string

##### spark_launcher_log_dir

- 含义：Directory for saving Spark logs.
- 默认值：`sys_log_dir + "/spark_launcher_log"`

##### yarn_client_path

- 含义：Root directory of Yarn client.
- 默认值：`StarRocksFE.STARROCKS_HOME_DIR + "/lib/yarn-client/hadoop/bin/yarn"`

##### yarn_config_dir

- 含义：Directory for saving Yarn configuration files.
- 默认值：`StarRocksFE.STARROCKS_HOME_DIR + "/lib/yarn-config"`

##### export_checker_interval_second

- 含义：Scheduling interval of export job scheduler.
- 默认值：5

##### export_task_pool_size

- 含义：Size of the export task thread pool.
- 默认值：5

##### broker_client_timeout_ms

- 含义：Default timeout for Broker RPC.
- 单位：millisecond
- 默认值 10s

#### Storage (FE Static)

##### tablet_sched_balancer_strategy

- 含义：Tablet balancing strategy. Parameter alias is `tablet_balancer_strategy`. Value range: `disk_and_tablet` and `be_load_score`.
- 默认值：`disk_and_tablet`

##### tablet_sched_storage_cooldown_second

- 含义：From the time point of Table creation, the automatic cool-down delay based on time. Cool-down refers to the migration from SSD medium to HDD medium.<br />Parameter alias is `storage_cooldown_second`. The default value of `-1` indicates no automatic cool-down. If you want to enable the automatic cool-down feature, please explicitly set the parameter value to be greater than 0.
- 单位：second.
- 默认值：-1

##### tablet_stat_update_interval_second

- 含义：Interval at which FE requests each BE to collect Tablet statistics.
- 单位：second.
- 默认值：300

#### StarRocks Storage and Computing Separation Cluster

##### run_mode

- 含义：Operation mode of StarRocks cluster. Valid values: `shared_data` and `shared_nothing` (default). `shared_data` indicates running StarRocks in storage-computing separation mode. `shared_nothing` indicates running StarRocks in storage-computing integrated mode.<br />**Note**<br />StarRocks cluster does not support a mixed deployment of storage-computing separation and storage-computing integrated modes.<br />Do not change `run_mode` after the cluster deployment, otherwise the cluster will be unable to start again. It does not support converting from a storage-computing integrated cluster to a storage-computing separation cluster, and vice versa.
- 默认值：shared_nothing

##### cloud_native_meta_port

- 含义：Port on which the cloud-native metadata service listens.
- 默认值：6090

##### cloud_native_storage_type

- 含义：The type of storage you are using. In the storage-computing separation mode, StarRocks supports storing data in HDFS, Azure Blob (supported since v3.1.1), and object storage compatible with the S3 protocol (such as AWS S3, Google GCP, Alibaba Cloud OSS, and MinIO). Valid values: `S3` (default), `AZBLOB`, and `HDFS`. If you specify `S3` for this item, you must add configurations prefixed with `aws_s3`. If you specify `AZBLOB` for this item, you must add configurations prefixed with `azure_blob`. If you specify `HDFS` for this item, you only need to specify `cloud_native_hdfs_url`.
- 默认值：S3

##### cloud_native_hdfs_url

- 含义：URL of the HDFS storage, for example, `hdfs://127.0.0.1:9000/user/xxx/starrocks/`.

##### aws_s3_path

- 含义：Path of the S3 storage space used for storing data, composed of the name of the S3 bucket and its sub-path (if any), such as `testbucket/subpath`.

##### aws_s3_region

- 含义：Region of the S3 storage space to be accessed, such as `us-west-2`.

##### aws_s3_endpoint

- 含义：Connection address for accessing the S3 storage space, such as `https://s3.us-west-2.amazonaws.com`.

##### aws_s3_use_aws_sdk_default_behavior

- 含义：Whether to use the default authentication credentials of AWS SDK. Valid values: `true` and `false` (default).
- 默认值：false

##### aws_s3_use_instance_profile

- 含义：Whether to use an Instance Profile or Assumed Role as secure credentials to access S3. Valid values: `true` and `false` (default).<ul><li>If you are using IAM user credentials (Access Key and Secret Key) to access S3, you need to set this item to `false` and specify `aws_s3_access_key` and `aws_s3_secret_key`.</li><li>If you are using an Instance Profile to access S3, you need to set this item to `true`.</li><li>If you are using an Assumed Role to access S3, you need to set this item to `true` and specify `aws_s3_iam_role_arn`.</li><li>If you are using an external AWS account to access S3 through Assumed Role authentication, you need to specify `aws_s3_external_id` additionally.</li></ul>
- 默认值：false

##### aws_s3_access_key

- 含义：Access Key for accessing the S3 storage space.

##### aws_s3_secret_key

- 含义：Secret Key for accessing the S3 storage space.

##### aws_s3_iam_role_arn

- 含义：ARN of the IAM Role with access to the S3 storage space.

##### aws_s3_external_id

- 含义：External ID for accessing the S3 storage space across AWS accounts.

##### azure_blob_path

- 含义：Path of the Azure Blob Storage used for storing data, composed of the container name in the Storage Account and the sub-path (if any), such as `testcontainer/subpath`.

##### azure_blob_endpoint

- 含义：Connection address of the Azure Blob Storage, such as `https://test.blob.core.windows.net`.

##### azure_blob_shared_key

- 含义：Shared Key for accessing the Azure Blob Storage.

##### azure_blob_sas_token

- 含义：Shared Access Signature (SAS) for accessing the Azure Blob Storage.
- 默认值：N/A

#### Other Static Parameters

##### plugin_dir

- 含义：Installation directory of plugins.
- 默认值：`STARROCKS_HOME_DIR/plugins`

##### small_file_dir

- 含义：小文件的根目录。
- 默认值：`StarRocksFE.STARROCKS_HOME_DIR + "/small_files"`

##### max_agent_task_threads_num

- 含义：代理任务线程池中用于处理代理任务的最大线程数。
- 默认值：4096

##### auth_token

- 含义：用于内部身份验证的集群令牌。为空则在 Leader FE 第一次启动时随机生成一个。
- 默认值：空字符串

##### tmp_dir

- 含义：临时文件的保存目录，例如备份和恢复过程中产生的临时文件。<br />这些过程完成以后，所产生的临时文件会被清除掉。
- 默认值：`StarRocksFE.STARROCKS_HOME_DIR + "/temp_dir"`

##### locale

- 含义：FE 所使用的字符集。
- 默认值：zh_CN.UTF-8

##### hive_meta_load_concurrency

- 含义：Hive 元数据支持的最大并发线程数。
- 默认值：4

##### hive_meta_cache_refresh_interval_s

- 含义：刷新 Hive 外表元数据缓存的时间间隔。
- 单位：秒。
- 默认值：7200

##### hive_meta_cache_ttl_s

- 含义：Hive 外表元数据缓存的失效时间。
- 单位：秒。
- 默认值：86400

##### hive_meta_store_timeout_s

- 含义：连接 Hive Metastore 的超时时间。
- 单位：秒。
- 默认值：10

##### es_state_sync_interval_second

- 含义：FE 获取 Elasticsearch Index 和同步 StarRocks 外部表元数据的时间间隔。
- 单位：秒
- 默认值：10

##### enable_auth_check

- 含义：是否开启鉴权检查功能。取值范围：`TRUE` 和 `FALSE`。`TRUE` 表示开该功能。`FALSE`表示关闭该功能。
- 默认值：TRUE

##### enable_metric_calculator

- 含义：是否开启定期收集指标 (Metrics) 的功能。取值范围：`TRUE` 和 `FALSE`。`TRUE` 表示开该功能。`FALSE`表示关闭该功能。
- 默认值：TRUE

## BE 配置项

部分 BE 节点配置项为动态参数，您可以通过命令在线修改。其他配置项为静态参数，需要通过修改 **be.conf** 文件后重启 BE 服务使相关修改生效。

### 查看 BE 配置项

您可以通过以下命令查看 BE 配置项：

```shell
curl http://<BE_IP>:<BE_HTTP_PORT>/varz
```

### 配置 BE 动态参数

您可以通过 `curl` 命令在线修改 BE 节点动态参数。

```shell
curl -XPOST http://be_host:http_port/api/update_config?configuration_item=value
```

以下是 BE 动态参数列表。

#### report_task_interval_seconds

- 含义：汇报单个任务的间隔。建表，删除表，导入，schema change 都可以被认定是任务。
- 单位：秒
- 默认值：10

#### report_disk_state_interval_seconds

- 含义：汇报磁盘状态的间隔。汇报各个磁盘的状态，以及其中数据量等。
- 单位：秒
- 默认值：60

#### report_tablet_interval_seconds

- 含义：汇报 tablet 的间隔。汇报所有的 tablet 的最新版本。
- 单位：秒
- 默认值：60

#### report_workgroup_interval_seconds

- 含义：汇报 workgroup 的间隔。汇报所有 workgroup 的最新版本。
- 单位：秒
- 默认值：5

#### max_download_speed_kbps

- 含义：单个 HTTP 请求的最大下载速率。这个值会影响 BE 之间同步数据副本的速度。
- 单位：KB/s
- 默认值：50000

#### download_low_speed_limit_kbps

- 含义：单个 HTTP 请求的下载速率下限。如果在 `download_low_speed_time` 秒内下载速度一直低于`download_low_speed_limit_kbps`，那么请求会被终止。
- 单位：KB/s
- 默认值：50

#### download_low_speed_time

- 含义：见 `download_low_speed_limit_kbps`。
- 单位：秒
- 默认值：300

#### status_report_interval

- 含义：查询汇报 profile 的间隔，用于 FE 收集查询统计信息。
- 单位：秒
- 默认值：5

#### scanner_thread_pool_thread_num

- 含义：存储引擎并发扫描磁盘的线程数，统一管理在线程池中。
- 默认值：48

#### thrift_client_retry_interval_ms

- 含义：Thrift client 默认的重试时间间隔。
- 单位：毫秒
- 默认值：100

#### scanner_thread_pool_queue_size

- 含义：存储引擎支持的扫描任务数。
- 默认值：102400

#### scanner_row_num

- 含义：每个扫描线程单次执行最多返回的数据行数。
- 默认值：16384

#### max_scan_key_num

- 含义：查询最多拆分的 scan key 数目。
- 默认值：1024

#### max_pushdown_conditions_per_column

- 含义：单列上允许下推的最大谓词数量，如果超出数量限制，谓词不会下推到存储层。
- 默认值：1024

#### exchg_node_buffer_size_bytes  

- 含义：Exchange 算子中，单个查询在接收端的 buffer 容量。<br />这是一个软限制，如果数据的发送速度过快，接收端会触发反压来限制发送速度。
- 单位：字节
- 默认值：10485760

#### column_dictionary_key_ratio_threshold

- 含义：字符串类型的取值比例，小于这个比例采用字典压缩算法。
- 单位：%
- 默认值：0

#### memory_limitation_per_thread_for_schema_change

- 含义：单个 schema change 任务允许占用的最大内存。
- 单位：GB
- 默认值：2

#### update_cache_expire_sec

- 含义：Update Cache 的过期时间。
- 单位：秒
- 默认值：360

#### file_descriptor_cache_clean_interval

- 含义：文件句柄缓存清理的间隔，用于清理长期不用的文件句柄。
- 单位：秒
- 默认值：3600

#### disk_stat_monitor_interval

- 含义：磁盘健康状态检测的间隔。
- 单位：秒
- 默认值：5

#### unused_rowset_monitor_interval

- 含义：清理过期 Rowset 的时间间隔。
- 单位：秒
- 默认值：30

#### max_percentage_of_error_disk  

- 含义：错误磁盘达到一定比例，BE 退出。
- 单位：%
- 默认值：0

#### default_num_rows_per_column_file_block

- 含义：每个 row block 最多存放的行数。
- 默认值：1024

#### pending_data_expire_time_sec  

- 含义：存储引擎保留的未生效数据的最大时长。
- 单位：秒
- 默认值：1800

#### inc_rowset_expired_sec

- 含义：导入生效的数据，存储引擎保留的时间，用于增量克隆。
- 单位：秒
- 默认值：1800

#### tablet_rowset_stale_sweep_time_sec

- 含义：失效 rowset 的清理间隔。
- 单位：秒
- 默认值：1800

#### snapshot_expire_time_sec

- 含义：快照文件清理的间隔，默认 48 个小时。
- 单位：秒
- 默认值：172800

#### trash_file_expire_time_sec

- 含义：回收站清理的间隔，默认 72 个小时。
- 单位：秒
- 默认值：259200

#### base_compaction_check_interval_seconds

- 含义：Base Compaction 线程轮询的间隔。
- 单位：秒
- 默认值：60

#### min_base_compaction_num_singleton_deltas

- 含义：触发 BaseCompaction 的最小 segment 数。
- 默认值：5

#### max_base_compaction_num_singleton_deltas

- 含义：单次 BaseCompaction 合并的最大 segment 数。
- 默认值：100

#### base_compaction_interval_seconds_since_last_operation

- 含义：上一轮 BaseCompaction 距今的间隔，是触发 BaseCompaction 条件之一。
- 单位：秒
- 默认值：86400

#### cumulative_compaction_check_interval_seconds

- 含义：CumulativeCompaction 线程轮询的间隔。
- 单位：秒
- 默认值：1

#### update_compaction_check_interval_seconds

- 含义：Primary key 模型 Update compaction 的检查间隔。
- 单位：秒
- 默认值：60

#### min_compaction_failure_interval_sec

- 含义：Tablet Compaction 失败之后，再次被调度的间隔。
- 单位：秒
- 默认值：120

#### periodic_counter_update_period_ms

- 含义：Counter 统计信息的间隔。
- 单位：毫秒
- 默认值：500

#### load_error_log_reserve_hours  

- 含义：导入数据信息保留的时长。
- 单位：小时
- 默认值：48

#### streaming_load_max_mb

- 含义：流式导入单个文件大小的上限。自 3.0 版本起，默认值由 10240 变为 102400。
- 单位：MB
- 默认值：102400

#### streaming_load_max_batch_size_mb  

- 含义：流式导入单个 JSON 文件大小的上限。
- 单位：MB
- 默认值：100

#### memory_maintenance_sleep_time_s

- 含义：触发 ColumnPool GC 任务的时间间隔。StarRocks 会周期运行 GC 任务，尝试将空闲内存返还给操作系统。
- 单位：秒
- 默认值：10

#### write_buffer_size

- 含义：MemTable 在内存中的 buffer 大小，超过这个限制会触发 flush。
- 单位：字节
- 默认值：104857600

#### tablet_stat_cache_update_interval_second

- 含义：Tablet Stat Cache 的更新间隔。
- 单位：秒
- 默认值：300

#### result_buffer_cancelled_interval_time

- 含义：BufferControlBlock 释放数据的等待时间。
- 单位：秒
- 默认值：300

#### thrift_rpc_timeout_ms

- 含义：Thrift 超时的时长。
- 单位：毫秒
- 默认值：5000

#### max_consumer_num_per_group

- 含义：Routine load 中，每个consumer group 内最大的 consumer 数量。
- 默认值：3

#### max_memory_sink_batch_count

- 含义：Scan cache 的最大缓存批次数量。
- 默认值：20

#### scan_context_gc_interval_min

- 含义：Scan context 的清理间隔。
- 单位：分钟
- 默认值：5

#### path_gc_check_step

- 含义：单次连续 scan 最大的文件数量。
- 默认值：1000

#### path_gc_check_step_interval_ms

- 含义：多次连续 scan 文件间隔时间。
- 单位：毫秒
- 默认值：10

#### path_scan_interval_second

- 含义：gc 线程清理过期数据的间隔时间。
- 单位：秒
- 默认值：86400

#### storage_flood_stage_usage_percent

- 含义：如果空间使用率超过该值且剩余空间小于 `storage_flood_stage_left_capacity_bytes`，会拒绝 Load 和 Restore 作业。
- 单位：%
- 默认值：95

#### storage_flood_stage_left_capacity_bytes

- 含义：如果剩余空间小于该值且空间使用率超过 `storage_flood_stage_usage_percent`，会拒绝 Load 和 Restore 作业，默认 100GB。
- 单位：字节
- 默认值：107374182400

#### tablet_meta_checkpoint_min_new_rowsets_num  

- 含义：自上次 TabletMeta Checkpoint 至今新创建的 rowset 数量。
- 默认值：10

#### tablet_meta_checkpoint_min_interval_secs

- 含义：TabletMeta Checkpoint 线程轮询的时间间隔。
- 单位：秒
- 默认值：600

#### max_runnings_transactions_per_txn_map

- 含义：每个分区内部同时运行的最大事务数量。
- 默认值：100

#### tablet_max_pending_versions

- 含义：Primary Key 表每个 tablet 上允许已提交 (committed) 但是未 apply 的最大版本数。
- 默认值：1000

#### tablet_max_versions

- 含义：每个 tablet 上允许的最大版本数。如果超过该值，新的写入请求会失败。
- 默认值：1000

#### alter_tablet_worker_count

- 含义：进行 schema change 的线程数。自 2.5 版本起，该参数由静态变为动态。
- 默认值：3

#### max_hdfs_file_handle

- 含义：最多可以打开的 HDFS 文件句柄数量。
- 默认值：1000

#### be_exit_after_disk_write_hang_second

- 含义：磁盘挂起后触发 BE 进程退出的等待时间。
- 单位：秒
- 默认值：60

#### min_cumulative_compaction_failure_interval_sec

- 含义：Cumulative Compaction 失败后的最小重试间隔。
- 单位：秒
- 默认值：30

#### size_tiered_level_num

- 含义：Size-tiered Compaction 策略的 level 数量。每个 level 最多保留一个 rowset，因此稳定状态下最多会有和 level 数相同的 rowset。 |
- 默认值：7

#### size_tiered_level_multiple

- 含义：Size-tiered Compaction 策略中，相邻两个 level 之间相差的数据量的倍数。
- 默认值：5

#### size_tiered_min_level_size

- 含义：Size-tiered Compaction 策略中，最小 level 的大小，小于此数值的 rowset 会直接触发 compaction。
- 单位：字节
- 默认值：131072

#### storage_page_cache_limit

- 含义：PageCache 的容量，STRING，可写为容量大小，例如： `20G`、`20480M`、`20971520K` 或 `21474836480B`。也可以写为 PageCache 占系统内存的比例，例如，`20%`。该参数仅在 `disable_storage_page_cache` 为 `false` 时生效。
- 默认值：20%

#### disable_storage_page_cache

- 含义：是否开启 PageCache。开启 PageCache 后，StarRocks 会缓存最近扫描过的数据，对于查询重复性高的场景，会大幅提升查询效率。`true` 表示不开启。自 2.4 版本起，该参数默认值由 `true` 变更为 `false`。自 3.1 版本起，该参数由静态变为动态。
- 默认值：FALSE

#### max_compaction_concurrency

- 含义：Compaction 线程数上限（即 BaseCompaction + CumulativeCompaction 的最大并发）。该参数防止 Compaction 占用过多内存。-1 代表没有限制。0 表示不允许 compaction。
- 默认值：-1

#### internal_service_async_thread_num

- 含义：单个 BE 上与 Kafka 交互的线程池大小。当前 Routine Load FE 与 Kafka 的交互需经由 BE 完成，而每个 BE 上实际执行操作的是一个单独的线程池。当 Routine Load 任务较多时，可能会出现线程池线程繁忙的情况，可以调整该配置。
- 默认值：10

#### update_compaction_ratio_threshold

- 含义：存算分离集群下主键模型表单次 Compaction 可以合并的最大数据比例。如果单个 Tablet 过大，建议适当调小该配置项取值。自 v3.1.5 起支持。
- 默认值：0.5

#### max_garbage_sweep_interval

- 含义：磁盘进行垃圾清理的最大间隔。自 3.0 版本起，该参数由静态变为动态。
- 单位：秒
- 默认值：3600

#### min_garbage_sweep_interval

- 含义：磁盘进行垃圾清理的最小间隔。自 3.0 版本起，该参数由静态变为动态。
- 单位：秒
- 默认值：180

### 配置 BE 静态参数

以下 BE 配置项为静态参数，不支持在线修改，您需要在 **be.conf** 中修改并重启 BE 服务。

#### hdfs_client_enable_hedged_read

- 含义：指定是否开启 Hedged Read 功能。
- 默认值：false
- 引入版本：3.0

#### hdfs_client_hedged_read_threadpool_size  

- 含义：指定 HDFS 客户端侧 Hedged Read 线程池的大小，即 HDFS 客户端侧允许有多少个线程用于服务 Hedged Read。该参数从 3.0 版本起支持，对应 HDFS 集群配置文件 `hdfs-site.xml` 中的 `dfs.client.hedged.read.threadpool.size` 参数。
- 默认值：128
- 引入版本：3.0

#### hdfs_client_hedged_read_threshold_millis

- 含义：指定发起 Hedged Read 请求前需要等待多少毫秒。例如，假设该参数设置为 `30`，那么如果一个 Read 任务未能在 30 毫秒内返回结果，则 HDFS 客户端会立即发起一个 Hedged Read，从目标数据块的副本上读取数据。该参数从 3.0 版本起支持，对应 HDFS 集群配置文件 `hdfs-site.xml` 中的 `dfs.client.hedged.read.threshold.millis` 参数。
- 单位：毫秒
- 默认值：2500
- 引入版本：3.0

#### be_port

- 含义：BE 上 thrift server 的端口，用于接收来自 FE 的请求。
- 默认值：9060

#### brpc_port

- 含义：bRPC 的端口，可以查看 bRPC 的一些网络统计信息。
- 默认值：8060

#### brpc_num_threads

- 含义：bRPC 的 bthreads 线程数量，-1 表示和 CPU 核数一样。
- 默认值：-1

#### priority_networks

- 含义：以 CIDR 形式 10.10.10.0/24 指定 BE IP 地址，适用于机器有多个 IP，需要指定优先使用的网络。
- 默认值：空字符串

#### starlet_port

- 含义：存算分离集群中 CN（v3.0 中的 BE）的额外 Agent 服务端口。
- 默认值：9070

#### heartbeat_service_port

- 含义：心跳服务端口（thrift），用户接收来自 FE 的心跳。
- 默认值：9050

#### heartbeat_service_thread_count

- 含义：心跳线程数。
- 默认值：1

#### create_tablet_worker_count

- 含义：创建 tablet 的线程数。
- 默认值：3

#### drop_tablet_worker_count

- 含义：删除 tablet 的线程数。
- 默认值：3

#### push_worker_count_normal_priority

- 含义：导入线程数，处理 NORMAL 优先级任务。
- 默认值：3

#### push_worker_count_high_priority

- 含义：导入线程数，处理 HIGH 优先级任务。
- 默认值：3

#### transaction_publish_version_worker_count

- 含义：生效版本的最大线程数。当该参数被设置为小于或等于 `0` 时，系统默认使用 CPU 核数的一半，以避免因使用固定值而导致在导入并行较高时线程资源不足。自 2.5 版本起，默认值由 `8` 变更为 `0`。
- 默认值：0

#### clear_transaction_task_worker_count

- 含义：清理事务的线程数。
- 默认值：1

#### clone_worker_count

- 含义：克隆的线程数。
- 默认值：3

#### storage_medium_migrate_count

- 含义：介质迁移的线程数，SATA 迁移到 SSD。
- 默认值：1

#### check_consistency_worker_count

- 含义：计算 tablet 的校验和 (checksum)。
- 默认值：1

#### sys_log_dir

- 含义：存放日志的地方，包括 INFO，WARNING，ERROR，FATAL 等日志。
- 默认值：`${STARROCKS_HOME}/log`

#### user_function_dir

- 含义：UDF 程序存放的路径。
- 默认值：`${STARROCKS_HOME}/lib/udf`

#### small_file_dir

- 含义：保存文件管理器下载的文件的目录。
- 默认值：`${STARROCKS_HOME}/lib/small_file`

#### sys_log_level

- 含义：日志级别，INFO < WARNING < ERROR < FATAL。
- 默认值：INFO

#### sys_log_roll_mode

- 含义：日志拆分的大小，每 1G 拆分一个日志。
- 默认值：SIZE-MB-1024

#### sys_log_roll_num

- 含义：日志保留的数目。
- 默认值：10

#### sys_log_verbose_modules

- 含义：日志打印的模块。有效值为 BE 的 namespace，包括 `starrocks`、`starrocks::debug`、`starrocks::fs`、`starrocks::io`、`starrocks::lake`、`starrocks::pipeline`、`starrocks::query_cache`、`starrocks::stream` 以及 `starrocks::workgroup`。
- 默认值：空字符串

#### sys_log_verbose_level

- 含义：日志显示的级别，用于控制代码中 VLOG 开头的日志输出。
- 默认值：10

#### log_buffer_level

- 含义：日志刷盘的策略，默认保持在内存中。
- 默认值：空字符串

#### num_threads_per_core

- 含义：每个 CPU core 启动的线程数。
- 默认值：3

#### compress_rowbatches

- 含义：BE 之间 RPC 通信是否压缩 RowBatch，用于查询层之间的数据传输。
- 默认值：TRUE

#### serialize_batch

- 含义：BE 之间 RPC 通信是否序列化 RowBatch，用于查询层之间的数据传输。
- 默认值：FALSE

#### storage_root_path

- 含义: 存储数据的目录以及存储介质类型，多块盘配置使用分号 `;` 隔开。<br />如果为 SSD 磁盘，需在路径后添加 `,medium:ssd`，如果为 HDD 磁盘，需在路径后添加 `,medium:hdd`。例如：`/data1,medium:hdd;/data2,medium:ssd`。
- 默认值: `${STARROCKS_HOME}/storage`

#### max_length_for_bitmap_function

- 含义: bitmap 函数输入值的最大长度。
- 单位: 字节。
- 默认值: 1000000

#### max_length_for_to_base64

- 含义: to_base64() 函数输入值的最大长度。
- 单位: 字节
- 默认值: 200000

#### max_tablet_num_per_shard

- 含义: 每个 shard 的 tablet 数目，用于划分 tablet，防止单个目录下 tablet 子目录过多。
- 默认值: 1024

#### file_descriptor_cache_capacity

- 含义: 文件句柄缓存的容量。
- 默认值: 16384
  
#### min_file_descriptor_number

- 含义: BE 进程的文件句柄 limit 要求的下线。
- 默认值: 60000

#### index_stream_cache_capacity

- 含义: BloomFilter/Min/Max 等统计信息缓存的容量。
- 默认值: 10737418240

#### base_compaction_num_threads_per_disk

- 含义: 每个磁盘 BaseCompaction 线程的数目。
- 默认值: 1

#### base_cumulative_delta_ratio

- 含义: BaseCompaction 触发条件之一：Cumulative 文件大小达到 Base 文件的比例。
- 默认值: 0.3

#### compaction_trace_threshold

- 含义: 单次 Compaction 打印 trace 的时间阈值，如果单次 compaction 时间超过该阈值就打印 trace。
- 单位: 秒
- 默认值: 60

#### be_http_port

- 含义: HTTP Server 端口。
- 默认值: 8040

#### webserver_num_workers

- 含义: HTTP Server 线程数。
- 默认值: 48

#### load_data_reserve_hours

- 含义: 小批量导入生成的文件保留的时。
- 默认值: 4

#### number_tablet_writer_threads

- 含义: 流式导入的线程数。
- 默认值: 16

#### streaming_load_rpc_max_alive_time_sec

- 含义: 流式导入 RPC 的超时时间。
- 默认值: 1200

#### fragment_pool_thread_num_min

- 含义: 最小查询线程数，默认启动 64 个线程。
- 默认值: 64

#### fragment_pool_thread_num_max

- 含义: 最大查询线程数。
- 默认值: 4096

#### fragment_pool_queue_size

- 含义: 单节点上能够处理的查询请求上限。
- 默认值: 2048

#### enable_token_check

- 含义: Token 开启检验。
- 默认值: TRUE

#### enable_prefetch

- 含义: 查询提前预取。
- 默认值: TRUE

#### load_process_max_memory_limit_bytes

- 含义: 单节点上所有的导入线程占据的内存上限，100GB。
- 单位: 字节
- 默认值: 107374182400

#### load_process_max_memory_limit_percent

- 含义: 单节点上所有的导入线程占据的内存上限比例。
- 默认值: 30

#### sync_tablet_meta

- 含义: 存储引擎是否开 sync 保留到磁盘上。
- 默认值: FALSE

#### routine_load_thread_pool_size

- 含义: 单节点上 Routine Load 线程池大小。从 3.1.0 版本起，该参数已经废弃。单节点上 Routine Load 线程池大小完全由 FE 动态参数`max_routine_load_task_num_per_be` 控制。
- 默认值: 10

#### brpc_max_body_size

- 含义: bRPC 最大的包容量。
- 单位: 字节
- 默认值: 2147483648

#### tablet_map_shard_size

- 含义: Tablet 分组数。
- 默认值: 32

#### enable_bitmap_union_disk_format_with_set

- 含义: Bitmap 新存储格式，可以优化 bitmap_union 性能。
- 默认值: FALSE

#### mem_limit

- 含义: BE 进程内存上限。可设为比例上限（如 "80%"）或物理上限（如 "100GB"）。
- 默认值: 90%

#### flush_thread_num_per_store

- 含义: 每个 Store 用以 Flush MemTable 的线程数。
- 默认值: 2

#### datacache_enable

- 含义: 是否启用 Data Cache。<ul><li>`true`：启用。</li><li>`false`：不启用，为默认值。</li></ul> 如要启用，设置该参数值为 `true`。
- 默认值: false

#### datacache_disk_path  

- 含义: 磁盘路径。支持添加多个路径，多个路径之间使用分号(;) 隔开。建议 BE 机器有几个磁盘即添加几个路径。
- 默认值: N/A

#### datacache_meta_path  

- 含义: Block 的元数据存储目录，可自定义。推荐创建在 **`$STARROCKS_HOME`** 路径下。
- 默认值: N/A

#### datacache_block_size

- 含义: 单个 block 大小，单位：字节。默认值为 `1048576`，即 1 MB。
- 默认值: 1048576

#### datacache_mem_size

- 含义: 内存缓存数据量的上限，可设为比例上限（如 `10%`）或物理上限（如 `10G`, `21474836480` 等）。默认值为 `10%`。推荐将该参数值最低设置成 10 GB。
- 单位: N/A
- 默认值: 10%

#### datacache_disk_size

- 含义: 单个磁盘缓存数据量的上限，可设为比例上限（如 `80%`）或物理上限（如 `2T`, `500G` 等）。举例：在 `datacache_disk_path` 中配置了 2 个磁盘，并设置 `datacache_disk_size` 参数值为 `21474836480`，即 20 GB，那么最多可缓存 40 GB 的磁盘数据。默认值为 `0`，即仅使用内存作为缓存介质，不使用磁盘。
- 单位: N/A
- 默认值: 0

#### jdbc_connection_pool_size

- 含义: JDBC 连接池大小。每个 BE 节点上访问 `jdbc_url` 相同的外表时会共用同一个连接池。
- 默认值: 8

#### jdbc_minimum_idle_connections  

- 含义: JDBC 连接池中最少的空闲连接数量。
- 默认值: 1

#### jdbc_connection_idle_timeout_ms  

- 含义: JDBC 空闲连接超时时间。如果 JDBC 连接池内的连接空闲时间超过此值，连接池会关闭超过 `jdbc_minimum_idle_connections` 配置项中指定数量的空闲连接。
- 单位: 毫秒
- 默认值: 600000

#### query_cache_capacity  

- 含义: 指定 Query Cache 的大小。单位: 字节。默认为 512 MB。最小不低于 4 MB。如果当前的 BE 内存容量无法满足您期望的 Query Cache 大小，可以增加 BE 的内存容量，然后再设置合理的 Query Cache 大小。<br />每个 BE 都有自己私有的 Query Cache 存储空间，BE 只 Populate 或 Probe 自己本地的 Query Cache 存储空间。
- 单位: 字节
- 默认值：536870912

#### enable_event_based_compaction_framework  

- 含义：是否开启基于事件的压实框架。`true` 代表开启。`false` 代表关闭。开启则能够在 tablet 数比较多或者单个 tablet 数据量比较大的场景下大幅降低压实的开销。
- 默认值：TRUE

#### lake_service_max_concurrency

- 含义：在存算分离集群中，RPC 请求的最大并发数。当达到此阈值时，新请求会被拒绝。将此项设置为 0 表示对并发不做限制。
- 单位：N/A
- 默认值：0