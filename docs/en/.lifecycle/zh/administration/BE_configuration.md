---
displayed_sidebar: English
---

# BE配置

部分BE配置项是动态参数，您可以在BE节点仍然在线时通过命令设置它们。其余的是静态参数。您只能通过更改相应的配置文件**be.conf**中的BE节点的静态参数来设置BE节点的静态参数，并重新启动BE节点以使更改生效。

## 查看BE配置项

您可以使用以下命令查看BE配置项：

```shell
curl http://<BE_IP>:<BE_HTTP_PORT>/varz
```

## 配置BE动态参数

您可以使用`curl`命令配置BE节点的动态参数。

```Shell
curl -XPOST http://be_host:http_port/api/update_config?<configuration_item>=<value>
```

BE动态参数如下。

#### report_task_interval_seconds

- **默认值：** 10秒
- **描述：** 报告任务状态的时间间隔。任务可以是创建表、删除表、加载数据或更改表结构。

#### report_disk_state_interval_seconds

- **默认值：** 60秒
- **描述：** 报告存储卷状态的时间间隔，包括卷内的数据大小。

#### report_tablet_interval_seconds

- **默认值：** 60秒
- **描述：** 报告所有平板电脑的最新版本的时间间隔。

#### report_workgroup_interval_seconds

- **默认值：** 5秒
- **描述：** 报告所有工作组的最新版本的时间间隔。

#### max_download_speed_kbps

- **默认值：** 50,000KB/s
- **描述：** 每个HTTP请求的最大下载速度。此值会影响BE节点之间数据副本同步的性能。

#### download_low_speed_limit_kbps

- **默认值：** 50KB/s
- **描述：** 每个HTTP请求的下载速度下限。当HTTP请求在配置项`download_low_speed_time`指定的时间跨度内以低于此值的速度持续运行时，该请求将中止。

#### download_low_speed_time

- **默认值：** 300秒
- **描述：** HTTP请求在下载速度低于限制的情况下可以运行的最长时间。当HTTP请求持续运行，其运行速度低于此配置项中指定的时间跨度内的`download_low_speed_limit_kbps`值时，该请求将中止。

#### status_report_interval

- **默认值：** 5秒
- **描述：** 查询上报其配置文件的时间间隔，FE可用于查询统计信息的收集。

#### scanner_thread_pool_thread_num

- **默认值：** 48（线程数）
- **描述：** 存储引擎用于并发存储卷扫描的线程数。所有线程都在线程池中进行管理。

#### thrift_client_retry_interval_ms

- **默认值：** 100毫秒
- **描述：** thrift客户端重试的时间间隔。

#### scanner_thread_pool_queue_size

- **默认值：** 102,400
- **描述：** 存储引擎支持的扫描任务数。

#### scanner_row_num

- **默认值：** 16,384
- **描述：** 每个扫描线程在扫描中返回的最大行数。

#### max_scan_key_num

- **默认值：** 1,024（每个查询的最大扫描键数）
- **描述：** 每个查询分段的最大扫描键数。

#### max_pushdown_conditions_per_column

- **默认值：** 1,024
- **描述：** 每列中允许下推的最大条件数。如果条件数超过此限制，则不会将谓词下推到存储层。

#### exchg_node_buffer_size_bytes

- **默认值：** 10,485,760字节
- **描述：** 每个查询的交换节点接收端的最大缓冲区大小。此配置项为软限制。当数据以过快的速度发送到接收端时，会触发背压。

#### memory_limitation_per_thread_for_schema_change

- **默认值：** 2GB
- **描述：** 每个模式更改任务允许的最大内存大小。

#### update_cache_expire_sec

- **默认值：** 360秒
- **描述：** 更新缓存的过期时间。

#### file_descriptor_cache_clean_interval

- **默认值：** 3,600秒
- **描述：** 清除一段时间内未使用的文件描述符的时间间隔。

#### disk_stat_monitor_interval

- **默认值：** 5秒
- **描述：** 监视磁盘健康状态的时间间隔。

#### unused_rowset_monitor_interval

