---
displayed_sidebar: "中文"
---

# System Architecture

StarRocks has a simple architecture, with only two types of processes at the core of the entire system: FE (Frontend) and BE (Backend) or CN (Compute Node). This makes it easy to deploy and maintain. Nodes can be horizontally scaled online, and both metadata and business data have replica mechanisms to ensure that the entire system is free of single points of failure. StarRocks provides a MySQL protocol interface and supports standard SQL syntax. Users can easily query and analyze data in StarRocks using a MySQL client.

As the StarRocks product evolves, the system architecture has also evolved from the original shared-nothing architecture to a shared-data architecture.

- Prior to version 3.0, a shared-nothing architecture was used, where BE was responsible for both data storage and computation. Data access and analysis were performed locally, providing extremely fast query and analysis experiences.
- Starting from version 3.0, a shared-data architecture was introduced. The data storage function was separated from the original BE, and the BE nodes were upgraded to stateless CN nodes. Data can be persistently stored in remote object storage or HDFS, and the local disk of CN is used only to cache hot data to accelerate queries. The shared-data architecture supports dynamic addition and removal of computing nodes, achieving the ability to scale in and out in seconds.

The diagram below shows the evolution of the architecture from shared-nothing to shared-data.

![evolution](../assets/architecture_evolution.png)

## Shared-Nothing Architecture

As a typical representative of MPP databases, prior to version 3.0, StarRocks used a shared-nothing architecture, where BE was responsible for both data storage and computation. When querying, BE's local data could be accessed directly for local computation, avoiding data transmission and copying, thereby achieving extremely fast query and analysis performance. The shared-nothing architecture supports multi-replica data storage, enhancing the high-concurrency query capabilities and data reliability of the cluster. The shared-nothing architecture is suitable for scenarios that pursue ultimate query performance.

### Node Introduction

Under the shared-nothing architecture, StarRocks consists of FE and BE: FE is responsible for metadata management and building execution plans, while BE is responsible for actual execution and data storage management. BE uses local storage and ensures high availability through a mechanism of multiple replicas.

#### FE

FE is the frontend node of StarRocks, responsible for managing metadata, client connections, query planning, and query scheduling. Each FE node retains a complete set of metadata in memory, so that each FE node can provide the same level of service.

FE has three roles: Leader FE, Follower FE, and Observer FE, with the following differences.

| **FE Role** | **Metadata Read/Write**                                    | **Leader Election**                                      |
| --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Leader          | Leader FE provides metadata read/write services, while Follower and Observer only have read permissions with no write permissions. Follower and Observer route metadata write requests to Leader, and after Leader updates the metadata, it is synchronized to Follower and Observer through BDB JE (Berkeley DB Java Edition). The metadata write is considered successful only when over half of the Follower nodes successfully synchronize. | Automatically elected by Follower. If the current Leader node fails, Follower initiates a new round of elections. |
| Follower        | Only has metadata read permissions with no write permissions. It asynchronously synchronizes data by playing back Leader's metadata logs. | Follower participates in Leader election and automatically elects a Leader through a protocol similar to Paxos BDBJE. Over half of the Follower nodes must be alive to perform leader election. |
| Observer        | Same as Follower.                                            | Observer is mainly used to expand the cluster's query concurrency capabilities and is optional for deployment. Observer does not participate in leader election and does not increase the cluster's leader election pressure. |

#### BE

BE is the backend node of StarRocks, responsible for data storage and SQL computation.

- In terms of data storage, BE nodes are completely peer-to-peer. FE allocates data to the corresponding BE nodes according to a certain strategy. BE is responsible for writing imported data into the corresponding format for storage and generating related indexes.
- When executing SQL computations, a SQL statement is first planned into logical execution units according to semantics, and is then split into specific physical execution units according to data distribution. Physical execution units are executed on the corresponding BE nodes, enabling local computation to avoid data transmission and copying, thereby achieving extremely fast query performance.

### Data Management

StarRocks uses columnar storage and partition-bucketing mechanisms for data management. A table can be divided into multiple partitions, and the data within a partition can be bucketed based on one or more columns, splitting the data into multiple tablets. A tablet is the smallest data management unit in StarRocks. Each tablet is stored in the form of multiple replicas on different BE nodes. Users can specify the number and size of tablets, and StarRocks will manage the distribution information of each replica.

The diagram below shows the data division and the mechanism of multiple replicas for tablets in StarRocks. The table is divided into 4 partitions according to date, and the first partition is further divided into 4 tablets. Each tablet uses 3 replicas for backup, distributed across 3 different BE nodes.

