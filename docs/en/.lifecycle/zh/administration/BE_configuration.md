---
displayed_sidebar: English
---

#### max_runnings_transactions_per_txn_map

- **默认值:** 100（每个分区的并发事务的最大数量）
- **描述：** 每个分区中可以同时运行的最大事务数。

#### tablet_max_pending_versions

- **默认值:** 1,000（主键平板电脑上可容忍的最大挂起版本数）
- **描述：** 主键平板电脑上可容忍的最大挂起版本数。挂起版本是指已提交但尚未应用的版本。

#### tablet_max_versions

- **默认值:** 1,000（平板电脑上允许的最大版本数）
- **描述：** 平板电脑上允许的最大版本数。如果版本数超过此值，新的写入请求将失败。

#### max_hdfs_file_handle

- **默认值:** 1,000（HDFS文件描述符的最大数量）
- **描述：** 可以打开的HDFS文件描述符的最大数量。

#### be_exit_after_disk_write_hang_second

- **默认值:** 60 秒
- **描述：** BE在磁盘挂起后等待退出的时间长度。

#### min_cumulative_compaction_failure_interval_sec

- **默认值:** 30 秒
- **描述：** 累积压缩失败后的最小时间间隔。

#### size_tiered_level_num

- **默认值:** 7（大小分层压缩的级别数）
- **描述：** 大小分层压缩策略的级别数。每个级别最多保留一个行集。因此，在稳定的情况下，行集的数量最多与此配置项中指定的级别数相同。

#### size_tiered_level_multiple

- **默认值:** 5（大小分层压缩中相邻级别之间的数据大小倍数）
- **描述：** 大小分层压缩策略中相邻级别之间的数据大小倍数。

#### size_tiered_min_level_size

- **默认值:** 131,072 字节
- **描述：** 大小分层压缩策略中最小级别的数据大小。小于此值的行集将立即触发数据压缩。

#### storage_page_cache_limit

- **默认值:** 20%
- **描述：** PageCache的大小。可以指定为大小，例如`20G`、`20,480M`、`20,971,520K`或`21,474,836,480B`。也可以指定为与内存大小的比率（百分比），例如`20%`。仅在`disable_storage_page_cache`设置为`false`时生效。

#### internal_service_async_thread_num

- **默认值:** 10（线程数）
- **描述：** 与Kafka交互时在每个BE上允许的线程池大小。目前，负责处理例程加载请求的FE依赖于BE与Kafka进行交互，而StarRocks中的每个BE都有自己的用于与Kafka交互的线程池。如果将大量的例程加载任务分布到一个BE上，BE与Kafka交互的线程池可能会过于繁忙，无法及时处理所有任务。在这种情况下，您可以调整此参数的值以适应您的需求。

#### update_compaction_ratio_threshold

- **默认值:** 0.5
- **描述：** 共享数据集群中主键表压缩可以合并的最大数据比例。如果单个平板电脑过大，建议缩小此值。此参数从v3.1.5开始支持。

## 配置BE静态参数

您只能通过更改相应的配置文件**be.conf**中的静态参数来设置BE的静态参数，并重新启动BE以使更改生效。

BE静态参数如下。

#### hdfs_client_enable_hedged_read

- **默认值：** false
- **单位：** N/A
- **描述：** 指定是否启用hedged read功能。此参数从v3.0开始支持。

#### hdfs_client_hedged_read_threadpool_size

- **默认值：** 128
- **单位：** N/A
- **描述：** 指定HDFS客户端上hedged read线程池的大小。线程池大小限制了在HDFS客户端中运行hedged read的线程数。此参数从v3.0开始支持。它相当于HDFS集群的hdfs-site.xml文件中的dfs.client.hedged.read.threadpool.size参数。

#### hdfs_client_hedged_read_threshold_millis

- **默认值：** 2500
- **单位：** 毫秒
- **描述：** 指定在启动hedged read之前等待的毫秒数。例如，您将此参数设置为30。在这种情况下，如果从一个块读取的时间超过30毫秒，HDFS客户端将立即启动对另一个块副本的新读取。此参数从v3.0开始支持。它相当于HDFS集群的hdfs-site.xml文件中的dfs.client.hedged.read.threshold.millis参数。

#### be_port

- **默认值：** 9060
- **单位：** N/A
- **描述：** BE thrift服务器端口，用于接收FE的请求。

#### brpc_port

- **默认值：** 8060
- **单位：** N/A
- **描述：** BE bRPC端口，用于查看bRPC的网络统计信息。

#### brpc_num_threads

- **默认值：** -1
- **单位：** N/A
- **描述：** bRPC的bthread数。值-1表示与CPU线程数相同的数量。

#### priority_networks

- **默认值：** 空字符串
- **单位：** N/A
- **描述：** 如果托管BE节点的机器具有多个IP地址，则用于指定BE节点的优先级IP地址的CIDR格式的IP地址。