- **默认值：** 30秒
- **描述：** 清除过期行集的时间间隔。

#### max_percentage_of_error_disk

- **默认值：** 0%
- **描述：** 存储卷中可容忍的最大错误百分比，超过此百分比相应的BE节点将退出。

#### default_num_rows_per_column_file_block

- **默认值：** 1,024
- **描述：** 每个行块中可以存储的最大行数。

#### pending_data_expire_time_sec

- **默认值：** 1,800秒
- **描述：** 存储引擎中待处理数据的过期时间。

#### inc_rowset_expired_sec

- **默认值：** 1,800秒
- **描述：** 传入数据的过期时间。此配置项用于增量克隆。

#### tablet_rowset_stale_sweep_time_sec

- **默认值：** 1,800秒
- **描述：** 在平板电脑中扫描过时行集的时间间隔。

#### snapshot_expire_time_sec

- **默认值：** 172,800秒
- **描述：** 快照文件的过期时间。

#### trash_file_expire_time_sec

- **默认值：** 86,400秒
- **描述：** 清理回收站文件的时间间隔。自v2.5.17、v3.0.9和v3.1.6起，默认值已从259,200更改为86,400。

#### base_compaction_check_interval_seconds

- **默认值：** 60秒
- **描述：** 基本压实的线程轮询时间间隔。

#### min_base_compaction_num_singleton_deltas

- **默认值：** 5
- **描述：** 触发基本压实的最小段数。

#### max_base_compaction_num_singleton_deltas

- **默认值：** 100
- **描述：** 每个基本压实中可以压实的最大段数。

#### base_compaction_interval_seconds_since_last_operation

- **默认值：** 86,400秒
- **描述：** 自上次基本压实以来的时间间隔。此配置项是触发基本压实的条件之一。

#### cumulative_compaction_check_interval_seconds

- **默认值：** 1秒
- **描述：** 累积压实的线程轮询时间间隔。

#### update_compaction_check_interval_seconds

- **默认值：** 60秒
- **描述：** 检查主键表的更新压实的时间间隔。

#### min_compaction_failure_interval_sec

- **默认值：** 120秒
- **描述：** 自上次压实失败以来，可以安排平板压实的最小时间间隔。

#### max_compaction_concurrency

- **默认值：** -1
- **描述：** 压实的最大并发性（基本压实和累积压实）。值-1表示并发性没有限制。

#### periodic_counter_update_period_ms

- **默认值：** 500毫秒
- **描述：** 收集计数器统计信息的时间间隔。

#### load_error_log_reserve_hours

- **默认值：** 48小时
- **描述：** 数据加载日志的保留时间。

#### streaming_load_max_mb

- **默认值：** 10,240MB
- **描述：** 可以流式传输到StarRocks中的文件的最大大小。

#### streaming_load_max_batch_size_mb

- **默认值：** 100MB
- **描述：** 可以流式传输到StarRocks的JSON文件的最大大小。

#### memory_maintenance_sleep_time_s

- **默认值：** 10秒
- **描述：** 触发ColumnPool GC的时间间隔。StarRocks会定期执行GC，并将释放的内存返回给操作系统。

#### write_buffer_size

- **默认值：** 104,857,600字节
- **描述：** 内存中MemTable的缓冲区大小。此配置项是触发刷新的阈值。

#### tablet_stat_cache_update_interval_second

- **默认值：** 300秒
- **描述：** 更新Tablet Stat缓存的时间间隔。

#### result_buffer_cancelled_interval_time

- **默认值：** 300秒
- **描述：** BufferControlBlock释放数据前的等待时间。

#### thrift_rpc_timeout_ms

- **默认值：** 5,000毫秒
- **描述：** thrift RPC的超时。

#### max_consumer_num_per_group

- **默认值：** 3（例程负载的消费者组中的最大消费者数）
- **描述：** Routine Load的消费者组中的最大消费者数。

#### max_memory_sink_batch_count

- **默认值：** 20（最大扫描缓存批次数）
- **描述：** 扫描缓存批次的最大数量。

#### scan_context_gc_interval_min

