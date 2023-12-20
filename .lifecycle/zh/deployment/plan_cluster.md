---
displayed_sidebar: English
---

# 规划 StarRocks 集群

本主题从节点数量、CPU 核心数量、内存大小和存储容量等多个角度介绍了如何为生产环境中的 StarRocks 集群规划资源。

## 节点数量

StarRocks 主要包含两种类型的组件：FE 节点和 BE 节点。每个节点都必须单独部署在一台物理机或虚拟机上。

### FE 节点数量

FE 节点主要负责元数据管理、客户端连接管理、查询计划和查询调度。

在生产环境中，我们建议您至少部署**三个**Follower FE节点在您的StarRocks集群中，以防单点故障（SPOF）。

StarRocks 采用 BDB JE 协议来管理跨 FE 节点的元数据。StarRocks 会从所有 Follower FE 节点中选举出一个 Leader FE 节点。只有 Leader FE 节点可以写入元数据。其他 Follower FE 节点只能根据 Leader FE 节点的日志来更新它们的元数据。每当 Leader FE 节点出现故障，只要超过半数的 Follower FE 节点仍然存活，StarRocks 就会重新选举一个新的 Leader FE 节点。

如果您的应用产生了大量的并发查询请求，您可以在集群中添加 Observer FE 节点。Observer FE 节点仅处理查询请求，并不参与 Leader FE 节点的选举。

### BE 节点数量

BE 节点负责数据存储和 SQL 执行。

在生产环境中，我们建议您至少部署**三个**BE节点在您的StarRocks集群中，以确保数据的高可靠性和服务的高可用性。当至少部署三个BE节点并将它们添加到StarRocks集群时，就会自动形成一个高可用的BE集群。一个BE节点的失败不会影响BE服务的整体可用性。

您可以增加 BE 节点的数量，以使 StarRocks 集群能够处理更多的高并发查询。

### CN 节点数量

CN 节点是 StarRocks 的可选组件，仅负责 SQL 执行。

您可以增加 CN 节点的数量，以便在不改变 StarRocks 集群中的数据分布的情况下，弹性地扩展计算资源。

## CPU 和内存

通常情况下，FE 服务不会消耗过多的 CPU 和内存资源。我们建议为每个 FE 节点分配 8 个 CPU 核心和 16 GB RAM。

与 FE 服务不同，BE 服务在处理大型数据集上的高并发或复杂查询时，可能会显著增加 CPU 和内存的使用。因此，我们建议为每个 BE 节点分配 16 个 CPU 核心和 64 GB RAM。

## 存储容量

### FE 存储

由于 FE 节点仅在其存储中维护 StarRocks 的元数据，所以在大多数情况下，每个 FE 节点 100 GB 的硬盘驱动器（HDD）存储空间就足够了。

### BE 存储

#### 估计 BE 的初始存储空间

您的 StarRocks 集群所需的总存储空间受到原始数据大小、数据副本数量和您所使用的数据压缩算法的压缩比率的共同影响。

您可以使用以下公式来估算所有 BE 节点所需的总存储空间：

```Plain
Total BE storage space = Raw data size * Replica count/Compression ratio

Raw data size = Sum of the space taken up by all fields in a row * Row count
```

在 StarRocks 中，表中的数据首先被划分成多个分区，然后进一步划分成多个 tablet。Tablet 是 StarRocks 中数据管理的基本逻辑单元。为了确保数据的高可靠性，您可以为每个 tablet 维护多个副本，并将它们分布在不同的 BE 上。默认情况下，StarRocks 维护三个副本。

目前，StarRocks 支持四种数据压缩算法，它们按照压缩比从高到低依次是：zlib、Zstandard（或 zstd）、LZ4 和 Snappy。这些算法可以提供从 3:1 到 5:1 的压缩比。

确定了总存储空间之后，您可以简单地将其除以集群中 BE 节点的数量，以估算每个 BE 节点的平均存储空间。

#### 随着需要增加额外的存储空间

如果随着原始数据的增长，BE 的存储空间不足，您可以通过垂直或水平扩展集群，或者简单地增加云存储来进行补充。

- 向您的 StarRocks 集群添加新的 BE 节点

  您可以向 StarRocks 集群添加新的 BE 节点，以便数据能够重新均匀分布到更多的节点。详细操作指南，请参见[扩展 StarRocks 集群 - 横向扩展 BE](../administration/Scale_up_down.md)。

  在添加了新的 BE 节点后，StarRocks 会自动在所有 BE 节点间重新平衡数据。这种自动平衡适用于所有类型的表。

- 为您的 BE 节点增加额外的存储卷

  您也可以为现有的 BE 节点增加额外的存储卷。详细操作指南，请参见[扩展 StarRocks 集群 - 纵向扩展 BE](../administration/Scale_up_down.md)。

  增加了额外存储卷后，StarRocks 将自动在所有表中重新平衡数据。

- 增加云存储

  如果您的 StarRocks 集群部署在云端，您可以根据需求扩大云存储。详细操作指南，请联系您的云服务提供商。