#### heartbeat_service_port

- **默认值：** 9050
- **单位：** N/A
- **描述：** BE心跳服务端口，用于接收FE的心跳。

#### starlet_port

- **默认值：** 9070
- **单位：** N/A
- **描述：** 共享数据集群中CN（v3.0中的BE）的额外代理服务端口。

#### heartbeat_service_thread_count

- **默认值：** 1
- **单位：** N/A
- **描述：** BE心跳服务的线程数。

#### create_tablet_worker_count

- **默认值：** 3
- **单位：** N/A
- **描述：** 用于创建平板电脑的线程数。

#### drop_tablet_worker_count

- **默认值：** 3
- **单位：** N/A
- **描述：** 用于删除平板电脑的线程数。

#### push_worker_count_normal_priority

- **默认值：** 3
- **单位：** N/A
- **描述：** 用于处理具有NORMAL优先级的加载任务的线程数。

#### push_worker_count_high_priority

- **默认值：** 3
- **单位：** N/A
- **描述：** 用于处理具有HIGH优先级的加载任务的线程数。

#### transaction_publish_version_worker_count

- **默认值：** 0
- **单位：** N/A
- **描述：** 用于发布版本的最大线程数。当此值设置为小于或等于0时，系统使用CPU核心数的一半作为值，以避免在并发性高但仅使用固定数量的线程时出现线程资源不足的情况。从v2.5开始，默认值已从8更改为0。

#### clear_transaction_task_worker_count

- **默认值：** 1
- **单位：** N/A
- **描述：** 用于清除事务的线程数。

#### alter_tablet_worker_count

- **默认值：** 3
- **单位：** N/A
- **描述：** 用于模式更改的线程数。

#### clone_worker_count

- **默认值：** 3
- **单位：** N/A
- **描述：** 用于克隆的线程数。

#### storage_medium_migrate_count

- **默认值：** 1
- **单位：** N/A
- **描述：** 用于存储介质迁移（从SATA到SSD）的线程数。

#### check_consistency_worker_count

- **默认值：** 1
- **单位：** N/A
- **描述：** 用于检查平板电脑一致性的线程数。

#### sys_log_dir

- **默认值：** `${STARROCKS_HOME}/log`
- **单位：** N/A
- **描述：** 存储系统日志（包括INFO、WARNING、ERROR和FATAL）的目录。

#### user_function_dir

- **默认值：** `${STARROCKS_HOME}/lib/udf`
- **单位：** N/A
- **描述：** 用于存储用户定义函数（UDF）的目录。

#### small_file_dir

- **默认值：** `${STARROCKS_HOME}/lib/small_file`
- **单位：** N/A
- **描述：** 用于存储文件管理器下载的文件的目录。

#### sys_log_level

- **默认值：** INFO
- **单位：** N/A
- **描述：** 将系统日志条目分类为的严重级别。有效值：INFO、WARN、ERROR和FATAL。

#### sys_log_roll_mode

- **默认值：** SIZE-MB-1024
- **单位：** N/A
- **描述：** 系统日志被分割成日志卷的模式。有效值包括`TIME-DAY`、`TIME-HOUR`和`SIZE-MB-`大小。默认值表示日志被分割成每个1GB的卷。

#### sys_log_roll_num

- **默认值：** 10
- **单位：** N/A
- **描述：** 保留的日志卷数。

#### sys_log_verbose_modules

- **默认值：** 空字符串
- **单位：** N/A
- **描述：** 要打印的日志模块。例如，如果将此配置项设置为OLAP，则StarRocks仅打印OLAP模块的日志。有效值是BE中的命名空间，包括starrocks、starrocks::debug、starrocks::fs、starrocks::io、starrocks::lake、starrocks::pipeline、starrocks::query_cache、starrocks::stream和starrocks::workgroup。

#### sys_log_verbose_level

- **默认值：** 10
- **单位：** N/A
- **描述：** 要打印的日志级别。此配置项用于控制代码中使用VLOG发起的日志的输出。

#### log_buffer_level

- **默认值：** 空字符串
- **单位：** N/A
- **描述：** 刷新日志的策略。默认值表示日志在内存中缓冲。有效值为-1和0。-1表示日志不会在内存中缓冲。

#### num_threads_per_core

- **默认值：** 3
- **单位：** N/A
- **描述：** 在每个CPU核心上启动的线程数。

#### compress_rowbatches

- **默认值：** TRUE
- **单位：** N/A
- **描述：** 一个布尔值，用于控制是否在BE之间的RPC中压缩行批次。TRUE表示压缩行批次，FALSE表示不压缩。

#### serialize_batch

- **默认值：** FALSE
- **单位：** N/A
- **描述：** 一个布尔值，用于控制是否在BE之间的RPC中序列化行批次。TRUE表示序列化行批次，FALSE表示不序列化。