- **默认值：** 5分钟
- **描述：** 清理扫描上下文的时间间隔。

#### path_gc_check_step

- **默认值：** 1,000（每次连续扫描的最大文件数）
- **描述：** 每次可以连续扫描的最大文件数。

#### path_gc_check_step_interval_ms

- **默认值：** 10毫秒
- **描述：** 文件扫描之间的时间间隔。

#### path_scan_interval_second

- **默认值：** 86,400秒
- **描述：** GC清理过期数据的时间间隔。

#### storage_flood_stage_usage_percent

- **默认值：** 95
- **描述：** 如果BE存储目录的存储使用率（百分比）超过此值，并且剩余存储空间小于`storage_flood_stage_left_capacity_bytes`，则拒绝加载和还原作业。

#### storage_flood_stage_left_capacity_bytes

- **默认值：** 107,374,182,400字节（剩余存储空间阈值，用于拒绝加载和还原作业）
- **描述：** 如果BE存储目录的剩余存储空间小于此值，并且存储使用率（百分比）超过`storage_flood_stage_usage_percent`，则拒绝加载和还原作业。

#### tablet_meta_checkpoint_min_new_rowsets_num

- **默认值：** 10（触发TabletMeta检查点的最小行集数）

- **说明：** 自上次 TabletMeta 检查点以来创建的最小行集数。

#### tablet_meta_checkpoint_min_interval_secs

- **默认值：** 600 秒
- **描述：** TabletMeta 检查点的线程轮询时间间隔。

#### max_runnings_transactions_per_txn_map

- **默认值：** 100（每个分区的最大并发事务数）
- **描述：** 每个分区中可以并发运行的最大事务数。

#### tablet_max_pending_versions

- **默认值：** 1,000（主键平板电脑上挂起的最大版本数）
- **描述：** 主键平板电脑上可容忍的最大挂起版本数。挂起的版本是指已提交但尚未应用的版本。

#### tablet_max_versions

- **默认值：** 1,000（平板电脑上允许的最大版本数）
- **描述：** 平板电脑上允许的最大版本数。如果版本数超过此值，则新的写入请求将失败。

#### max_hdfs_file_handle

- **默认值：** 1,000（HDFS 文件描述符的最大数量）
- **描述：** 可以打开的最大 HDFS 文件描述符数。

#### be_exit_after_disk_write_hang_second

- **默认值：** 60 秒
- **描述：** 磁盘挂起后 BE 等待退出的时间长度。

#### min_cumulative_compaction_failure_interval_sec

- **默认值：** 30 秒
- **描述：** 累积压缩在失败时重试的最短时间间隔。

#### size_tiered_level_num

- **默认值：** 7（大小分层压实的级别数）
- **描述：** Size-tiered Compaction 策略的级别数。每个级别最多保留一个行集。因此，在稳定条件下，行集数最多与此配置项中指定的级别编号一样多。

#### size_tiered_level_multiple

- **默认值：** 5（大小分层压缩中连续级别之间数据大小的倍数）
- **描述：** 大小分层压缩策略中两个连续级别之间的数据大小倍数。

#### size_tiered_min_level_size

- **默认值：** 131,072 字节
- **描述：** 大小分层压缩策略中最小级别的数据大小。小于此值的行集会立即触发数据压缩。

#### storage_page_cache_limit

- **默认值：** 20%
- **描述：** PageCache 大小。它可以指定为大小，例如 `20G`、 `20,480M`或 `20,971,520K` `21,474,836,480B`。也可以将其指定为与内存大小的比率（百分比），例如 `20%`.仅当 `disable_storage_page_cache` 设置为 `false`时才生效。

#### internal_service_async_thread_num

- **默认值：** 10（线程数）
- **描述：** 每个 BE 上允许与 Kafka 交互的线程池大小。目前，负责处理 Routine Load 请求的 FE 依赖于 BE 与 Kafka 进行交互，StarRocks 中的每个 BE 都有自己的线程池，用于与 Kafka 的交互。如果将大量例程加载任务分发给 BE，则 BE 与 Kafka 交互的线程池可能太忙，无法及时处理所有任务。在这种情况下，您可以根据需要调整此参数的值。

