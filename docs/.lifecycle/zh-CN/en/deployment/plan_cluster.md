---
displayed_sidebar: "Chinese"
---

# 计划StarRocks集群

本主题描述了如何从节点数、CPU核心数、内存大小和存储大小的角度规划生产环境中StarRocks集群的资源。

## 节点数

StarRocks主要由两种类型的组件组成：FE节点和BE节点。每个节点必须单独部署在物理机器或虚拟机上。

### FE节点数

FE节点主要负责元数据管理、客户端连接管理、查询规划和查询调度。

在生产环境中，我们建议您在StarRocks集群中至少部署**三个**Follower FE节点，以防止单点故障（SPOFs）。

StarRocks使用BDB JE协议在FE节点之间管理元数据。StarRocks从所有Follower FE节点中选举Leader FE节点。只有Leader FE节点可以写入元数据。其他Follower FE节点仅根据Leader FE节点的日志更新其元数据。每当Leader FE节点失败时，StarRocks会重新选举新的Leader FE节点，只要超过半数的Follower FE节点是活动的。

如果您的应用程序生成高并发的查询请求，可以将Observer FE节点添加到您的集群中。Observer FE节点仅处理查询请求，并不参与Leader FE节点的选举。

### BE节点数

BE节点负责数据存储和SQL执行。

在生产环境中，我们建议您在StarRocks集群中至少部署**三个**BE节点，以确保高数据可靠性和服务可用性。当至少部署并添加三个BE节点到您的StarRocks集群时，将自动形成高可用性的BE集群。一个BE节点的故障不会影响BE服务的整体可用性。

您可以增加BE节点的数量，以使您的StarRocks集群能够处理高并发查询。

### CN节点数

CN节点是StarRocks的可选组件，仅负责SQL执行。

您可以增加CN节点的数量，以弹性扩展StarRocks集群的计算资源，而不会改变数据在StarRocks集群中的分布。

## CPU和内存

通常，FE服务不会占用大量的CPU和内存资源。我们建议为每个FE节点分配8个CPU核心和16 GB RAM。

与FE服务不同，如果您的应用程序在大型数据集上执行高并发或复杂查询，BE服务可能会大量消耗CPU和内存资源。因此，我们建议为每个BE节点分配16个CPU核心和64 GB RAM。

## 存储容量

### FE存储

因为FE节点只在其存储中维护StarRocks的元数据，所以在大多数情况下，每个FE节点的100 GB HDD存储空间就足够了。

### BE存储

#### 估算BE的初始存储空间

您的StarRocks集群所需的总存储空间同时受原始数据大小、数据副本数量以及您使用的数据压缩算法的压缩比的影响。

通过以下公式，您可以估算所有BE节点所需的总存储空间：

```Plain
Total BE storage space = 原始数据大小 * 副本数量/压缩比

原始数据大小 = 行中所有字段占用的空间之和 * 行数
```

在StarRocks中，表中的数据首先被划分为多个分区，然后再划分为多个Tablet。Tablet是StarRocks中数据管理的基本逻辑单元。为了确保高数据可靠性，您可以在每个Tablet上维护多个副本，并将它们存储在不同的BE上。StarRocks默认维护三个副本。

目前，StarRocks支持四种数据压缩算法，按压缩比从高到低排序分别为：zlib、Zstandard（或zstd）、LZ4和Snappy。它们可以提供从3:1到5:1的压缩比。

确定总存储空间后，您可以将其简单地除以集群中BE节点的数量，以估算每个BE节点的平均存储空间。

#### 随着数据量增加而增加额外存储空间

如果BE存储空间随着原始数据的增长而耗尽，您可以通过垂直或水平扩展集群，或简单地扩展您的云存储来补充它。

- 将新的BE节点添加到StarRocks集群

  您可以将新的BE节点添加到StarRocks集群，以使数据可以均匀重新分布到更多节点。有关详细说明，请参见[扩展您的StarRocks集群 - 扩展BE节点](../administration/Scale_up_down.md)。

  添加新的BE节点后，StarRocks会自动重新平衡所有BE节点中的数据。这种自动平衡支持所有表类型。

- 为已有的BE节点添加额外存储卷

  您还可以为已有的BE节点添加额外的存储卷。有关详细说明，请参见[扩展您的StarRocks集群 - 扩展BE节点](../administration/Scale_up_down.md)。

  添加额外的存储卷后，StarRocks会自动重新平衡所有表中的数据。

- 添加云存储

  如果您的StarRocks集群部署在云上，您可以根据需要扩展您的云存储。有关详细说明，请联系您的云服务提供商。