---
displayed_sidebar: "Chinese"
---

# 架构

StarRocks具有简单的架构。整个系统仅由两种类型的组件组成，即前端（FE）和后端（BE）或计算节点（CN）。StarRocks不依赖任何外部组件，简化了部署和维护。节点可以进行水平扩展而无需服务停机。此外，StarRocks具有用于元数据和服务数据的副本机制，提高了数据可靠性并有效地防止了单点故障（SPOF）。

StarRocks兼容MySQL协议并支持标准SQL。用户可以轻松地从MySQL客户端连接到StarRocks以获得即时和有价值的见解。

## 架构演变

随着StarRocks不断发展，系统架构已从最初的存储-计算耦合架构（共享无）转变为存储-计算分离架构（共享数据）。

- 在3.0版本之前，StarRocks使用存储-计算耦合架构。BE负责数据存储和计算。数据访问和计算在本地节点上执行，以最小化数据移动并减少查询延迟，从而提供超快的查询和分析体验。

- 从3.0版本开始，StarRocks引入了存储-计算分离架构。数据存储被分离出BE，并且BE升级为无状态的CN节点。数据持久存储在远程对象存储或HDFS中，而CN本地磁盘用于缓存热数据以加速查询。存储-计算分离架构支持动态添加和移除计算节点，实现按需扩展。

以下图示了架构的演变。

![Architecture evolution](../assets/architecture_evolution.png)

## 存储-计算耦合

作为典型的大规模并行处理（MPP）数据库，StarRocks在3.0版本之前使用存储-计算耦合架构。在此架构中，BE负责数据存储和计算。在BE模式下直接访问本地数据，可以进行本地计算，避免数据传输和复制，从而提供了超快的查询和分析性能。该架构支持多副本数据存储，增强了集群处理高并发查询的能力，并确保了数据的可靠性。它非常适用于追求最佳查询性能的场景。

### 节点

在存储-计算耦合架构中，StarRocks由两种类型的节点组成：FE和BE。

- FE负责元数据管理和构建执行计划。
- BE执行查询计划并存储数据。BE利用本地存储加速查询，并使用多副本机制确保高可用性。

### FE

FE负责元数据管理、客户端连接管理、查询规划和查询调度。每个FE在内存中存储和维护完整的元数据副本，保证FE之间的服务无差别。FE可以作为领导者、跟随者和观察者工作。跟随者可以根据类似Paxos的BDB JE协议从跟随者FE中选举出一个领导者。BDB JE是伯克利DB Java Edition的缩写。

| **FE角色** | **元数据管理**                                           | **领导者选举**                                                         |
| --------------- | ------------------------------------------------------- | ------------------------------------- |
| 领导FE            | 领导FE负责读取和写入元数据。跟随者和观察者FE只能读取元数据。他们将元数据写入请求路由到领导FE。领导FE更新元数据，然后使用BDE JE将元数据更改同步到跟随者和观察者FE。仅在元数据更改同步到超过一半的跟随者FE之后，数据写入才被视为成功。 | 领导FE是从跟随者FE中选举出来的。 要执行领导者选举，需要集群中超过一半的跟随者FE是活动的。当领导FE失败时，跟随者FE将启动另一轮领导者选举。 |
| 跟随者FE         | 跟随者只能读取元数据。它们从领导FE同步和重放日志以更新元数据。 | 跟随者参与领导者选举，该选举需要超过集群中一半的跟随者是活动的。 |
| 观察者FE        | 观察者从领导FE同步和重放日志以更新元数据。 | 观察者主要用于增加集群的查询并发性。观察者不参与领导者选举，因此不会给集群增加领导者选举压力。 |

### BE

BE负责数据存储和SQL执行。

- 数据存储：BE具有相同的数据存储能力。FE根据预定义规则将数据分发给BE。BE对输入的数据进行转换，将数据写入所需格式，并为数据生成索引。

- SQL执行：当SQL查询到达时，FE根据查询语义将其解析为逻辑执行计划，然后将逻辑计划转换为可以在BE上执行的物理执行计划。存储目标数据的BE执行查询。这消除了数据传输和复制的需求，实现了高查询性能。

### 数据管理

StarRocks是一个面向列的数据库系统。它使用分区和分桶机制来管理数据。表中的数据首先被划分为多个分区，然后进一步划分为多个Tablet。Tablet是StarRocks中数据管理的基本逻辑单元。每个Tablet可以在不同的BE上存储多个副本。您可以指定Tablet的数量，由StarRocks负责Tablet。

