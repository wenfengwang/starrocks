---
displayed_sidebar: English
---

# 数据湖相关常见问题解答

本主题介绍了关于数据湖的一些常见问题（FAQ），并提供了这些问题的解决方案。本主题中提及的某些指标只能从SQL查询的性能分析文件中获得。要获取SQL查询的性能分析文件，您必须设置 enable_profile=true。

## HDFS节点响应慢

### 问题描述

当您访问存储在HDFS集群中的数据文件时，可能会发现运行的SQL查询的性能分析文件中的__MAX_OF_FSIOTime与__MIN_OF_FSIOTime指标之间存在很大差异，这表明有HDFS节点响应慢的情况。以下是一个典型的性能分析文件示例，它显示了HDFS节点响应变慢的问题：

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

您可以采用以下解决方案之一来解决此问题：

- **【推荐】**启用[数据缓存](../data_source/data_cache.md)功能，通过自动将外部存储系统中的数据缓存到您的StarRocks集群的BE（Backend）中，从而减少慢速HDFS节点对查询的影响。
- 启用[Hedged Read](https://hadoop.apache.org/docs/r2.8.3/hadoop-project-dist/hadoop-common/release/2.4.0/RELEASENOTES.2.4.0.html)（对冲读取）功能。启用此功能后，如果从一个数据块的读取速度很慢，StarRocks将启动一个新的读取操作，与原始读取并行执行，针对不同的数据块副本进行读取。无论哪一个读取操作先完成，另一个读取操作就会被取消。**Hedged Read功能可以帮助加快读取速度，但同时也会显著增加Java虚拟机（JVM）上的堆内存使用。因此，如果您的物理机器提供的内存容量有限，我们建议不要启用Hedged Read功能。**

#### 【推荐】数据缓存

详见[数据缓存](../data_source/data_cache.md)部分。

#### Hedged Read（对冲读取）

在HDFS集群中启用和配置Hedged Read功能，可以在BE配置文件be.conf中使用以下参数（自v3.0版本起支持）：

|参数|默认值|说明|
|---|---|---|
|hdfs_client_enable_hedged_read|false|是否启用对冲读功能。|
|hdfs_client_hedged_read_threadpool_size|128|指定 HDFS 客户端上对冲读取线程池的大小。线程池大小限制了 HDFS 客户端中专用于运行对冲读取的线程数量。此参数相当于 HDFS 集群的 hdfs-site.xml 文件中的 dfs.client.hedged.read.threadpool.size 参数。|
|hdfs_client_hedged_read_threshold_millis|2500|指定启动对冲读取之前要等待的毫秒数。例如，您将此参数设置为 30。在这种情况下，如果对块的读取在 30 毫秒内没有返回，您的 HDFS 客户端会立即针对不同的块副本启动对冲读取。此参数相当于 HDFS 集群的 hdfs-site.xml 文件中的 dfs.client.hedged.read.threshold.millis 参数。|

如果您的查询性能分析文件中以下任一指标的值超过0，则表示Hedged Read功能已经启用。

|指标|描述|
|---|---|
|TotalHedgedReadOps|启动的对冲读取数量。|
|TotalHedgedReadOpsInCurThread|StarRocks 必须在当前线程而不是新线程中启动对冲读取的次数，因为对冲读取线程池已达到 hdfs_client_hedged_read_threadpool_size 参数指定的最大大小。|
|TotalHedgedReadOpsWin|对冲读取超过其原始读取的次数。|