#### update_compaction_ratio_threshold

- **默认值：** 0.5
- **描述：** 压缩可以为共享数据集群中的主键表合并的最大数据比例。如果单个平板电脑变得过大，我们建议缩小此值。从 v3.1.5 开始支持此参数。

## 配置 BE 静态参数

只能通过在对应的配置文件 **be.conf** 中更改 BE 的静态参数来设置，并重新启动 BE 以使更改生效。

BE静态参数如下。

#### hdfs_client_enable_hedged_read

- **默认值**：false
- **单位**：不适用
- **描述**：指定是否启用对冲读取功能。从 v3.0 开始支持此参数。

#### hdfs_client_hedged_read_threadpool_size

- **默认值**：128
- **单位**：不适用
- **描述**：指定 HDFS 客户端上对冲读取线程池的大小。线程池大小限制了专用于在 HDFS 客户端中运行对冲读取的线程数。从 v3.0 开始支持此参数。它等同于 HDFS 集群的 hdfs-site.xml 文件中的 dfs.client.hedged.read.threadpool.size 参数。

#### hdfs_client_hedged_read_threshold_millis

- **默认值**：2500
- **单位**：毫秒
- **描述**：指定在启动对冲读取之前要等待的毫秒数。例如，您已将此参数设置为 30。在这种情况下，如果在 30 毫秒内未返回对块的读取，则 HDFS 客户端会立即针对不同的块副本启动新的读取。从 v3.0 开始支持此参数。它等同于 HDFS 集群的 hdfs-site.xml 文件中的 dfs.client.hedged.read.threshold.millis 参数。

#### be_port

- **默认值**：9060
- **单位**：不适用
- **描述**：BE thrift服务器端口，用于接收FE的请求。

#### brpc_port

- **默认值**：8060
- **单位**：不适用
- **描述**：BE bRPC端口，用于查看bRPC的网络统计信息。

#### brpc_num_threads

- **默认值**：-1
- **单位**：不适用
- **描述**：bRPC 的 bthread 数。值 -1 表示与 CPU 线程的数量相同。

#### priority_networks

- **默认值**：空字符串
- **单位**：不适用
- **描述**：如果托管 BE 节点的计算机具有多个 IP 地址，则用于指定 BE 节点的优先级 IP 地址的 CIDR 格式的 IP 地址。

#### heartbeat_service_port

- **默认值**：9050
- **单位**：不适用
- **描述**：BE心跳服务端口，用于接收FE的心跳。

#### starlet_port

- **默认值**：9070
- **单位**：不适用
- **描述**：共享数据集群中 CN（v3.0 中的 BE）的额外代理服务端口。

#### heartbeat_service_thread_count

- **默认值**：1
- **单位**：不适用
- **描述**：BE心跳服务的线程数。

#### create_tablet_worker_count

- **默认值**：3
- **单位**：不适用
- **描述**：用于创建平板电脑的线程数。

#### drop_tablet_worker_count

- **默认值**：3
- **单位**：不适用
- **描述**：用于放置平板电脑的线程数。

#### push_worker_count_normal_priority

- **默认值**：3
- **单位**：不适用
- **描述**：用于处理具有 NORMAL 优先级的加载任务的线程数。

#### push_worker_count_high_priority

- **默认值**：3
- **单位**：不适用
- **描述**：用于处理具有高优先级的加载任务的线程数。

#### transaction_publish_version_worker_count

- **默认值**：0
- **单位**：不适用
- **描述**：用于发布版本的最大线程数。当该值设置为小于或等于 0 时，系统使用 CPU 核心数的一半作为该值，以避免在导入并发较高但只使用固定线程数时线程资源不足。从 v2.5 开始，默认值已从 8 更改为 0。

#### clear_transaction_task_worker_count

- **默认值**：1
- **单位**：不适用
- **描述**：用于清算事务的线程数。

#### alter_tablet_worker_count

- **默认值**：3
- **单位**：不适用
- **描述**：用于架构更改的线程数。

