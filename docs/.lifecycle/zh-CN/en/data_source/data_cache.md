---
displayed_sidebar: "Chinese"
---

# 数据缓存

本主题描述了数据缓存的工作原理以及如何启用数据缓存以提高对外部数据的查询性能。

在数据湖分析中，StarRocks作为OLAP引擎来扫描存储在外部存储系统（如HDFS和Amazon S3）中的数据文件。随着需要扫描的文件数量增加，I/O开销也会增加。此外，一些特定即席查询场景中，对相同数据的频繁访问会导致I/O开销翻倍。

为了优化这些场景下的查询性能，StarRocks 2.5提供了数据缓存功能。此功能将外部存储系统中的数据根据预定义策略分割为多个块，并将数据缓存在StarRocks后端（BEs）上。这消除了每次访问请求都需要从外部系统拉取数据的需要，并加速了热数据的查询和分析。数据缓存仅在通过使用外部目录或外部表（不包括面向JDBC兼容数据库的外部表）查询外部存储系统的数据时有效。当查询StarRocks本地表时，它不起作用。

## 工作原理

StarRocks将外部存储系统中的数据分割为相同大小的多个块（默认为1 MB），并将数据缓存在BEs上。块是数据缓存的最小单元，可以配置。

例如，如果将块大小设置为1 MB，并且希望从Amazon S3查询一个大小为128 MB的Parquet文件，StarRocks将文件分割为128个块。 这些块为[0, 1 MB)，[1 MB, 2 MB)，[2 MB, 3 MB)...[127 MB, 128 MB)。 StarRocks为每个块分配一个全局唯一的ID，称为缓存键。缓存键由以下三部分组成。

```纯文本
hash(filename) + fileModificationTime + blockId
```

以下表格提供了每个部分的描述。

| **组件项** | **描述**                                              |
| ------------------ | ------------------------------------------------------------ |
| 文件名           | 数据文件的名称。                                   |
| 文件修改时间 | 数据文件的最后修改时间。                  |
| 块ID            | StarRocks在拆分数据文件时为块分配的ID。该ID在相同的数据文件下是唯一的，但在您的StarRocks集群内不是唯一的。 |

如果查询命中[1 MB, 2 MB)块，StarRocks会执行以下操作：

1. 检查块是否存在于缓存中。
2. 如果块存在，StarRocks从缓存中读取块。如果块不存在，StarRocks会从Amazon S3读取块并将其缓存在BE上。

启用数据缓存后，StarRocks会缓存从外部存储系统读取的数据块。如果不希望缓存此类数据块，请运行以下命令：

```SQL
SET enable_populate_datacache = false;
```

有关`enable_populate_datacache`的更多信息，请参阅[系统变量](../reference/System_variable.md)。

## 块的存储介质

StarRocks使用BE机器的内存和磁盘来缓存块。它支持仅在内存上缓存或在内存和磁盘上都缓存。

如果将磁盘用作存储介质，则缓存速度直接受磁盘性能的影响。因此，我们建议您使用高性能的磁盘，比如NVMe磁盘进行数据缓存。如果没有高性能的磁盘，可以增加更多磁盘来减轻磁盘I/O压力。

## 缓存替换策略

