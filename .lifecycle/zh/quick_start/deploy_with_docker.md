---
displayed_sidebar: English
sidebar_position: 1
---

# 使用 Docker 部署 StarRocks

本快速入门教程将指导您通过 Docker 在本地机器上部署 **StarRocks** 的步骤。在开始之前，您可以先阅读 [StarRocks Architecture](../introduction/Architecture.md) 以了解更多概念性的细节。

遵循这些步骤，您可以部署一个简易的 StarRocks 集群，包括一个 **FE 节点** 和一个 **BE 节点**。这将帮助您完成后续的快速入门教程，包括[创建表](../quick_start/Create_table.md)、和[加载和查询数据](../quick_start/Import_and_query.md)，从而让您熟悉 StarRocks 的基本操作。

> **警告**
> 本教程所用的 Docker 镜像部署 **StarRocks** 仅适合于验证小型数据集的 **DEMO** 环境。不推荐用于大规模的测试或生产环境。若要部署高可用性的 **StarRocks** 集群，请查看[部署概览](../deployment/deployment_overview.md)，了解更适合您场景的其他选项。

## 先决条件

在 Docker 中部署 StarRocks 之前，请确保满足以下要求：

- **硬件**

  建议在至少具备 8 个 CPU 核心和 16 GB 内存的机器上部署 StarRocks。

- **软件**

  您的机器上必须安装以下软件：

  - [Docker 引擎](https://docs.docker.com/engine/install/)（17.06.0 版本或更高）并且元数据目录所在的磁盘分区至少有 5GB 的空闲空间。详情请参见 https://github.com/StarRocks/starrocks/issues/35608 。
  - MySQL 客户端（5.5 版本或更高）

## 第 1 步：下载 StarRocks Docker 镜像

从 [StarRocks Docker Hub](https://hub.docker.com/r/starrocks/allin1-ubuntu/tags) 下载 StarRocks Docker 镜像。您可以根据镜像的标签选择特定版本。

```Bash
sudo docker run -p 9030:9030 -p 8030:8030 -p 8040:8040 \
    -itd starrocks/allin1-ubuntu
```

> **故障排除**
> 如果主机上的上述端口被占用，系统会显示错误信息：“docker: Error response from daemon: driver failed programming external connectivity on endpoint tender_torvalds (): Bind for 0.0.0.0:xxxx failed: port is already allocated.” 您可以通过更改命令中冒号（:）之前的端口来分配主机上的可用端口。

您可以通过运行以下命令来检查容器是否已创建并正常运行：

```Bash
sudo docker ps
```

如下所示，如果您的 StarRocks 容器状态为 Up，则说明您已成功地在 Docker 容器中部署了 StarRocks。

```Plain
CONTAINER ID   IMAGE                                          COMMAND                  CREATED         STATUS                 PORTS                                                                                                                             NAMES
8962368f9208   starrocks/allin1-ubuntu:branch-3.0-0afb97bbf   "/bin/sh -c ./start_…"   4 minutes ago   Up 4 minutes           0.0.0.0:8037->8030/tcp, :::8037->8030/tcp, 0.0.0.0:8047->8040/tcp, :::8047->8040/tcp, 0.0.0.0:9037->9030/tcp, :::9037->9030/tcp   xxxxx
```

## 第 2 步：连接到 StarRocks

StarRocks 正确部署后，您可以通过 MySQL 客户端连接到它。

```Bash
mysql -P9030 -h127.0.0.1 -uroot --prompt="StarRocks > "
```

> **警告**
> 如果您在 `docker run` 命令中为 `9030` 分配了其他端口，您必须在上述命令中将 `9030` 替换为您分配的端口。

您可以执行以下 SQL 来检查 FE 节点的状态：

```SQL
SHOW PROC '/frontends'\G
```

示例：

```Plain
StarRocks > SHOW PROC '/frontends'\G
*************************** 1. row ***************************
             Name: 8962368f9208_9010_1681370634632
               IP: 8962368f9208
      EditLogPort: 9010
         HttpPort: 8030
        QueryPort: 9030
          RpcPort: 9020
             Role: LEADER
        ClusterId: 555505802
             Join: true
            Alive: true
ReplayedJournalId: 99
    LastHeartbeat: 2023-04-13 07:28:50
         IsHelper: true
           ErrMsg: 
        StartTime: 2023-04-13 07:24:11
          Version: BRANCH-3.0-0afb97bbf
1 row in set (0.02 sec)
```

- 如果 Alive 字段为 true，说明该 FE 节点已正确启动并加入到集群中。
- 如果 Role 字段为 FOLLOWER，说明该 FE 节点有资格被选举为 Leader FE 节点。
- 如果 Role 字段为 LEADER，说明该 FE 节点是 Leader FE 节点。

您可以执行以下 SQL 来检查 BE 节点的状态：

```SQL
SHOW PROC '/backends'\G
```

示例：

```Plain
StarRocks > SHOW PROC '/backends'\G
*************************** 1. row ***************************
            BackendId: 10004
                   IP: 8962368f9208
        HeartbeatPort: 9050
               BePort: 9060
             HttpPort: 8040
             BrpcPort: 8060
        LastStartTime: 2023-04-13 07:24:25
        LastHeartbeat: 2023-04-13 07:29:05
                Alive: true
 SystemDecommissioned: false
ClusterDecommissioned: false
            TabletNum: 30
     DataUsedCapacity: 0.000 
        AvailCapacity: 527.437 GB
        TotalCapacity: 1.968 TB
              UsedPct: 73.83 %
       MaxDiskUsedPct: 73.83 %
               ErrMsg: 
              Version: BRANCH-3.0-0afb97bbf
               Status: {"lastSuccessReportTabletsTime":"2023-04-13 07:28:26"}
    DataTotalCapacity: 527.437 GB
          DataUsedPct: 0.00 %
             CpuCores: 16
    NumRunningQueries: 0
           MemUsedPct: 0.02 %
           CpuUsedPct: 0.1 %
1 row in set (0.00 sec)
```

如果 Alive 字段为 true，说明该 BE 节点已正确启动并加入到集群中。

## 停止并删除 Docker 容器

完成整个快速入门教程后，您可以通过容器 ID 停止并删除托管您 StarRocks 集群的容器。

> **注意**
> 您可以通过运行 `sudo docker ps` 获取您的 Docker 容器的 `container_id`。

运行以下命令来停止容器：

```Bash
# Replace <container_id> with the container ID of your StarRocks cluster.
sudo docker stop <container_id>
```

如果您不再需要该容器，可以通过运行以下命令来删除它：

```Bash
# Replace <container_id> with the container ID of your StarRocks cluster.
sudo docker rm <container_id>
```

> **警告**
> 容器的删除是不可逆的。在删除容器之前，请确保已备份了容器中的重要数据。

## 下一步

在部署了 StarRocks 之后，您可以继续学习有关[创建表](../quick_start/Create_table.md)以及[加载和查询数据](../quick_start/Import_and_query.md)的快速入门教程。

<img referrerpolicy="no-referrer-when-downgrade" src="https://static.scarf.sh/a.png?x-pxid=f5ae0b2c-3578-4a40-9056-178e9837cfe0" />

