---
displayed_sidebar: "Chinese"
sidebar_position: 1
---

# 使用Docker部署StarRocks

本快速入门指南将指导您通过在本地机器上使用Docker部署StarRocks的程序。在开始之前，您可以阅读[StarRocks架构](../introduction/Architecture.md)以获取更多的概念细节。

按照这些步骤，您可以部署一个简单的StarRocks集群，包括**一个FE节点**和**一个BE节点**。这有助于您完成接下来的快速入门教程，例如[创建表](../quick_start/Create_table.md)和[加载和查询数据](../quick_start/Import_and_query.md)，并让您熟悉StarRocks的基本操作。

> **注意**
>
> 本教程中使用的Docker镜像仅适用于需要使用小型数据集验证DEMO的情况，并不推荐用于大规模测试或生产环境。要部署高可用性的StarRocks集群，请参阅[部署概述](../deployment/deployment_overview.md)以获取适合您场景的其他选项。

## 先决条件

在Docker中部署StarRocks之前，请确保满足以下要求：

- **硬件**

  我们建议在具有8个CPU核心和16 GB内存或更多的机器上部署StarRocks。

- **软件**

  您必须在机器上安装以下软件：

  - [Docker Engine](https://docs.docker.com/engine/install/) (17.06.0或更高版本)并且您必须在meta目录的磁盘分区中至少有5GB的可用空间。详情请参阅https://github.com/StarRocks/starrocks/issues/35608。
  - MySQL客户端（5.5或更高版本）

## 步骤1：下载StarRocks Docker镜像

从[StarRocks Docker Hub](https://hub.docker.com/r/starrocks/allin1-ubuntu/tags)下载StarRocks Docker镜像。您可以根据镜像的标签选择特定的版本。

```Bash
sudo docker run -p 9030:9030 -p 8030:8030 -p 8040:8040 \
    -itd starrocks/allin1-ubuntu
```

> **故障排除**
>
> 如果主机上任何一个上述端口已被占用，系统会打印“docker: Error response from daemon: driver failed programming external connectivity on endpoint tender_torvalds (): Bind for 0.0.0.0:xxxx failed: port is already allocated.”。您可以通过更改命令中冒号（:）前面的端口来分配主机上的可用端口。

您可以通过运行以下命令来检查容器是否已创建并正常运行：

```Bash
sudo docker ps
```

如下所示，如果您的StarRocks容器的`STATUS`为`Up`，则您已成功在Docker容器中部署了StarRocks。

```Plain
CONTAINER ID   IMAGE                                          COMMAND                  CREATED         STATUS                 PORTS                                                                                                                             NAMES
8962368f9208   starrocks/allin1-ubuntu:branch-3.0-0afb97bbf   "/bin/sh -c ./start_…"   4 minutes ago   Up 4 minutes           0.0.0.0:8037->8030/tcp, :::8037->8030/tcp, 0.0.0.0:8047->8040/tcp, :::8047->8040/tcp, 0.0.0.0:9037->9030/tcp, :::9037->9030/tcp   xxxxx
```

## 步骤2：连接StarRocks

StarRocks部署成功后，您可以通过MySQL客户端连接到它。

```Bash
mysql -P9030 -h127.0.0.1 -uroot --prompt="StarRocks > "
```

> **注意**
>
> 如果您在`docker run`命令中为`9030`分配了不同的端口，您必须将上述命令中的`9030`替换为您分配的端口。

您可以通过执行以下SQL来检查FE节点的状态：

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

- 如果字段`Alive`为`true`，则此FE节点已正确启动并添加到集群中。
- 如果字段`Role`为`FOLLOWER`，则此FE节点有资格被选为Leader FE节点。
- 如果字段`Role`为`LEADER`，则此FE节点是Leader FE节点。

您可以通过执行以下SQL来检查BE节点的状态：

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

如果字段`Alive`为`true`，则此BE节点已正确启动并添加到集群中。

## 停止并移除Docker容器

在完成整个快速入门教程后，您可以使用容器ID停止和删除托管StarRocks集群的容器。

> **注意**
>
> 您可以通过运行`sudo docker ps`来获取Docker容器的`container_id`。

运行以下命令停止容器：

```Bash
# 用StarRocks集群的容器ID替换<container_id>。
sudo docker stop <container_id>
```

如果您不再需要容器，可以通过运行以下命令删除它：

```Bash
# 用StarRocks集群的容器ID替换<container_id>。
sudo docker rm <container_id>
```

> **注意**
>
> 删除容器是不可逆的。在删除之前，请确保您对容器中的重要数据进行了备份。

## 下一步操作

部署了StarRocks之后，您可以继续进行[创建表](../quick_start/Create_table.md)和[加载和查询数据](../quick_start/Import_and_query.md)的快速入门教程。