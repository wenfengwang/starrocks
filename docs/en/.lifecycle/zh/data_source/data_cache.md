---
displayed_sidebar: English
---

# 数据缓存

本主题介绍了数据缓存的工作原理以及如何启用数据缓存以提高对外部数据的查询性能。

在数据湖分析中，StarRocks作为OLAP引擎，用于扫描存储在外部存储系统（如HDFS和Amazon S3）中的数据文件。随着要扫描的文件数增加，I/O开销也会增加。此外，在某些特定的临时方案中，频繁访问相同的数据会使I/O开销增加。

为了优化这些场景下的查询性能，StarRocks 2.5提供了数据缓存功能。该功能将外部存储系统中的数据根据预定义策略拆分为多个块，并将数据缓存在StarRocks后端（BE）上。这样就无需为每个访问请求从外部系统拉取数据，并加快了对热数据的查询和分析。数据缓存仅在使用外部目录或外部表（不包括兼容JDBC的外部表）从外部存储系统查询数据时有效。在查询StarRocks原生表时，数据缓存不起作用。

## 工作原理

StarRocks将外部存储系统中的数据拆分为多个相同大小的块（默认为1 MB），并将数据缓存在BE上。块是数据缓存的最小单位，可以进行配置。

例如，如果将块大小设置为1 MB，并且要查询一个128 MB的Parquet文件来自Amazon S3，StarRocks会将该文件拆分为128个块。这些块分别是[0, 1 MB)、[1 MB, 2 MB)、[2 MB, 3 MB)……[127 MB, 128 MB)。StarRocks为每个块分配一个全局唯一的ID，称为缓存键。缓存键由以下三部分组成。

```Plain
hash(filename) + fileModificationTime + blockId
```

下表提供了每个部分的说明。

| **组件项** | **描述**                                              |
| ------------------ | ------------------------------------------------------------ |
| 文件名           | 数据文件的名称。                                   |
| 文件修改时间 | 数据文件的上次修改时间。                  |
| 块ID            | StarRocks在拆分数据文件时分配给块的ID。该ID在同一个数据文件下是唯一的，但在StarRocks集群中不是唯一的。 |

如果查询命中了[1 MB, 2 MB)块，StarRocks会执行以下操作：

1. 检查缓存中是否存在该块。
2. 如果该块存在，StarRocks会从缓存中读取该块。如果该数据块不存在，StarRocks会从Amazon S3读取该数据块并将其缓存在BE上。

启用数据缓存后，StarRocks会缓存从外部存储系统读取的数据块。如果不想缓存这些数据块，请运行以下命令：

```SQL
SET enable_populate_datacache = false;
```

有关`enable_populate_datacache`的更多信息，请参见[系统变量](../reference/System_variable.md)。

## 块的存储介质

StarRocks使用BE机器的内存和磁盘来缓存块。它支持仅在内存上缓存或同时在内存和磁盘上缓存。

如果使用磁盘作为存储介质，缓存速度会直接受到磁盘性能的影响。因此，建议您使用高性能磁盘（如NVMe磁盘）进行数据缓存。如果没有高性能磁盘，可以添加更多磁盘以减轻磁盘I/O压力。

## 缓存替换策略

StarRocks使用最近最少使用（LRU）策略进行数据缓存和丢弃。

- StarRocks首先从内存中读取数据。如果内存中没有找到数据，StarRocks会从磁盘读取数据，并尝试将从磁盘读取的数据加载到内存中。
- 从内存中丢弃的数据将写入磁盘。从磁盘中丢弃的数据将被删除。

## 启用数据缓存

数据缓存默认处于禁用状态。要启用此功能，请在StarRocks集群中配置FE和BE。

### FE的配置

您可以通过以下方法之一为FE启用数据缓存：