分区和Tablet减少了表扫描并增加了查询并发性。副本促进了数据备份和恢复，防止数据丢失。

在下图中，表根据时间分为四个分区。第一个分区中的数据进一步划分为四个Tablet。每个Tablet有三个副本，分别存储在三个不同的BE上。

![数据管理](../assets/data_manage.png)

由于一个表被划分为多个Tablet，StarRocks可以将一个SQL语句调度到所有Tablet进行并行处理，充分利用多台物理机器和核心的计算能力。这也有助于将查询压力分摊到多个节点，增加了服务可用性。您可以按需添加物理机器来实现高并发。

Tablet的分布不受物理节点的影响或限制。如果BE的数量发生变化（例如，添加或删除BE），正在进行的服务可以继续进行而无需中断。节点更改将触发Tablet的自动迁移。如果添加了BE，则一些Tablet将自动迁移到新的BE以实现更均匀的数据分布。如果删除了BE，则这些BE上的Tablet将自动迁移到其他BE上，确保副本数量不变。自动Tablet迁移有助于轻松实现StarRocks集群的自动扩展，消除了手动数据再分布的需求。

StarRocks使用多副本机制（默认为3）用于Tablet。副本确保了高数据可靠性和服务可用性。一个节点的失败不会影响整体服务的可用性。您还可以增加副本数量以实现高查询并发。

### 限制

这种架构有其自身的局限性：

- 增长成本：用户必须将存储与计算一起扩展，增加了不希望出现的存储成本。随着数据量的增长，对存储和计算资源的需求不成比例地增加，导致资源效率低下。
- 复杂的架构：在多个副本中保持数据一致性给系统增加了复杂性，增加了其故障风险。
- 限制的弹性：扩展操作将导致数据再平衡，导致用户体验不佳。

## 存储-计算分离

在新的存储-计算分离架构中，数据存储功能与BE分离。现在称为“计算节点（CN）”的BE仅负责数据计算和缓存热数据。数据存储在低成本、可靠的远程存储系统中，例如Amazon S3、GCP、Azure Blob Storage和其他支持S3的存储，比如MinIO。当缓存命中时，查询性能与存储-计算耦合架构相媲美。CN节点可以在几秒内根据需求添加或移除。这种架构降低了存储成本，确保更好的资源隔离，以及高弹性和可伸缩性。

存储-计算分离架构与存储-计算耦合架构相比保持了简单的架构。它仅由两种类型的节点组成：FE和CN。唯一的区别是用户必须提供后端对象存储。

![shared-data-arch](../assets/architecture_shared_data.png)

### 节点

存储-计算分离架构中的FE提供与存储-计算耦合架构相同的功能。

BE的存储功能被分离。本地存储转变为共享存储。BE节点升级为无状态的CN节点，负责数据加载、查询计算和缓存管理等任务。

### 存储

目前，StarRocks共享数据集群支持两种存储解决方案：对象存储（例如AWS S3、Google GCS、Azure Blob Storage和MinIO）以及部署在传统数据中心中的HDFS。该技术统一了特定存储桶或HDFS目录中的数据的存储。

在共享数据集群中，数据文件格式与共享无集群保持一致（具有耦合存储和计算）。数据以段文件的形式组织，并在云原生表中重复使用各种索引技术。

### 缓存

StarRocks共享数据集群将数据存储和计算分离，使得二者可以独立扩展，从而降低成本并增强弹性。然而，这种架构可能会影响查询性能。

为了缓解影响，StarRocks建立了一个多层数据访问系统，覆盖内存、本地磁盘和远程存储，以更好地满足各种业务需求。

+ Queries against hot data scan the cache directly and then the local disk, while cold data needs to be loaded from the object storage into the local cache to accelerate subsequent queries. By keeping hot data close to compute units, StarRocks achieves truly high-performance computation and cost-effective storage. Moreover, access to cold data has been optimized with data prefetch strategies, effectively eliminating performance limits for queries. 

+ Users can decide whether to enable caching when creating tables. If caching is enabled, data will be written to both the local disk and backend object storage. During queries, the CN nodes first read data from the local disk. If the data is not found, it will be retrieved from the backend object storage and simultaneously cached on the local disk.