![data_management](../assets/data_manage.png)

When executing SQL statements, StarRocks can concurrently process all tablets, fully utilizing the computational capabilities provided by multiple machines and multiple cores. Users can also distribute high-concurrency request pressure to multiple physical nodes and expand the system's high-concurrency capabilities by adding physical nodes.

StarRocks supports multiple replica storage for tablets (default is three), which ensures high reliable data storage and high availability of services. With three replicas, the abnormality of one node does not affect the availability of services, and the read and write services of the cluster can continue as usual. Increasing the number of replicas also helps improve the system's high-concurrency query capabilities.

When the number of BE nodes changes (such as when scaling in or out), StarRocks can automatically add or remove nodes without the need to stop services. Changes in nodes trigger automatic migration of tablets. When nodes are added, a portion of the tablets will automatically balance to the new nodes, ensuring more balanced data distribution within the cluster; when nodes are reduced, the tablets on the machine to be decommissioned will automatically balance to other nodes, thereby automatically ensuring that the number of replicas remains unchanged. Administrators can easily achieve elastic scaling without manually redistributing data.

The advantage of the shared-nothing architecture lies in its extremely fast query performance, but it also has some limitations:

- 成本高：需要使用三副本保证数据可靠性；随着用户存储数据量的增加，需要不断扩容存储资源，导致计算资源浪费。
- 架构复杂：存算一体架构需要维护多数据副本的一致性，增加了系统的复杂度。
- 弹性不够：存算一体模式下，扩缩容会触发数据重新平衡，弹性体验不佳。

## 存算分离

StarRocks 存算分离技术在现有存算一体架构的基础上，将计算和存储进行解耦。在存算分离新架构中，数据持久化存储在更为可靠和廉价的远程对象存储（比如 S3）或 HDFS 上。CN 本地磁盘只用于缓存热数据来加速查询。在本地缓存命中的情况下，存算分离可以获得与存算一体架构相同的查询性能。存算分离架构下，用户可以动态增删计算节点，实现秒级的扩缩容。存算分离大大降低了数据存储成本和扩容成本，有助于实现资源隔离和计算资源的弹性伸缩。

与存算一体架构类似，存算分离版本拥有同样简洁的架构，整个系统依然只有 FE 和 CN 两种服务进程，用户唯一需要额外提供的是后端对象存储。

![image](../assets/architecture_shared_data.png)

### 节点介绍

存算分离架构下，FE 的功能保持不变。BE 原有的存储功能被抽离，数据存储从本地存储 (local storage) 升级为共享存储 (shared storage)。BE 节点升级为无状态的 CN 节点，只缓存热数据。CN 会执行数据导入、查询计算、缓存数据管理等任务。

### 存储

目前，StarRocks 存算分离技术支持如下后端存储方式，用户可根据需求自由选择：

- 兼容 AWS S3 协议的对象存储系统（支持主流的对象存储系统如 AWS S3、Google GCP、阿里云 OSS、腾讯云 COS、百度云 BOS、华为云 OBS 以及 MinIO 等）
- Azure Blob Storage
- 传统数据中心部署的 HDFS

在数据格式上，StarRocks 存算分离数据文件与存算一体保持一致，各种索引技术在存算分离表中也同样适用，不同的是，描述数据文件的元数据（如 TabletMeta 等）被重新设计以更好地适应对象存储。

### 缓存

为了提升存算分离架构的查询性能，StarRocks 构建了分级的数据缓存体系，将最热的数据缓存在内存中，距离计算最近，次热数据则缓存在本地磁盘，冷数据位于对象存储，数据根据访问频率在三级存储中自由流动。

查询时，热数据通常会直接从缓存中命中，而冷数据则需要从对象存储中读取并填充至本地缓存，以加速后续访问。通过内存、本地磁盘、远端存储，StarRocks 存算分离构建了一个多层次的数据访问体系，用户可以指定数据冷热规则以更好地满足业务需求，让热数据靠近计算，真正实现高性能计算和低成本存储。

StarRocks 存算分离的统一缓存允许用户在建表时决定是否开启缓存。如果开启，数据写入时会同步写入本地磁盘以及后端对象存储，查询时，CN 节点会优先从本地磁盘读取数据，如果未命中，再从后端对象存储读取原始数据并同时缓存在本地磁盘。

同时，针对未被缓存的冷数据，StarRocks 也进行了针对性优化，可根据应用访问模式，利用数据预读技术、并行扫描技术等手段，减少对于后端对象存储的访问频次，提升查询性能。