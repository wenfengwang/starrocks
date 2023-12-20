---
displayed_sidebar: English
---

# 数据湖相关常见问题解答

本主题介绍有关数据湖的一些常见问题（FAQ）并提供这些问题的解决方案。本主题中提到的一些指标只能从 SQL 查询的 profile 中获取。要获取 SQL 查询的 profile，必须指定 `set enable_profile=true`。

## 慢 HDFS 节点

### 问题描述

当您访问 HDFS 集群中存储的数据文件时，您可能会发现您运行的 SQL 查询的 profile 中的 `__MAX_OF_FSIOTime` 和 `__MIN_OF_FSIOTime` 指标值之间存在巨大差异，这表明 HDFS 节点速度较慢。以下示例是指示 HDFS 节点速度下降问题的典型 profile：

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

- **[推荐]** 启用 [Data Cache](../data_source/data_cache.md) 功能，通过自动将外部存储系统中的数据缓存到 StarRocks 集群的 BEs 中，消除 HDFS 节点速度较慢对查询的影响。
- 启用 [Hedged Read](https://hadoop.apache.org/docs/r2.8.3/hadoop-project-dist/hadoop-common/release/2.4.0/RELEASENOTES.2.4.0.html) 功能。启用此功能后，如果从块读取速度很慢，StarRocks 会启动新的读取，该读取与原始读取并行运行，以针对不同的块副本进行读取。每当两个读取之一返回时，另一个读取就会被取消。**Hedged Read 功能有助于加速读取，但也会显著增加 Java 虚拟机（JVM）上的堆内存消耗。因此，如果您的物理机内存容量较小，建议您不要启用 Hedged Read 功能。**

#### [推荐] 数据缓存

请参阅 [Data Cache](../data_source/data_cache.md)。

#### Hedged Read

使用 BE 配置文件 `be.conf` 中的以下参数（从 v3.0 开始支持）在 HDFS 集群中启用和配置 Hedged Read 功能。

|参数|默认值|说明|
|---|---|---|
|hdfs_client_enable_hedged_read|false|指定是否启用 Hedged Read 功能。|
|hdfs_client_hedged_read_threadpool_size|128|指定 HDFS 客户端上 Hedged Read 线程池的大小。线程池大小限制了 HDFS 客户端中专用于运行 Hedged Read 的线程数量。此参数相当于 HDFS 集群的 `hdfs-site.xml` 文件中的 `dfs.client.hedged.read.threadpool.size` 参数。|
|hdfs_client_hedged_read_threshold_millis|2500|指定启动 Hedged Read 之前要等待的毫秒数。例如，您将此参数设置为 30。在这种情况下，如果对块的读取在 30 毫秒内没有返回，您的 HDFS 客户端会立即针对不同的块副本启动 Hedged Read。此参数相当于 HDFS 集群的 `hdfs-site.xml` 文件中的 `dfs.client.hedged.read.threshold.millis` 参数。|

如果查询 profile 中以下任一指标的值超过 `0`，则启用 Hedged Read 功能。

|指标|描述|
|---|---|
|TotalHedgedReadOps|启动的 Hedged Read 数量。|
|TotalHedgedReadOpsInCurThread|StarRocks 必须在当前线程而不是新线程中启动 Hedged Read 的次数，因为 Hedged Read 线程池已达到 `hdfs_client_hedged_read_threadpool_size` 参数指定的最大大小。|
|TotalHedgedReadOpsWin|Hedged Read 超过其原始读取的次数。|