StarRocks使用[最近最少使用](https://en.wikipedia.org/wiki/Cache_replacement_policies#Least_recently_used_(LRU)) (LRU)策略来缓存和丢弃数据。

- StarRocks首先从内存中读取数据。如果在内存中未找到数据，则StarRocks将从磁盘中读取数据并尝试将从磁盘读取的数据加载到内存中。
- 从内存中丢弃的数据将被写入磁盘。从磁盘中丢弃的数据将被删除。

## 启用数据缓存

数据缓存默认情况下是禁用的。要启用此功能，请配置StarRocks集群中的FE和BE。

### FE的配置

您可以通过以下方法之一为FE启用数据缓存：

- 根据您的需求为给定会话启用数据缓存。

  ```SQL
  SET enable_scan_datacache = true;
  ```

- 为所有活动会话启用数据缓存。

  ```SQL
  SET GLOBAL enable_scan_datacache = true;
  ```

### BE的配置

将以下参数添加到每个BE的**conf/be.conf**文件中。然后重新启动每个BE以使设置生效。

| **参数**          | **描述**                                              | **默认值** |
| ---------------------- | ------------------------------------------------------------ | -------------------|
| datacache_enable     | 是否启用数据缓存。<ul><li>`true`：启用数据缓存。</li><li>`false`：禁用数据缓存。</li></ul> | false |
| datacache_disk_path  | 磁盘的路径。您可以配置多个磁盘，并使用分号（;）将磁盘路径分隔开。我们建议您配置的路径数量与BE机器的磁盘数量相同。当BE启动时，StarRocks会自动创建一个磁盘缓存目录（如果不存在父目录则创建失败）。 | `${STARROCKS_HOME}/datacache` |
| datacache_meta_path  | 块元数据的存储路径。您可以不指定此参数。 | `${STARROCKS_HOME}/datacache` |
| datacache_mem_size   | 可缓存在内存中的最大数据量。您可以将其设置为百分比（例如，`10%`）或物理限制（例如，`10G`，`21474836480`）。我们建议您将此参数的值设置为至少10 GB。 | `10%` |
| datacache_disk_size  | 单个磁盘中可缓存的最大数据量。您可以将其设置为百分比（例如，`80%`）或物理限制（例如，`2T`，`500G`）。例如，如果将两个磁盘路径配置为`datacache_disk_path`参数，将`datacache_disk_size`参数的值设置为`21474836480`（20 GB），则可以在这两个磁盘上最多缓存40 GB的数据。  | `0`，表示仅使用内存来缓存数据。  |

设置这些参数的示例。

```纯文本

# 启用数据缓存。
datacache_enable = true

# 配置磁盘路径。假设BE机器配备两块磁盘。
datacache_disk_path = /home/disk1/sr/dla_cache_data/;/home/disk2/sr/dla_cache_data/

# 将datacache_mem_size设置为2 GB。
datacache_mem_size = 2147483648

# 将datacache_disk_size设置为1.2 TB。
datacache_disk_size = 1288490188800
```

## 检查查询是否命中数据缓存

您可以通过分析查询配置文件中的以下指标来检查查询是否命中数据缓存：

- `DataCacheReadBytes`：StarRocks直接从内存和磁盘中读取的数据量。
- `DataCacheWriteBytes`：从外部存储系统加载到StarRocks的内存和磁盘中的数据量。
- `BytesRead`：总共读取的数据量，包括StarRocks从外部存储系统、内存和磁盘中读取的数据。

示例1：在此示例中，StarRocks从外部存储系统读取了大量数据（7.65 GB），只从内存和磁盘中读取了少量数据（518.73 MB）。这意味着命中的数据缓存较少。

```纯文本
- 表：lineorder
- DataCacheReadBytes：518.73 MB
  - __DataCacheReadBytes的最大值：4.73 MB
  - __DataCacheReadBytes的最小值：16.00 KB
- DataCacheReadCounter：684
  - __DataCacheReadCounter的最大值：4
  - __DataCacheReadCounter的最小值：0
- DataCacheReadTimer：737.357微秒
- DataCacheWriteBytes：7.65 GB
  - __DataCacheWriteBytes的最大值：64.39 MB
  - __DataCacheWriteBytes的最小值：0.00 
- DataCacheWriteCounter：7.887K (7887)
  - __DataCacheWriteCounter的最大值：65
  - __DataCacheWriteCounter的最小值：0
- DataCacheWriteTimer：23.467毫秒
  - __DataCacheWriteTimer的最大值：62.280毫秒
  - __DataCacheWriteTimer的最小值：0纳秒
- BufferUnplugCount：15
  - __BufferUnplugCount的最大值：2
  - __BufferUnplugCount的最小值：0
- BytesRead：7.65 GB
  - __BytesRead的最大值：64.39 MB
  - __BytesRead的最小值：0.00
```

示例2：在此示例中，StarRocks从数据缓存中读取了大量数据（46.08 GB），没有从外部存储系统中读取任何数据，这意味着StarRocks仅从数据缓存中读取数据。

```纯文本
表：lineitem
- DataCacheReadBytes: 46.08 GB
   - __MAX_OF_DataCacheReadBytes: 194.99 MB
   - __MIN_OF_DataCacheReadBytes: 81.25 MB
- DataCacheReadCounter: 72.237K (72237)
   - __MAX_OF_DataCacheReadCounter: 299
   - __MIN_OF_DataCacheReadCounter: 118
- DataCacheReadTimer: 856.481ms
   - __MAX_OF_DataCacheReadTimer: 1s547ms
   - __MIN_OF_DataCacheReadTimer: 261.824ms
- DataCacheWriteBytes: 0.00 
- DataCacheWriteCounter: 0
- DataCacheWriteTimer: 0ns
- BufferUnplugCount: 1.231K (1231)
   - __MAX_OF_BufferUnplugCount: 81
   - __MIN_OF_BufferUnplugCount: 35
- BytesRead: 46.08 GB
   - __MAX_OF_BytesRead: 194.99 MB
   - __MIN_OF_BytesRead: 81.25 MB