#### storage_root_path

- **默认值：** `${STARROCKS_HOME}/storage`
- **单位：** N/A
- **描述：** 存储卷的目录和介质。
  - 多个卷用分号(`;`)分隔。
  - 如果存储介质是SSD，在目录末尾添加`medium:ssd`。
  - 如果存储介质是HDD，在目录末尾添加`medium:hdd`。

#### max_length_for_bitmap_function

- **默认值：** 1000000
- **单位：** 字节
- **描述：** 位图函数输入值的最大长度。

#### max_length_for_to_base64

- **默认值：** 200000
- **单位：** 字节
- **描述：** to_base64()函数输入值的最大长度。

#### max_tablet_num_per_shard

- **默认值：** 1024
- **单位：** N/A
- **描述：** 每个分片中的最大平板电脑数量。此配置项用于限制每个存储目录下的平板电脑子目录数量。

#### max_garbage_sweep_interval

- **默认值：** 3600
- **单位：** 秒
- **描述：** 存储卷上垃圾收集的最大时间间隔。

#### min_garbage_sweep_interval

- **默认值：** 180
- **单位：** 秒
- **描述：** 存储卷上垃圾收集的最小时间间隔。

#### file_descriptor_cache_capacity

- **默认值：** 16384
- **单位：** N/A
- **描述：** 可以缓存的文件描述符数量。

#### min_file_descriptor_number

- **默认值：** 60000
- **单位：** N/A
- **描述：** BE进程中的最小文件描述符数。

#### index_stream_cache_capacity

- **默认值：** 10737418240
- **单位：** 字节
- **描述：** BloomFilter、Min和Max的统计信息的缓存容量。

#### disable_storage_page_cache

- **默认值：** FALSE
- **单位：** N/A
- **描述：** 一个布尔值，用于控制是否禁用PageCache。
  - 当启用PageCache时，StarRocks会缓存最近扫描的数据。
  - 当频繁重复执行类似的查询时，PageCache可以显著提高查询性能。
  - TRUE表示禁用PageCache。
  - 从StarRocks v2.4开始，此项的默认值已从TRUE更改为FALSE。

#### base_compaction_num_threads_per_disk

- **默认值：** 1
- **单位：** N/A
- **描述：** 每个存储卷上用于基本压缩的线程数。

#### base_cumulative_delta_ratio

- **默认值：** 0.3
- **单位：** N/A
- **描述：** 累积文件大小与基本文件大小的比例。达到此值的比例是触发基本压缩的条件之一。

#### compaction_trace_threshold

- **默认值：** 60
- **单位：** 秒
- **描述：** 每个压缩的时间阈值。如果压缩时间超过时间阈值，StarRocks将打印相应的跟踪信息。

#### be_http_port

- **默认值：** 8040
- **单位：** N/A
- **描述：** HTTP服务器端口。

#### be_http_num_workers

- **默认值：** 48
- **单位：** N/A
- **描述：** HTTP服务器使用的线程数。

#### load_data_reserve_hours

- **默认值：** 4
- **单位：** 小时
- **描述：** 小规模加载生成的文件的保留时间。

#### number_tablet_writer_threads

- **默认值：** 16
- **单位：** N/A
- **描述：** 用于流式加载的线程数。

#### streaming_load_rpc_max_alive_time_sec

- **默认值：** 1200
- **单位：** 秒
- **描述：** 流式加载的RPC超时时间。

#### fragment_pool_thread_num_min

- **默认值：** 64
- **单位：** N/A
- **描述：** 用于查询的最小线程数。

#### fragment_pool_thread_num_max

- **默认值：** 4096
- **单位：** N/A
- **描述：** 用于查询的最大线程数。

#### fragment_pool_queue_size

- **默认值：** 2048
- **单位：** N/A
- **描述：** 每个BE节点上可以处理的查询数量的上限。

#### enable_token_check

- **默认值：** TRUE
- **单位：** N/A
- **描述：** 一个布尔值，用于控制是否启用令牌检查。TRUE表示启用令牌检查，FALSE表示禁用。

#### enable_prefetch

- **默认值：** TRUE
- **单位：** N/A
- **描述：** 一个布尔值，用于控制是否启用查询的预取。TRUE表示启用预取，FALSE表示禁用。

#### load_process_max_memory_limit_bytes

- **默认值：** 107374182400
- **单位：** 字节
- **描述：** 在BE节点上所有加载进程占用的内存资源的最大大小限制。

#### load_process_max_memory_limit_percent

- **默认值：** 30
- **单位：** %
- **描述：** 在BE节点上所有加载进程占用的内存资源的最大百分比限制。

#### sync_tablet_meta

- **默认值：** FALSE
- **单位：** N/A
- **描述：** 一个布尔值，用于控制是否启用平板电脑元数据的同步。TRUE表示启用同步，FALSE表示禁用。