- 根据需要为特定会话启用数据缓存。

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
| datacache_enable     | 是否启用数据缓存。<ul><li>`true`：数据缓存已启用。</li><li>`false`：数据缓存已禁用。</li></ul> | false |
| datacache_disk_path  | 磁盘的路径。您可以配置多个磁盘，并使用分号（;）分隔磁盘路径。建议您配置的路径数与BE机器的磁盘数相同。当BE启动时，StarRocks会自动创建一个磁盘缓存目录（如果不存在父目录，则创建失败）。 | `${STARROCKS_HOME}/datacache` |
| datacache_meta_path  | 块元数据的存储路径。您可以不指定此参数。 | `${STARROCKS_HOME}/datacache` |
| datacache_mem_size   | 内存中可以缓存的最大数据量。您可以将其设置为百分比（例如，`10%`）或物理限制（例如，`10G`、`21474836480`）。建议您至少将该参数的值设置为10GB。 | `10%` |
| datacache_disk_size  | 单个磁盘中可以缓存的最大数据量。您可以将其设置为百分比（例如，`80%`）或物理限制（例如，`2T`、`500G`）。例如，如果为`datacache_disk_path`参数配置了两个磁盘路径，并将`datacache_disk_size`参数的值设置为`21474836480`（20GB），则这两个磁盘中最多可以缓存40GB的数据。 | `0`，表示仅使用内存缓存数据。 |

设置这些参数的示例。

```Plain

# 启用数据缓存。
datacache_enable = true  

# 配置磁盘路径。假设BE机器配备了两个磁盘。
datacache_disk_path = /home/disk1/sr/dla_cache_data/;/home/disk2/sr/dla_cache_data/ 

# 将datacache_mem_size设置为2GB。
datacache_mem_size = 2147483648

# 将datacache_disk_size设置为1.2TB。
datacache_disk_size = 1288490188800
```

## 检查查询是否命中数据缓存

您可以通过分析查询配置文件中的以下指标来检查查询是否命中数据缓存：

- `DataCacheReadBytes`：StarRocks直接从内存和磁盘读取的数据量。
- `DataCacheWriteBytes`：从外部存储系统加载到StarRocks内存和磁盘的数据量。
- `BytesRead`：读取的数据总量，包括StarRocks从外部存储系统、内存和磁盘读取的数据。

示例1：在此示例中，StarRocks从外部存储系统读取了大量数据（7.65GB），而只从内存和磁盘读取了少量数据（518.73MB）。这意味着命中的数据缓存很少。

```Plain
 - 表：lineorder
 - DataCacheReadBytes：518.73MB
   - __MAX_OF_DataCacheReadBytes：4.73MB
   - __MIN_OF_DataCacheReadBytes：16.00KB
 - DataCacheReadCounter：684
   - __MAX_OF_DataCacheReadCounter：4
   - __MIN_OF_DataCacheReadCounter：0
 - DataCacheReadTimer：737.357us
 - DataCacheWriteBytes：7.65GB
   - __MAX_OF_DataCacheWriteBytes：64.39MB
   - __MIN_OF_DataCacheWriteBytes：0.00 
 - DataCacheWriteCounter：7.887K（7887）
   - __MAX_OF_DataCacheWriteCounter：65
   - __MIN_OF_DataCacheWriteCounter：0
 - DataCacheWriteTimer：23.467ms
   - __MAX_OF_DataCacheWriteTimer：62.280ms
   - __MIN_OF_DataCacheWriteTimer：0ns
 - BufferUnplugCount：15
   - __MAX_OF_BufferUnplugCount：2
   - __MIN_OF_BufferUnplugCount：0
 - BytesRead：7.65GB
   - __MAX_OF_BytesRead：64.39MB
   - __MIN_OF_BytesRead：0.00
```

示例2：在此示例中，StarRocks从数据缓存中读取了大量数据（46.08GB），而没有从外部存储系统读取数据，这意味着StarRocks只从数据缓存中读取数据。

```Plain
表：lineitem
- DataCacheReadBytes：46.08GB
 - __MAX_OF_DataCacheReadBytes：194.99MB
 - __MIN_OF_DataCacheReadBytes：81.25MB
- DataCacheReadCounter：72.237K（72237）
 - __MAX_OF_DataCacheReadCounter：299
 - __MIN_OF_DataCacheReadCounter：118
- DataCacheReadTimer：856.481ms
 - __MAX_OF_DataCacheReadTimer：1s547ms
 - __MIN_OF_DataCacheReadTimer：261.824ms
- DataCacheWriteBytes：0.00 
- DataCacheWriteCounter：0
- DataCacheWriteTimer：0ns
- BufferUnplugCount：1.231K（1231）
 - __MAX_OF_BufferUnplugCount：81
 - __MIN_OF_BufferUnplugCount：35
- BytesRead：46.08GB
 - __MAX_OF_BytesRead：194.99MB
 - __MIN_OF_BytesRead：81.25MB
```
