---
displayed_sidebar: "中文"
---

# 数据湖相关 FAQ

本文介绍有关数据湖的常见问题以及解决方法。在文章中，很多提及的指标需要你通过 `set enable_profile=true` 设置以搜集 SQL 查询的 Profile 并进行分析。

## HDFS 慢节点问题

### 问题描述

访问 HDFS 上存储的数据文件时，如果发现 SQL 查询的 Profile 中 `__MAX_OF_FSIOTime` 与 `__MIN_OF_FSIOTime` 两项指标差距很大，则说明当前环境存在 HDFS 慢节点问题。以下 Profile 数据即展示了经典的 HDFS 慢节点情况：

```plaintext
 - InputStream: 0
   - AppIOBytesRead: 22.72 GB
     - __MAX_OF_AppIOBytesRead: 187.99 MB
     - __MIN_OF_AppIOBytesRead: 64.00 KB
   - AppIOCounter: 964.862K (964862)
     - __MAX_OF_AppIOCounter: 7.795K (7795)
     - __MIN_OF_AppIOCounter: 1
   - AppIOTime: 1s372ms
     - __MAX_OF_AppIOTime: 4s358ms
     - __MIN_OF_AppIOTime: 1.539ms
   - FSBytesRead: 15.40 GB
     - __MAX_OF_FSBytesRead: 127.41 MB
     - __MIN_OF_FSBytesRead: 64.00 KB
   - FSIOCounter: 1.637K (1637)
     - __MAX_OF_FSIOCounter: 12
     - __MIN_OF_FSIOCounter: 1
   - FSIOTime: 9s357ms
     - __MAX_OF_FSIOTime: 60s335ms
     - __MIN_OF_FSIOTime: 1.536ms
```

### 解决方案

目前有两种解决方案：

- 【推荐】启用 [Data Cache](../data_source/data_cache.md) 功能。通过将远端数据自动缓存到 BE 节点上，消除 HDFS 慢节点对查询影响。
- 启用 [Hedged Read](https://hadoop.apache.org/docs/r2.8.3/hadoop-project-dist/hadoop-common/release/2.4.0/RELEASENOTES.2.4.0.html) 功能。开启之后，如果从某个数据块读取数据较慢，StarRocks 将发起一个新的 Read 任务，与原来的 Read 任务同时进行，目的是从该数据块的副本上进行读取。无论哪个 Read 任务先返回结果，另一个 Read 任务将被取消。**Hedged Read 能够加快数据读取速度，但会显著增加 Java 虚拟机（简称“JVM”）堆内存的使用。因此，在物理机内存较小时，不推荐开启 Hedged Read 功能。**

#### 【推荐】Data Cache

请参阅 [Data Cache](../data_source/data_cache.md)。

#### Hedged Read

从 3.0 版本开始，在 BE 的配置文件 `be.conf` 中，通过设置以下参数来启用并配置 HDFS 集群的 Hedged Read 功能。

| 参数名称                                 | 默认值 | 说明                                                         |
| ---------------------------------------- | ------ | ------------------------------------------------------------ |
| hdfs_client_enable_hedged_read           | false  | 是否启用 Hedged Read 功能。 |
| hdfs_client_hedged_read_threadpool_size  | 128    | HDFS 客户端侧 Hedged Read 线程池大小，即允许多少线程来服务 Hedged Read。该参数与 HDFS 集群配置文件 `hdfs-site.xml` 中的 `dfs.client.hedged.read.threadpool.size` 对应。 |
| hdfs_client_hedged_read_threshold_millis | 2500   | 触发 Hedged Read 请求前需要等待的时间，单位为毫秒。例如，若该参数设置为 `30`，那么如若一个 Read 任务在 30 毫秒内未返回结果，则 HDFS 客户端会立即发起 Hedged Read，从数据块的副本上读取数据。该参数与 HDFS 集群配置文件 `hdfs-site.xml` 中的 `dfs.client.hedged.read.threshold.millis` 对应。 |

如果 Profile 中观测到下列任一指标的数值大于 `0`，意味着 Hedged Read 功能已成功开启。

| 指标                            | 说明                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| TotalHedgedReadOps             | Hedged Read 发起次数。                                      |
| TotalHedgedReadOpsInCurThread  | 由于 Hedged Read 线程池大小限制（通过 `hdfs_client_hedged_read_threadpool_size` 配置）而不能开启新线程，只能在当前线程中触发 Hedged Read 的次数。 |
| TotalHedgedReadOpsWin          | Hedged Read 任务相比原始 Read 任务更早返回结果的次数。 |