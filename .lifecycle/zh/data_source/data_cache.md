---
displayed_sidebar: English
---

# 数据缓存

本文档介绍了数据缓存的工作原理以及如何启用数据缓存来提升外部数据的查询性能。

在数据湖分析中，StarRocks 作为一个 OLAP 引擎，用于扫描存储在外部存储系统中的数据文件，比如 HDFS 和 Amazon S3。随着待扫描文件数量的增加，I/O 开销也随之增加。此外，在某些即席查询场景中，频繁访问同一数据会使 I/O 开销翻倍。

为了在这些场景中优化查询性能，StarRocks 2.5 引入了数据缓存功能。此功能基于预设策略将外部存储系统中的数据分割成多个块，并将这些数据缓存在 StarRocks 后端（BE）上。这消除了每次访问请求都从外部系统拉取数据的需求，从而加快了对热数据的查询和分析速度。数据缓存仅在您通过使用外部目录或外部表（不包括兼容 JDBC 的数据库的外部表）从外部存储系统查询数据时有效。查询 StarRocks 本地表时，数据缓存不生效。

## 工作原理

StarRocks 将外部存储系统中的数据分割成多个默认大小为 1 MB 的块，并将这些数据缓存在 BE 上。块是数据缓存的最小单位，可以进行配置。

例如，如果您将块大小设置为 1 MB，并想从 Amazon S3 查询一个 128 MB 的 Parquet 文件，StarRocks 将文件分割成 128 个块。这些块分别是 [0, 1 MB)、[1 MB, 2 MB)、[2 MB, 3 MB) ... [127 MB, 128 MB)。StarRocks 为每个块分配一个全局唯一的标识符，称为缓存键。一个缓存键包括以下三个部分：

```Plain
hash(filename) + fileModificationTime + blockId
```

下表描述了每个部分的内容：

|组件项目|描述|
|---|---|
|filename|数据文件的名称。|
|fileModificationTime|数据文件的最后修改时间。|
|blockId|StarRocks 在分割数据文件时分配给块的 ID。该 ID 在同一数据文件下是唯一的，但在您的 StarRocks 集群内不是唯一的。|

如果查询命中了 [1 MB, 2 MB) 区间的块，StarRocks 将执行以下操作：

1. 检查缓存中是否存在该块。
2. 如果块已存在，StarRocks 从缓存中读取该块。如果块不存在，StarRocks 从 Amazon S3 读取该块并将其缓存在 BE 上。

在启用数据缓存后，StarRocks 会缓存从外部存储系统读取的数据块。如果您不希望缓存这些数据块，请运行以下命令：

```SQL
SET enable_populate_datacache = false;
```

有关 `enable_populate_datacache` 的更多信息，请参见[系统变量](../reference/System_variable.md)部分。

## 块的存储介质

StarRocks 利用 BE 机器的内存和磁盘来缓存数据块。它支持仅在内存中缓存，或同时在内存和磁盘中缓存。

如果您选择磁盘作为存储介质，缓存速度将直接受到磁盘性能的影响。因此，我们推荐您使用高性能磁盘，如 NVMe 磁盘来进行数据缓存。如果您没有高性能磁盘，可以增加更多磁盘来减轻磁盘 I/O 压力。

## 缓存替换策略

