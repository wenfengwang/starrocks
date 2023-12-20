---
displayed_sidebar: English
---

# 规划StarRocks集群

本主题从节点数量、CPU核心数量、内存大小和存储大小等角度描述了如何为生产环境中的StarRocks集群规划资源。

## 节点数量

StarRocks主要由两种类型的组件构成：FE节点和BE节点。每个节点必须分别部署在物理机或虚拟机上。

### FE节点数量

FE节点主要负责元数据管理、客户端连接管理、查询规划和查询调度。

在生产环境中，我们建议您至少部署**三个**Follower FE节点在您的StarRocks集群中，以防止单点故障（SPOF）。

StarRocks使用BDB JE协议来跨FE节点管理元数据。StarRocks从所有Follower FE节点中选举出一个Leader FE节点。只有Leader FE节点可以写入元数据。其他Follower FE节点仅根据Leader FE节点的日志更新它们的元数据。每当Leader FE节点失败时，只要超过一半的Follower FE节点存活，StarRocks就会重新选举一个新的Leader FE节点。

如果您的应用产生了高并发查询请求，您可以在集群中增加Observer FE节点。Observer FE节点仅处理查询请求，并不参与Leader FE节点的选举。

### BE节点数量

BE节点负责数据存储和SQL执行。

在生产环境中，我们建议您至少部署**三个**BE节点在您的StarRocks集群中，以确保数据的高可靠性和服务的高可用性。当至少部署三个BE节点并将它们添加到您的StarRocks集群时，会自动形成一个高可用性的BE集群。一个BE节点的失败不会影响BE服务的整体可用性。

您可以增加BE节点的数量，以使您的StarRocks集群能够处理高并发查询。

### CN节点数量

CN节点是StarRocks的可选组件，仅负责SQL执行。

您可以增加CN节点的数量，以弹性地扩展计算资源，而无需改变您的StarRocks集群中的数据分布。

## CPU和内存

通常，FE服务不会消耗大量的CPU和内存资源。我们建议为每个FE节点分配8个CPU核心和16GB RAM。

与FE服务不同，BE服务在处理大数据集上的高并发或复杂查询时可能会显著消耗CPU和内存资源。因此，我们建议为每个BE节点分配16个CPU核心和64GB RAM。

## 存储容量

### FE存储

由于FE节点只维护StarRocks的元数据，所以在大多数情况下，每个FE节点100GB的HDD存储就足够了。

### BE存储

#### 估计BE的初始存储空间

您的StarRocks集群所需的总存储空间受到原始数据大小、数据副本数量和您使用的数据压缩算法的压缩比率的影响。

使用以下公式，您可以估算所有BE节点所需的总存储空间：

```Plain
Total BE storage space = Raw data size * Replica count / Compression ratio

Raw data size = Sum of the space taken up by all fields in a row * Row count
```

在StarRocks中，表中的数据首先被划分为多个分区，然后划分为多个tablet。Tablet是StarRocks中数据管理的基本逻辑单位。为了确保数据的高可靠性，您可以为每个tablet维护多个副本，并将它们存储在不同的BE节点上。默认情况下，StarRocks维护三个副本。

目前，StarRocks支持四种数据压缩算法，它们按压缩比从高到低排列为：zlib、Zstandard（或zstd）、LZ4和Snappy。这些算法可以提供3:1到5:1的压缩比。

确定了总存储空间后，您可以简单地将其除以集群中的BE节点数量，以估算每个BE节点的平均存储空间。

#### 随着需求增加额外存储空间

如果随着原始数据的增长BE存储空间变得不足，您可以通过垂直或水平扩展集群，或者简单地扩大云存储来补充它。

- 向您的StarRocks集群添加新的BE节点

  您可以向您的StarRocks集群添加新的BE节点，以便数据可以重新均匀地分布到更多的节点。有关详细说明，请参见[扩展您的StarRocks集群 - 横向扩展BE](../administration/Scale_up_down.md)。

  添加新的BE节点后，StarRocks会自动在所有BE节点之间重新平衡数据。所有表类型都支持此类自动平衡。

- 为您的BE节点添加额外的存储卷

  您还可以为现有的BE节点添加额外的存储卷。有关详细说明，请参见[扩展您的StarRocks集群 - 纵向扩展BE](../administration/Scale_up_down.md)。

  添加额外的存储卷后，StarRocks会自动在所有表中重新平衡数据。

- 添加云存储

  如果您的StarRocks集群部署在云上，您可以根据需要扩展云存储。有关详细说明，请联系您的云服务提供商。