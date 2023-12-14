---
displayed_sidebar: "Chinese"
---

# 与数据湖相关的常见问题解答

本主题描述了有关数据湖的一些常见问题（FAQ），并提供了这些问题的解决方案。本主题中提到的一些指标只能从SQL查询的概要文件中获取。要获取SQL查询的概要文件，必须指定`set enable_profile=true`。

## HDFS 节点缓慢

### 问题描述

当您访问存储在您的HDFS集群中的数据文件时，您可能会发现在您运行的SQL查询的概要文件中`__MAX_OF_FSIOTime`和`__MIN_OF_FSIOTime`指标的值之间存在很大差异，这表明HDFS节点缓慢。以下示例是一个典型的概要文件，表明出现了HDFS节点减速问题：

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

- **[推荐]** 启用[数据缓存](../data_source/data_cache.md)功能，该功能可以通过自动将外部存储系统的数据缓存到StarRocks集群的BE以消除HDFS节点对查询的影响。
- 启用[抵消读取](https://hadoop.apache.org/docs/r2.8.3/hadoop-project-dist/hadoop-common/release/2.4.0/RELEASENOTES.2.4.0.html)功能。启用此功能后，如果从块中读取的速度较慢，StarRocks会启动一个新的读取，与原始读取并行进行，以针对不同的块副本进行读取。一旦两次读取中的一次返回，另一次读取将被取消。**抵消读取功能可以帮助加速读取，但它也会显著增加Java虚拟机（JVM）的堆内存消耗。因此，如果您的物理机器提供的内存容量较小，我们建议您不要启用抵消读取功能。**

#### [推荐] 数据缓存

请参阅[数据缓存](../data_source/data_cache.md)。

#### 抵消读取

在BE配置文件`be.conf`中使用以下参数（从v3.0起开始支持）来启用和配置HDFS集群中的抵消读取功能。

| 参数                                    | 默认值         | 描述                                                         |
| ------------------------------------ | ------------- | ----------------------------------------------------------- |
| hdfs_client_enable_hedged_read           | false         | 指定是否启用抵消读取功能。                                    |
| hdfs_client_hedged_read_threadpool_size  | 128           | 指定HDFS客户端中抵消读取线程池的大小。线程池大小限制了在HDFS客户端中致力于运行抵消读取的线程数。该参数等同于HDFS集群的`hdfs-site.xml`文件中的`dfs.client.hedged.read.threadpool.size`参数。 |
| hdfs_client_hedged_read_threshold_millis | 2500          | 指定在启动抵消读取之前等待的毫秒数。例如，您已将此参数设置为`30`。在这种情况下，如果从块中读取的时间超过30毫秒，您的HDFS客户端将立即针对一个不同的块副本启动抵消读取。该参数等同于HDFS集群的`hdfs-site.xml`文件中的`dfs.client.hedged.read.threshold.millis`参数。 |

如果在您的查询概要文件中以下任何指标的值超过`0`，则抵消读取功能已启用。

| 指标                         | 描述                                                  |
| -------------------------- | --------------------------------------------------- |
| TotalHedgedReadOps             | 启动的抵消读取次数。                 |
| TotalHedgedReadOpsInCurThread  | StarRocks不得不在当前线程中而不是在新线程中启动抵消读取的次数，因为抵消读取线程池已达到`hdfs_client_hedged_read_threadpool_size`参数指定的最大大小。 |
| TotalHedgedReadOpsWin          | 抵消读取战胜原始读取次数。 |