#### clone_worker_count

- **默认值**：3
- **单位**：不适用
- **描述**：用于克隆的线程数。

#### storage_medium_migrate_count

- **默认值**：1
- **单位**：不适用
- **描述**：用于存储介质迁移（从 SATA 到 SSD）的线程数。

#### check_consistency_worker_count

- **默认值**：1
- **单位**：不适用
- **描述**：用于检查平板电脑一致性的线程数。

#### sys_log_dir

- **默认值**： `${STARROCKS_HOME}/log`
- **单位**：不适用
- **描述**：存储系统日志（包括 INFO、WARNING、ERROR 和 FATAL）的目录。

#### user_function_dir

- **默认值**： `${STARROCKS_HOME}/lib/udf`
- **单位**：不适用
- **描述**：用于存储用户定义函数 （UDF） 的目录。

#### small_file_dir

- **默认值**： `${STARROCKS_HOME}/lib/small_file`
- **单位**：不适用
- **描述**：用于存储文件管理器下载的文件的目录。

#### sys_log_level

- **默认值**：INFO
- **单位**：不适用
- **描述**：系统日志条目分类的严重性级别。取值范围：INFO、WARN、ERROR 和 FATAL。

#### sys_log_roll_mode

- **默认值**：SIZE-MB-1024
- **单位**：不适用
- **描述**：将系统日志分段为日志卷的模式。有效值包括 `TIME-DAY`、 `TIME-HOUR`和 `SIZE-MB-`size。默认值表示日志被分段为卷，每个卷为 1 GB。

#### sys_log_roll_num

- **默认值**：10
- **单位**：不适用
- **描述**：要保留的日志卷数。

#### sys_log_verbose_modules

- **默认值**：空字符串
- **单位**：不适用
- **描述**：要打印的日志的模块。例如，如果将该配置项设置为 OLAP，StarRocks 只会打印 OLAP 模块的日志。有效值为 BE 中的命名空间，包括 starrocks、starrocks：:d ebug、starrocks：：fs、starrocks：：io、starrocks：：lake、starrocks：:p ipeline、starrocks：：query_cache、starrocks：：stream 和 starrocks：：workgroup。

#### sys_log_verbose_level

- **默认值**：10
- **单位**：不适用
- **描述**：要打印的日志的级别。该配置项用于控制代码中VLOG发起的日志输出。

#### log_buffer_level

- **默认值**：空字符串
- **单位**：不适用
- **描述**：刷新日志的策略。默认值表示日志在内存中缓冲。有效值为 -1 和 0。-1 表示日志未在内存中缓冲。

#### num_threads_per_core

- **默认**：3
- **单位**：N/A
- **描述**：每个 CPU 核心上启动的线程数。

#### compress_rowbatches

- **默认**：TRUE
- **单位**：N/A
- **描述**：一个布尔值，用于控制是否在 BE 之间压缩 RPC 中的行批处理。TRUE 表示压缩行批处理，FALSE 表示不压缩行批处理。

#### serialize_batch

- **默认**：FALSE
- **单位**：N/A
- **描述**：一个布尔值，用于控制是否在 BE 之间序列化 RPC 中的行批处理。TRUE 表示序列化行批处理，FALSE 表示不序列化行批处理。

#### storage_root_path

- **默认**： `${STARROCKS_HOME}/storage`
- **单位**：N/A
- **描述**：存储卷的目录和介质。
  - 多个卷之间用分号 (`;`) 分隔。
  - 如果存储介质是 SSD，请在目录末尾添加 `medium:ssd`。
  - 如果存储介质为 HDD，请在目录末尾添加 `medium:hdd`。

#### max_length_for_bitmap_function

- **默认**：1000000
- **单位**：字节
- **描述**：位图函数输入值的最大长度。

#### max_length_for_to_base64

- **默认**：200000
- **单位**：字节
- **描述**：to_base64() 函数输入值的最大长度。

#### max_tablet_num_per_shard