StarRocks 采用[最近最少使用](https://en.wikipedia.org/wiki/Cache_replacement_policies#Least_recently_used_(LRU))（LRU）策略来缓存和淘汰数据。

- StarRocks 首先尝试从内存中读取数据。如果内存中未找到数据，StarRocks 会从磁盘中读取数据，并尝试将从磁盘读取的数据加载到内存中。
- 从内存中淘汰的数据会被写入磁盘。从磁盘中淘汰的数据会被删除。

## 启用数据缓存

默认情况下，数据缓存功能是禁用的。要启用此功能，需要在 StarRocks 集群中配置前端（FE）和后端（BE）。

### FE 的配置

您可以通过以下任一方式为 FE 启用数据缓存：

- 根据您的需求为指定会话启用数据缓存。

  ```SQL
  SET enable_scan_datacache = true;
  ```

- 为所有活跃会话启用数据缓存。

  ```SQL
  SET GLOBAL enable_scan_datacache = true;
  ```

### BE 的配置

在每个 BE 的 **conf/be.conf** 文件中添加以下参数。然后重启每个 BE 以使设置生效。

|参数|说明|默认值|
|---|---|---|
|datacache_enable|是否启用数据缓存。true：启用数据缓存。false：禁用数据缓存。|false|
|datacache_disk_path|磁盘的路径。您可以配置多个磁盘，并用分号（;）分隔磁盘路径。我们建议您配置的路径数量与BE机器的磁盘数量相同。 BE启动时，StarRocks会自动创建磁盘缓存目录（如果父目录不存在则创建失败）。|${STARROCKS_HOME}/datacache|
|datacache_meta_path|块元数据的存储路径。您可以不指定此参数。|${STARROCKS_HOME}/datacache|
|datacache_mem_size|内存中可以缓存的最大数据量。您可以将其设置为百分比（例如，10%）或物理限制（例如，10G、21474836480）。我们建议您将此参数的值设置为至少 10 GB。|10%|
|datacache_disk_size|单个磁盘中可以缓存的最大数据量。您可以将其设置为百分比（例如80%）或物理限制（例如2T、500G）。例如，为datacache_disk_path参数配置两个磁盘路径，并将datacache_disk_size参数设置为21474836480（20GB），则这两个磁盘最多可以缓存40GB数据。|0，表示只缓存内存用于缓存数据。|

设置这些参数的示例。

```Plain

# Enable Data Cache.
datacache_enable = true  

# Configure the disk path. Assume the BE machine is equipped with two disks.
datacache_disk_path = /home/disk1/sr/dla_cache_data/;/home/disk2/sr/dla_cache_data/ 

# Set datacache_mem_size to 2 GB.
datacache_mem_size = 2147483648

# Set datacache_disk_size to 1.2 TB.
datacache_disk_size = 1288490188800
```

## 检查查询是否命中数据缓存

您可以通过分析查询性能概况中的以下指标来检查查询是否命中了数据缓存：

- DataCacheReadBytes：StarRocks 直接从内存和磁盘读取的数据量。
- DataCacheWriteBytes：从外部存储系统加载到 StarRocks 内存和磁盘的数据量。
- BytesRead：读取的数据总量，包括 StarRocks 从外部存储系统、其内存和磁盘读取的数据。

示例 1：在此示例中，StarRocks 从外部存储系统读取了大量数据（7.65 GB），而从内存和磁盘读取的数据量较少（518.73 MB）。这表明缓存命中率较低。

```Plain
 - Table: lineorder
 - DataCacheReadBytes: 518.73 MB
   - __MAX_OF_DataCacheReadBytes: 4.73 MB
   - __MIN_OF_DataCacheReadBytes: 16.00 KB
 - DataCacheReadCounter: 684
   - __MAX_OF_DataCacheReadCounter: 4
   - __MIN_OF_DataCacheReadCounter: 0
 - DataCacheReadTimer: 737.357us
 - DataCacheWriteBytes: 7.65 GB
   - __MAX_OF_DataCacheWriteBytes: 64.39 MB
   - __MIN_OF_DataCacheWriteBytes: 0.00 
 - DataCacheWriteCounter: 7.887K (7887)
   - __MAX_OF_DataCacheWriteCounter: 65
   - __MIN_OF_DataCacheWriteCounter: 0
 - DataCacheWriteTimer: 23.467ms
   - __MAX_OF_DataCacheWriteTimer: 62.280ms
   - __MIN_OF_DataCacheWriteTimer: 0ns
 - BufferUnplugCount: 15
   - __MAX_OF_BufferUnplugCount: 2
   - __MIN_OF_BufferUnplugCount: 0
 - BytesRead: 7.65 GB
   - __MAX_OF_BytesRead: 64.39 MB
   - __MIN_OF_BytesRead: 0.00
```

示例 2：在此示例中，StarRocks 从数据缓存中读取了大量数据（46.08 GB），没有从外部存储系统读取数据，这意味着所有数据均来自数据缓存。

```Plain
Table: lineitem
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
```