#### routine_load_thread_pool_size

- **默认值：** 10
- **单位：** N/A
- **描述：** 每个BE上例程加载的线程池大小。从v3.1.0开始，此参数已弃用。现在，每个BE上例程加载的线程池大小由FE动态参数max_routine_load_task_num_per_be控制。

#### brpc_max_body_size

- **默认值：** 2147483648
- **单位：** 字节
- **描述：** bRPC的最大body大小。

#### tablet_map_shard_size

- **默认值：** 32
- **单位：** N/A
- **描述：** 平板电脑映射分片大小。该值必须是2的幂。

#### enable_bitmap_union_disk_format_with_set

- **默认值：** FALSE
- **单位：** N/A
- **描述：** 一个布尔值，用于控制是否启用BITMAP类型的新存储格式，该格式可以改善bitmap_union的性能。TRUE表示启用新的存储格式，FALSE表示禁用。

#### mem_limit

- **默认值：** 90%
- **单位：** N/A
- **描述：** BE进程内存上限。可以将其设置为百分比（“80%”）或物理限制（“100GB”）。

#### flush_thread_num_per_store

- **默认值：** 2
- **单位：** N/A
- **描述：** 每个存储中用于刷新MemTable的线程数。

#### datacache_enable

- **默认值：** false
- **单位：** N/A
- **描述：** 是否启用数据缓存。TRUE表示启用数据缓存，FALSE表示禁用数据缓存。

#### datacache_disk_path

- **默认值：** N/A
- **单位：** N/A
- **描述：** 磁盘的路径。我们建议您为此参数配置的路径数与BE机器上的磁盘数相同。多个路径需要用分号（;）分隔。

#### datacache_meta_path

- **默认值：** N/A
- **单位：** N/A
- **描述：** 块元数据的存储路径。您可以自定义存储路径。我们建议您将元数据存储在$STARROCKS_HOME路径下。

#### datacache_mem_size

- **默认值：** 10%
- **单位：** N/A
- **描述：** 可以在内存中缓存的数据的最大量。您可以将其设置为百分比（例如`10%`）或物理限制（例如`10G`、`21474836480`）。默认值为`10%`。我们建议您将此参数的值设置为至少10GB。

#### datacache_disk_size

- **默认值：** 0
- **单位：** N/A
- **描述：** 单个磁盘上可以缓存的数据的最大量。您可以将其设置为百分比（例如`80%`）或物理限制（例如`2T`、`500G`）。例如，如果您为`datacache_disk_path`参数配置了两个磁盘路径，并将`datacache_disk_size`参数的值设置为`21474836480`（20GB），则这两个磁盘上最多可以缓存40GB的数据。默认值为`0`，表示仅使用内存缓存数据。单位：字节。

#### jdbc_connection_pool_size

- **默认值：** 8
- **单位：** N/A
- **描述：** JDBC连接池大小。在每个BE节点上，访问具有相同jdbc_url的外部表的查询共享相同的连接池。

#### jdbc_minimum_idle_connections

- **默认值：** 1
- **单位：** N/A
- **描述：** JDBC连接池中的最小空闲连接数。

#### jdbc_connection_idle_timeout_ms

- **默认值：** 600000
- **单位：** N/A
- **描述：** 空闲连接在JDBC连接池中过期的时间长度。如果JDBC连接池中的连接空闲时间超过此值，连接池会关闭超出配置项jdbc_minimum_idle_connections数量的空闲连接。

#### query_cache_capacity

- **默认值：** 536870912
- **单位：** N/A
- **描述：** BE中查询缓存的大小。单位：字节。默认大小为512MB。大小不能小于4MB。如果BE的内存容量不足以满足您期望的查询缓存大小，可以增加BE的内存容量。

#### enable_event_based_compaction_framework

- **默认值：** TRUE
- **单位：** N/A
- **描述：** 是否启用基于事件的压缩框架。TRUE表示启用基于事件的压缩框架，FALSE表示禁用。启用基于事件的压缩框架可以大大减少压缩的开销，适用于平板电脑较多或单个平板电脑数据量较大的场景。

#### enable_size_tiered_compaction_strategy

- **默认值：** TRUE
- **单位：** N/A
- **描述：** 是否启用大小分层压缩策略。TRUE表示启用大小分层压缩策略，FALSE表示禁用。

#### lake_service_max_concurrency