- **默认**：1024
- **单位**：N/A
- **描述**：每个分片中的最大平板电脑数量。该配置项用于限制每个存储目录下的平板电脑子目录的数量。

#### max_garbage_sweep_interval

- **默认**：3600
- **单位**：秒
- **描述**：存储卷上垃圾回收的最大时间间隔。

#### min_garbage_sweep_interval

- **默认**：180
- **单位**：秒
- **描述**：存储卷上垃圾回收的最小时间间隔。

#### file_descriptor_cache_capacity

- **默认**：16384
- **单位**：N/A
- **描述**：可以缓存的文件描述符数。

#### min_file_descriptor_number

- **默认**：60000
- **单位**：N/A
- **描述**：BE 进程中文件描述符的最小数量。

#### index_stream_cache_capacity

- **默认**：10737418240
- **单位**：字节
- **描述**：BloomFilter、Min、Max统计信息的缓存容量。

#### disable_storage_page_cache

- **默认**：FALSE
- **单位**：N/A
- **描述**：一个布尔值，用于控制是否禁用 PageCache。
  - 开启 PageCache 后，StarRocks会缓存最近扫描的数据。
  - 当频繁重复类似的查询时，PageCache可以显著提高查询性能。
  - TRUE表示禁用 PageCache。
  - 自StarRocks v2.4起，此项的默认值已从TRUE更改为FALSE。

#### base_compaction_num_threads_per_disk

- **默认**：1
- **单位**：N/A
- **描述**：每个存储卷上用于基本压缩的线程数。

#### base_cumulative_delta_ratio

- **默认**：0.3
- **单位**：N/A
- **描述**：累积文件大小与基本文件大小的比率。达到此值的比率是触发基础压实的条件之一。

#### compaction_trace_threshold

- **默认**：60
- **单位**：秒
- **描述**：每个压缩的时间阈值。如果压缩时间超过时间阈值，StarRocks会打印相应的Trace。

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
- **描述**：小规模加载生成的文件的预留时间。

#### number_tablet_writer_threads

- **默认**：16
- **单位**：N/A
- **描述**：用于流加载的线程数。

#### streaming_load_rpc_max_alive_time_sec

- **默认**：1200
- **单位**：秒
- **描述**：流加载的RPC超时。

#### fragment_pool_thread_num_min

- **默认**：64
- **单位**：N/A
- **描述**：用于查询的最小线程数。

#### fragment_pool_thread_num_max

- **默认**：4096
- **单位**：N/A
- **描述**：用于查询的最大线程数。

#### fragment_pool_queue_size

- **默认**：2048
- **单位**：N/A
- **描述**：每个BE节点上可以处理的查询数上限。

#### enable_token_check

- **默认**：TRUE
- **单位**：N/A
- **描述**：一个布尔值，用于控制是否启用令牌检查。TRUE表示启用令牌检查，FALSE表示禁用令牌检查。

#### enable_prefetch

- **默认**：TRUE
- **单位**：N/A
- **描述**：一个布尔值，用于控制是否启用查询的预提取。TRUE表示启用预取，FALSE表示禁用预取。

#### load_process_max_memory_limit_bytes

- **默认**：107374182400
- **单位**：字节
- **描述**：BE节点上所有加载进程可以占用的内存资源的最大大小限制。

#### load_process_max_memory_limit_percent

- **默认**：30
- **单位**：%
- **描述**：BE节点上所有加载进程可以占用的内存资源的最大百分比限制。

#### sync_tablet_meta

- **默认**：FALSE
- **单位**：N/A
- **描述**：一个布尔值，用于控制是否启用平板电脑元数据的同步。TRUE表示启用同步，FALSE表示禁用同步。

#### routine_load_thread_pool_size

- **默认**：10
- **单位**：N/A
- **描述**：每个BE上例程加载的线程池大小。从v3.1.0开始，该参数已弃用。每个BE上例程加载的线程池大小现在由FE动态参数max_routine_load_task_num_per_be控制。

#### brpc_max_body_size

- **默认**：2147483648
- **单位**：字节
- **描述**：bRPC的最大正文大小。

#### tablet_map_shard_size

