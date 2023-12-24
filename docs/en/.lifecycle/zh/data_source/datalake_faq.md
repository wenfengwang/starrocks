---
displayed_sidebar: English
---

# 数据湖相关常见问题

本主题描述了有关数据湖的一些常见问题，并提供了针对这些问题的解决方案。本主题中提到的一些指标只能从 SQL 查询的配置文件中获取。要获取 SQL 查询的配置文件，必须指定 `set enable_profile=true`。

## HDFS节点速度慢

### 问题描述

当您访问存储在 HDFS 集群中的数据文件时，您可能会发现在您运行的 SQL 查询的配置文件中，`__MAX_OF_FSIOTime` 和 `__MIN_OF_FSIOTime` 这两个指标的值之间存在巨大差异，这表明 HDFS 节点速度较慢。以下示例是指示 HDFS 节点速度减慢问题的典型配置文件：

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

您可以使用以下解决方案之一来解决此问题：

- **[推荐]** 启用[数据缓存](../data_source/data_cache.md)功能，该功能通过自动将外部存储系统的数据缓存到 StarRocks 集群的 BE 中，消除 HDFS 节点对查询的影响。
- 启用[对冲读取](https://hadoop.apache.org/docs/r2.8.3/hadoop-project-dist/hadoop-common/release/2.4.0/RELEASENOTES.2.4.0.html)功能。启用该功能后，如果某个区块的读取速度较慢，StarRocks 会启动一个新的读取，该读取与原始读取并行运行，以读取不同的区块副本。每当两个读取中的一个返回时，另一个读取都会被取消。**对冲读取功能可以帮助加速读取，但它也会显著增加 Java 虚拟机（JVM）上的堆内存消耗。因此，如果您的物理机提供较小的内存容量，我们建议您不要启用对冲读取功能。**

#### [推荐] 数据缓存

请参阅[数据缓存](../data_source/data_cache.md)。

#### 对冲读取

在 BE 配置文件 `be.conf` 中使用以下参数（从 v3.0 开始支持）来启用和配置 Hedged Read 功能。

| 参数                                | 默认值 | 描述                                                         |
| ---------------------------------------- | ------------- | ------------------------------------------------------------------- |
| hdfs_client_enable_hedged_read           | false         | 指定是否启用对冲读取功能。                                    |
| hdfs_client_hedged_read_threadpool_size  | 128           | 指定 HDFS 客户端上 Hedged Read 线程池的大小。线程池大小限制了专用于在 HDFS 客户端中运行对冲读取的线程数。该参数等同于 HDFS 集群文件中的 `hdfs-site.xml` 参数 `dfs.client.hedged.read.threadpool.size`。|
| hdfs_client_hedged_read_threshold_millis | 2500          | 指定在启动对冲读取之前要等待的毫秒数。例如，您已将此参数设置为 `30`。在这种情况下，如果从块读取在 30 毫秒内未返回，则 HDFS 客户端会立即启动针对其他块副本的对冲读取。该参数等同于 HDFS 集群文件中的 `hdfs-site.xml` 参数 `dfs.client.hedged.read.threshold.millis`。|

如果查询配置文件中以下任何指标的值超过 `0`，则表示对冲读取功能已启用。

| 指标                         | 描述                                                  |
| ------------------------------ | ------------------------------------------------------------ |
| TotalHedgedReadOps             | 启动的对冲读取数。                 |
| TotalHedgedReadOpsInCurThread  | StarRocks 不得不在当前线程中而不是在新线程中启动对冲读取的次数，因为 Hedged Read 线程池已达到参数指定的最大大小 `hdfs_client_hedged_read_threadpool_size` 。 |
| TotalHedgedReadOpsWin          | 对冲读取次数超过其原始读取次数。 |
