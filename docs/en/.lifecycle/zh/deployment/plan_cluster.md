---
displayed_sidebar: English
---

# 规划 StarRocks 集群

本主题描述了如何从节点数、CPU 核心数、内存大小和存储大小的角度来规划生产环境中 StarRocks 集群的资源。

## 节点数

StarRocks 主要由两种类型的组件组成：FE 节点和 BE 节点。每个节点必须单独部署在物理机或虚拟机上。

### FE 节点数

FE 节点主要负责元数据管理、客户端连接管理、查询规划和查询调度。

在生产环境中，我们建议您在 StarRocks 集群中至少部署 **三个** Follower FE 节点，以防止单点故障（SPOF）。

StarRocks 使用 BDB JE 协议来管理 FE 节点上的元数据。StarRocks 会从所有 Follower FE 节点中选举出一个 Leader FE 节点。只有 Leader FE 节点可以写入元数据。其他 Follower FE 节点只会根据 Leader FE 节点的日志更新其元数据。每当 Leader FE 节点发生故障时，只要超过一半的 Follower FE 节点仍然存活，StarRocks 就会重新选举新的 Leader FE 节点。

如果您的应用程序生成高并发的查询请求，您可以向集群中添加 Observer FE 节点。Observer FE 节点只处理查询请求，不参与选举 Leader FE 节点。

### BE 节点数

BE 节点负责数据存储和 SQL 执行。

在生产环境中，我们建议您在 StarRocks 集群中至少部署 **三个 BE** 节点，以确保数据的高可靠性和服务的可用性。当至少部署并添加三个 BE 节点到 StarRocks 集群时，将自动形成一个高可用性的 BE 节点集群。一个 BE 节点的故障不会影响 BE 服务的整体可用性。

您可以增加 BE 节点的数量，以使 StarRocks 集群能够处理高并发的查询。

### CN 节点数

CN 节点是 StarRocks 的可选组件，只负责 SQL 的执行。

您可以增加 CN 节点的数量，以弹性扩展计算资源，而无需改变 StarRocks 集群中的数据分布。

## CPU 和内存

通常，FE 服务不会消耗大量的 CPU 和内存资源。我们建议为每个 FE 节点分配 8 个 CPU 核心和 16 GB RAM。

与 FE 服务不同，如果您的应用程序在大型数据集上处理高并发或复杂的查询，则 BE 服务可能会占用大量的 CPU 和内存。因此，我们建议为每个 BE 节点分配 16 个 CPU 核心和 64 GB RAM。

## 存储容量

### FE 存储

由于 FE 节点只在其存储中维护 StarRocks 的元数据，在大多数情况下，每个 FE 节点的 100 GB HDD 存储空间就足够了。

### BE 存储

#### 估算 BE 的初始存储空间

StarRocks 集群所需的总存储空间受原始数据大小、数据副本数和数据压缩算法的压缩比的影响。

使用以下公式，您可以估算所有 BE 节点所需的总存储空间：

```Plain
Total BE storage space = Raw data size * Replica count/Compression ratio

Raw data size = Sum of the space taken up by all fields in a row * Row count
```

在 StarRocks 中，表中的数据首先被划分为多个分区，然后被划分为多个 Tablet。Tablet 是 StarRocks 中数据管理的基本逻辑单元。为确保数据的高可靠性，您可以维护每个 Tablet 的多个副本，并将它们存储在不同的 BE 中。StarRocks 默认维护 3 个副本。

目前，StarRocks 支持四种数据压缩算法，按压缩比从高到低依次排列：zlib、Zstandard（或 zstd）、LZ4 和 Snappy。它们可以提供 3:1 到 5:1 的压缩比。

确定总存储空间后，只需将其除以集群中的 BE 节点数，即可估算出每个 BE 节点的平均存储空间。

#### 随时添加额外存储空间

如果 BE 存储空间随着原始数据的增长而耗尽，您可以通过垂直或水平扩展集群来补充它，或者简单地扩展云存储。

- 向 StarRocks 集群添加新的 BE 节点

  您可以向 StarRocks 集群添加新的 BE 节点，以便将数据重新均匀地重新分配到更多节点。有关详细说明，请参见 [扩展 StarRocks 集群 - 扩展 BE](../administration/Scale_up_down.md)。

  添加新的 BE 节点后，StarRocks 会自动在所有 BE 节点之间重新平衡数据。所有表类型都支持这种自动平衡。

- 向 BE 节点添加额外的存储卷

  您还可以向现有 BE 节点添加额外的存储卷。有关详细说明，请参见 [扩展 StarRocks 集群 - 扩展 BE](../administration/Scale_up_down.md)。

  添加额外的存储卷后，StarRocks 会自动重新平衡所有表中的数据。

- 添加云存储

  如果您的 StarRocks 集群部署在云上，您可以根据需要扩展云存储。有关详细说明，请联系您的云服务提供商。