- **默认**：32
- **单位**：N/A
- **描述**：平板电脑地图分片大小。该值必须是2的幂。

#### enable_bitmap_union_disk_format_with_set

- **默认**：FALSE
- **单位**：N/A
- **描述**：一个布尔值，用于控制是否开启BITMAP类型的新存储格式，可以提高bitmap_union的性能。TRUE表示启用新的存储格式，FALSE表示禁用它。

#### mem_limit

- **默认**：90%
- **单位**：N/A
- **描述**：BE进程内存上限。您可以将其设置为百分比（“80%”）或物理限制（“100GB”）。

#### flush_thread_num_per_store

- **默认**：2
- **单位**：N/A
- **描述**：每个存储中用于刷新MemTable的线程数。

#### datacache_enable

- **默认**：false
- **单位**：N/A
- **描述**：是否开启数据缓存。TRUE表示已启用数据缓存，FALSE表示已禁用数据缓存。

#### datacache_disk_path

- **默认**：N/A
- **单位**：N/A
- **描述**：磁盘的路径。建议为此参数配置的路径数与BE机器上的磁盘数相同。多条路径需要用分号 (`;`) 分隔。

#### datacache_meta_path

- **默认**：N/A
- **单位**：N/A
- **描述**：区块元数据的存储路径。您可以自定义存储路径。建议将元数据存储在$STARROCKS_HOME路径下。

#### datacache_mem_size

- **默认**：10%
- **单位**：N/A
- **描述**：内存中可以缓存的最大数据量。您可以将其设置为百分比（例如，`10%`）或物理限制（例如，`10G`、`21474836480`）。缺省值为`10%`。建议您至少将该参数的值设置为10 GB。

#### datacache_disk_size

- **默认**：0
- **单位**：N/A
- **描述**：单个磁盘上可以缓存的最大数据量。您可以将其设置为百分比（例如，`80%`）或物理限制（例如，`2T`、`500G`）。例如，如果为该参数配置了两个磁盘路径`datacache_disk_path`，并将该参数的值设置为`datacache_disk_size` `21474836480`（20 GB），则这两个磁盘上最多可以缓存40 GB的数据。默认值为`0`，表示仅使用内存来缓存数据。单位：字节。

#### jdbc_connection_pool_size

- **默认**：8
- **单位**：N/A
- **描述**：JDBC连接池大小。在每个BE节点上，使用相同jdbc_url访问外部表的查询共享相同的连接池。

#### jdbc_minimum_idle_connections

- **默认**：1
- **单位**：N/A
- **描述**：JDBC连接池中最小空闲连接数。

#### jdbc_connection_idle_timeout_ms

- **默认**：600000
- **单位**：N/A
- **描述**：JDBC连接池中的空闲连接过期的时间长度。如果JDBC连接池中的连接空闲时间超过此值，那么连接池将关闭超出配置项jdbc_minimum_idle_connections中指定数量的空闲连接。

#### query_cache_capacity

- **默认**：536870912
- **单位**：N/A
- **描述**：BE 中查询缓存的大小。单位：字节。默认大小为 512 MB。大小不能小于 4 MB。如果 BE 的内存容量不足以满足您预期的查询缓存大小，则可以增加 BE 的内存容量。

#### enable_event_based_compaction_framework

- **默认值**：TRUE
- **单位**：N/A
- **描述**：是否启用基于事件的压缩框架。TRUE 表示已启用基于事件的压缩框架，FALSE 表示已禁用。启用基于事件的压缩框架可以大大减少在存在许多平板或单个平板数据量较大的情况下压缩的开销。

#### enable_size_tiered_compaction_strategy

- **默认值**：TRUE
- **单位**：N/A
- **描述**：是否启用大小分层压缩策略。TRUE 表示已启用大小分层压缩策略，FALSE 表示已禁用该策略。

#### lake_service_max_concurrency

- **默认值**：0
- **单位**：N/A
- **描述**：共享数据集群中 RPC 请求的最大并发数。当达到此阈值时，传入请求将被拒绝。当此项设置为 0 时，对并发没有限制。