- **默认值：** 0
- **单位：** N/A
- **描述：** 共享数据集群中RPC请求的最大并发数。当达到此阈值时，将拒绝传入请求。当此项设置为0时，不限制并发数。
```
#### max_runnings_transactions_per_txn_map

- **默认值:** 100（每个分区的最大并发事务数）
- **描述:** 每个分区中可以并发运行的最大事务数。

#### tablet_max_pending_versions

- **默认值:** 1,000（主键平板电脑上待处理版本的最大数量）
- **描述:** 主键平板电脑上可容忍的最大待处理版本数。待处理版本是指已提交但尚未应用的版本。

#### tablet_max_versions

- **默认值:** 1,000（平板电脑上允许的最大版本数）
- **描述:** 平板电脑上允许的最大版本数。如果版本数超过此值，新的写入请求将失败。

#### max_hdfs_file_handle

- **默认值:** 1,000（最大HDFS文件描述符数量）
- **描述:** 可以打开的HDFS文件描述符的最大数量。

#### be_exit_after_disk_write_hang_second

- **默认值:** 60秒
- **描述:** 磁盘挂起后BE等待退出的时间长度。

#### min_cumulative_compaction_failure_interval_sec

- **默认值:** 30秒
- **描述:** 累积压缩失败时重试的最小时间间隔。

#### size_tiered_level_num

- **默认值:** 7（大小分层压缩的级别数）
- **描述:** Size-tiered Compaction策略的级别数。每个级别最多保留一个rowset。因此，在稳定的情况下，rowset的数量最多与该配置项中指定的级别数相同。

#### size_tiered_level_multiple

- **默认值:** 5（大小分层压缩中连续级别之间的数据大小的倍数）
- **描述:** Size-tiered Compaction策略中两个连续级别之间数据大小的倍数。

#### size_tiered_min_level_size

- **默认值:** 131,072字节
- **描述:** Size-tiered Compaction策略中最小级别的数据大小。小于该值的rowset立即触发数据压缩。

#### storage_page_cache_limit

- **默认值:** 20%
- **描述:** PageCache大小。它可以指定为大小，例如`20G`、`20,480M`、`20,971,520K`或`21,474,836,480B`。也可以指定为与内存大小的比率（百分比），例如`20%`。仅当`disable_storage_page_cache`设置为`false`时才生效。

#### internal_service_async_thread_num

- **默认值:** 10（线程数）
- **描述:** 每个BE上允许与Kafka交互的线程池大小。目前，负责处理Routine Load请求的FE依赖BE与Kafka交互，StarRocks中的每个BE都有自己的线程池用于与Kafka交互。如果大量的Routine Load任务分发到BE上，BE与Kafka交互的线程池可能会过于繁忙而无法及时处理所有任务。在这种情况下，您可以调整该参数的值以满足您的需要。

#### update_compaction_ratio_threshold

- **默认值:** 0.5
- **描述:** 共享数据集群中主键表压缩可以合并的最大数据比例。如果单个平板电脑变得过大，我们建议缩小此值。从v3.1.5开始支持此参数。

## 配置BE静态参数

您只能通过在相应的配置文件**be.conf**中更改BE的静态参数来设置BE的静态参数，并重新启动BE以使更改生效。

BE静态参数如下。

#### hdfs_client_enable_hedged_read

- **默认值:** false
- **单位:** 不适用
- **描述:** 指定是否启用对冲读功能。从v3.0开始支持此参数。

#### hdfs_client_hedged_read_threadpool_size

- **默认值:** 128
- **单位:** 不适用
- **描述:** 指定HDFS客户端上对冲读取线程池的大小。线程池大小限制了HDFS客户端中专用于运行对冲读取的线程数量。从v3.0开始支持此参数。它相当于HDFS集群的hdfs-site.xml文件中的dfs.client.hedged.read.threadpool.size参数。

#### hdfs_client_hedged_read_threshold_millis

- **默认值:** 2500
- **单位:** 毫秒
- **描述:** 指定在启动对冲读取之前等待的毫秒数。例如，您已将此参数设置为30。在这种情况下，如果从块中的读取在30毫秒内未返回，您的HDFS客户端立即启动针对不同块副本的新读取。此参数从v3.0版本开始支持。它相当于您的HDFS集群中hdfs-site.xml文件中的dfs.client.hedged.read.threshold.millis参数。

#### be_port

- **默认值:** 9060
- **单位:** 不适用
- **描述:** BE thrift服务器端口，用于接收来自FE的请求。

#### brpc_port

- **默认值:** 8060
- **单位:** 不适用
- **描述:** BE bRPC端口，用于查看bRPC的网络统计信息。

#### brpc_num_threads

- **默认值:** -1
- **单位:** 不适用
- **描述:** bRPC的bthread数量。值-1表示与CPU线程数相同。

#### priority_networks

- **默认值:** 空字符串
- **单位:** 不适用
- **描述:** CIDR格式的IP地址，用于指定BE节点的优先级IP地址（如果托管BE节点的计算机有多个IP地址）。

#### heartbeat_service_port

- **默认值:** 9050
- **单位:** 不适用
- **描述:** BE心跳服务端口，用于接收FE的心跳。

#### starlet_port

- **默认值:** 9070
- **单位:** 不适用
- **描述:** 共享数据集群中CN（v3.0中为BE）的额外代理服务端口。

#### heartbeat_service_thread_count

- **默认值:** 1
- **单位:** 不适用
- **描述:** BE心跳服务的线程数。

#### create_tablet_worker_count

- **默认值:** 3
- **单位:** 不适用
- **描述:** 用于创建tablet的线程数。

#### drop_tablet_worker_count

- **默认值:** 3
- **单位:** 不适用
- **描述:** 用于删除tablet的线程数。

#### push_worker_count_normal_priority

- **默认值:** 3
- **单位:** 不适用
- **描述:** 用于处理具有NORMAL优先级的加载任务的线程数。

#### push_worker_count_high_priority

- **默认值:** 3
- **单位:** 不适用
- **描述:** 用于处理具有HIGH优先级的加载任务的线程数。

#### transaction_publish_version_worker_count

- **默认值:** 0
- **单位:** 不适用
- **描述:** 发布版本所用的最大线程数。当该值设置为小于或等于0时，系统使用CPU核心数的一半作为该值，以避免导入并发量较高但仅使用固定线程数时线程资源不足。从v2.5开始，默认值已从8更改为0。

#### clear_transaction_task_worker_count

- **默认值:** 1
- **单位:** 不适用
- **描述:** 用于清理事务的线程数。

#### alter_tablet_worker_count

- **默认值:** 3
- **单位:** 不适用
- **描述:** 用于schema change的线程数。

#### clone_worker_count

- **默认值:** 3
- **单位:** 不适用
- **描述:** 用于克隆的线程数。

#### storage_medium_migrate_count

- **默认值:** 1
- **单位:** 不适用
- **描述:** 用于存储介质迁移（从SATA到SSD）的线程数。

#### check_consistency_worker_count

- **默认值:** 1
- **单位:** 不适用
- **描述:** 用于检查tablet一致性的线程数。

#### sys_log_dir

- **默认值:** `${STARROCKS_HOME}/log`
- **单位:** 不适用
- **描述:** 存放系统日志（包括INFO、WARNING、ERROR和FATAL）的目录。

#### user_function_dir

- **默认值:** `${STARROCKS_HOME}/lib/udf`
- **单位:** 不适用
- **描述:** 用于存储用户定义函数（UDFs）的目录。

#### small_file_dir

- **默认值:** `${STARROCKS_HOME}/lib/small_file`
- **单位:** 不适用
- **描述:** 用于存放文件管理器下载的文件的目录。

#### sys_log_level

- **默认值:** INFO
- **单位:** 不适用
- **描述:** 系统日志条目分类的严重性级别。有效值：INFO、WARN、ERROR和FATAL。

#### sys_log_roll_mode

- **默认值:** SIZE-MB-1024
- **单位:** 不适用
- **描述:** 系统日志被分段为日志卷的模式。有效值包括`TIME-DAY`、`TIME-HOUR`和`SIZE-MB-size`。默认值表示将日志分为卷，每卷大小为1GB。

#### sys_log_roll_num

- **默认值:** 10
- **单位:** 不适用
- **描述:** 要保留的日志卷数。

#### sys_log_verbose_modules

- **默认值:** 空字符串
- **单位:** 不适用
- **描述:** 要打印的日志模块。例如，如果将此配置项设置为OLAP，则StarRocks仅打印OLAP模块的日志。有效值为BE中的命名空间，包括starrocks、starrocks::debug、starrocks::fs、starrocks::io、starrocks::lake、starrocks::pipeline、starrocks::query_cache、starrocks::stream和starrocks::workgroup。

#### sys_log_verbose_level

- **默认值:** 10
- **单位:** 不适用
- **描述:** 要打印的日志级别。该配置项用于控制代码中VLOG发起的日志的输出。

#### log_buffer_level

- **默认值:** 空字符串
- **单位:** 不适用
- **描述:** 刷新日志的策略。默认值表示日志缓存在内存中。有效值为-1和0。-1表示日志不在内存中缓冲。

#### num_threads_per_core

- **默认值:** 3
- **单位:** 不适用
- **描述:** 每个CPU核心上启动的线程数。

#### compress_rowbatches
- **默认值**：TRUE
- **单位**：不适用
- **描述**：一个布尔值，用于控制是否在 BE 之间的 RPC 中压缩行批次。TRUE 表示压缩行批次，FALSE 表示不压缩它们。

#### serialize_batch

- **默认值**：FALSE
- **单位**：不适用
- **描述**：一个布尔值，用于控制是否在 BE 之间的 RPC 中序列化行批次。TRUE 表示序列化行批次，FALSE 表示不序列化它们。

#### storage_root_path

- **默认值**：`${STARROCKS_HOME}/storage`
- **单位**：不适用
- **描述**：存储卷的目录和介质。
  - 多个卷之间用分号 (`;`) 分隔。
  - 如果存储介质为 SSD，则在目录末尾添加 `medium:ssd`。
  - 如果存储介质为 HDD，则在目录末尾添加 `medium:hdd`。

#### max_length_for_bitmap_function

- **默认值**：1,000,000
- **单位**：字节
- **描述**：位图函数输入值的最大长度。

#### max_length_for_to_base64

- **默认值**：200,000
- **单位**：字节
- **描述**：to_base64() 函数的输入值的最大长度。

#### max_tablet_num_per_shard

- **默认值**：1,024
- **单位**：不适用
- **描述**：每个 shard 中的最大平板数。该配置项用于限制每个存储目录下的 tablet 子目录数量。

#### max_garbage_sweep_interval

- **默认值**：3,600
- **单位**：秒
- **描述**：存储卷上垃圾收集的最大时间间隔。

#### min_garbage_sweep_interval

- **默认值**：180
- **单位**：秒
- **描述**：存储卷上垃圾收集的最小时间间隔。

#### file_descriptor_cache_capacity

- **默认值**：16,384
- **单位**：不适用
- **描述**：可以缓存的文件描述符的数量。

#### min_file_descriptor_number

- **默认值**：60,000
- **单位**：不适用
- **描述**：BE 进程中文件描述符的最小数量。

#### index_stream_cache_capacity

- **默认值**：10,737,418,240
- **单位**：字节
- **描述**：BloomFilter、Min 和 Max 统计信息的缓存容量。

#### disable_storage_page_cache

- **默认值**：FALSE
- **单位**：不适用
- **描述**：一个布尔值，用于控制是否禁用 PageCache。
  - 启用 PageCache 后，StarRocks 会缓存最近扫描的数据。
  - PageCache 可以显著提高查询性能，特别是当频繁重复类似查询时。
  - TRUE 表示禁用 PageCache。
  - 自 StarRocks v2.4 起，此项的默认值已从 TRUE 更改为 FALSE。

#### base_compaction_num_threads_per_disk

- **默认值**：1
- **单位**：不适用
- **描述**：每个存储卷上用于基础压缩的线程数。

#### base_cumulative_delta_ratio

- **默认值**：0.3
- **单位**：不适用
- **描述**：累积文件大小与基础文件大小的比率。达到此比率是触发基础压缩的条件之一。

#### compaction_trace_threshold

- **默认值**：60
- **单位**：秒
- **描述**：每次压缩的时间阈值。如果压缩花费的时间超过时间阈值，StarRocks 会打印相应的跟踪信息。

#### be_http_port

- **默认值**：8,040
- **单位**：不适用
- **描述**：HTTP 服务器端口。

#### be_http_num_workers

- **默认值**：48
- **单位**：不适用
- **描述**：HTTP 服务器使用的线程数。

#### load_data_reserve_hours

- **默认值**：4
- **单位**：小时
- **描述**：小规模加载产生的文件的保留时间。

#### number_tablet_writer_threads

- **默认值**：16
- **单位**：不适用
- **描述**：用于 Stream Load 的线程数。

#### streaming_load_rpc_max_alive_time_sec

- **默认值**：1,200
- **单位**：秒
- **描述**：Stream Load 的 RPC 超时时间。

#### fragment_pool_thread_num_min

- **默认值**：64
- **单位**：不适用
- **描述**：用于查询的最小线程数。

#### fragment_pool_thread_num_max

- **默认值**：4,096
- **单位**：不适用
- **描述**：用于查询的最大线程数。

#### fragment_pool_queue_size

- **默认值**：2,048
- **单位**：不适用
- **描述**：每个 BE 节点上可以处理的查询数量的上限。

#### enable_token_check

- **默认值**：TRUE
- **单位**：不适用
- **描述**：一个布尔值，用于控制是否启用令牌检查。TRUE 表示启用令牌检查，FALSE 表示禁用令牌检查。

#### enable_prefetch

- **默认值**：TRUE
- **单位**：不适用
- **描述**：一个布尔值，用于控制是否启用查询的预取。TRUE 表示启用预取，FALSE 表示禁用预取。

#### load_process_max_memory_limit_bytes

- **默认值**：107,374,182,400
- **单位**：字节
- **描述**：BE 节点上所有加载进程可以占用的内存资源的最大大小限制。

#### load_process_max_memory_limit_percent

- **默认值**：30%
- **单位**：%
- **描述**：BE 节点上所有加载进程可以占用的内存资源的最大百分比限制。

#### sync_tablet_meta

- **默认值**：FALSE
- **单位**：不适用
- **描述**：一个布尔值，用于控制是否启用平板电脑元数据同步。TRUE 表示启用同步，FALSE 表示禁用同步。

#### routine_load_thread_pool_size

- **默认值**：10
- **单位**：不适用
- **描述**：每个 BE 上例程加载的线程池大小。从 v3.1.0 开始，此参数已被弃用。每个 BE 上例程加载的线程池大小现在由 FE 动态参数 max_routine_load_task_num_per_be 控制。

#### brpc_max_body_size

- **默认值**：2,147,483,648
- **单位**：字节
- **描述**：bRPC 的最大主体大小。

#### tablet_map_shard_size

- **默认值**：32
- **单位**：不适用
- **描述**：平板电脑地图分片大小。该值必须是 2 的幂。

#### enable_bitmap_union_disk_format_with_set

- **默认值**：FALSE
- **单位**：不适用
- **描述**：一个布尔值，用于控制是否启用 BITMAP 类型的新存储格式，可以提高 bitmap_union 的性能。TRUE 表示启用新的存储格式，FALSE 表示禁用它。

#### mem_limit

- **默认值**：90%
- **单位**：不适用
- **描述**：BE 进程内存上限。您可以将其设置为百分比（例如 "80%"）或物理限制（例如 "100GB"）。

#### flush_thread_num_per_store

- **默认值**：2
- **单位**：不适用
- **描述**：每个存储中用于刷新 MemTable 的线程数。

#### datacache_enable

- **默认值**：false
- **单位**：不适用
- **描述**：是否启用数据缓存。TRUE 表示启用数据缓存，FALSE 表示禁用数据缓存。

#### datacache_disk_path

- **默认值**：不适用
- **单位**：不适用
- **描述**：磁盘的路径。我们建议您为此参数配置的路径数量与 BE 机器上的磁盘数量相同。多个路径需要用分号 (`;`) 分隔。

#### datacache_meta_path

- **默认值**：不适用
- **单位**：不适用
- **描述**：块元数据的存储路径。您可以自定义存储路径。我们建议您将元数据存储在 $STARROCKS_HOME 路径下。

#### datacache_mem_size

- **默认值**：10%
- **单位**：不适用
- **描述**：内存中可以缓存的最大数据量。您可以将其设置为百分比（例如 `10%`）或物理限制（例如 `10G`、`21,474,836,480`）。默认值为 `10%`。我们建议您将此参数的值设置为至少 10 GB。

#### datacache_disk_size

- **默认值**：0
- **单位**：不适用
- **描述**：单个磁盘上可以缓存的最大数据量。您可以将其设置为百分比（例如 `80%`）或物理限制（例如 `2T`、`500G`）。例如，如果为 `datacache_disk_path` 参数配置两个磁盘路径，并将 `datacache_disk_size` 参数设置为 `21,474,836,480`（20 GB），则这两个磁盘上最多可以缓存 40 GB 数据。默认值为 `0`，表示仅使用内存来缓存数据。单位：字节。

#### jdbc_connection_pool_size

- **默认值**：8
- **单位**：不适用
- **描述**：JDBC 连接池大小。在每个 BE 节点上，访问具有相同 jdbc_url 的外部表的查询共享相同的连接池。

#### jdbc_minimum_idle_connections

- **默认值**：1
- **单位**：不适用
- **描述**：JDBC 连接池中的最小空闲连接数。

#### jdbc_connection_idle_timeout_ms

- **默认值**：600,000
- **单位**：毫秒
- **描述**：JDBC 连接池中的空闲连接过期的时间长度。如果 JDBC 连接池中的连接空闲时间超过该值，连接池将关闭超出配置项 jdbc_minimum_idle_connections 指定数量的空闲连接。

#### query_cache_capacity

- **默认值**：536,870,912
- **单位**：字节
- **描述**：BE 中的查询缓存大小。单位：字节。默认大小为 512 MB。大小不能小于 4 MB。如果 BE 的内存容量不足以提供您预期的查询缓存大小，您可以增加 BE 的内存容量。
#### 查询缓存容量

- **默认值**：536,870,912
- **单位**：字节
- **描述**：BE中查询缓存的大小。默认大小为512MB。大小不能小于4MB。如果BE的内存容量不足以支持您预期的查询缓存大小，您可以增加BE的内存容量。

#### 启用基于事件的压缩框架

- **默认值**：TRUE
- **单位**：N/A
- **描述**：是否启用基于事件的压缩框架。TRUE表示启用基于事件的压缩框架，FALSE表示禁用。启用基于事件的压缩框架可以在有许多Tablet或单个Tablet数据量很大的场景中，显著降低压缩的开销。

#### 启用大小分层压缩策略

- **默认值**：TRUE
- **单位**：N/A
- **描述**：是否启用大小分层压缩策略。TRUE表示启用大小分层压缩策略，FALSE表示禁用。

#### lake_service_max_concurrency

- **默认值**：0
- **单位**：N/A
- **描述**：在共享数据集群中，RPC请求的最大并发数。当达到此阈值时，将拒绝传入的请求。当此参数设置为0时，表示不对并发数进行限制。