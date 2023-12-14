---
displayed_sidebar: "Chinese"
sidebar_position: 1
---

# Deploying StarRocks Using Docker

This quick start guide will walk you through deploying StarRocks on your local machine using Docker. Before you get started, you may want to read the [System Architecture](../introduction/Architecture.md) for more conceptual details.

You can deploy a simple StarRocks cluster with **one FE node** and **one BE node** by following the steps below. You can then proceed to the quick start tutorials for [creating tables](../quick_start/Create_table.md) and [importing and querying data](../quick_start/Import_and_query.md) to get familiar with the basic operations of StarRocks.

> **Note**
>
> The StarRocks cluster deployed using the Docker image below is only suitable for small dataset validation DEMOs. It is not recommended for large-scale testing or production environments. For deploying a high-availability StarRocks cluster, refer to [Deployment Overview](../deployment/deployment_overview.md) for other deployment methods suitable for your scenario.

## Prerequisites

Before deploying StarRocks in a Docker container, make sure the following environment requirements are met:

- **Hardware Requirements**

  It is recommended to deploy StarRocks on a computer with a minimum of 8 CPU cores and 16 GB of memory.

- **Software Requirements**

  You need to install the following software on your computer:

  - [Docker Engine](https://docs.docker.com/engine/install/) (version 17.06.0 or above)
  - MySQL client (version 5.5 or above)

## Step 1: Deploying StarRocks via Docker Image

Download the StarRocks Docker image from [StarRocks Docker Hub](https://hub.docker.com/r/starrocks/allin1-ubuntu/tags). You can choose a specific version of the image based on the tag.

```Bash
sudo docker run -p 9030:9030 -p 8030:8030 -p 8040:8040 \
    -itd starrocks.docker.scarf.sh/starrocks/allin1-ubuntu
```

> **Troubleshooting**
>
> If any of the above ports are occupied, the system will print an error log "docker: Error response from daemon: driver failed programming external connectivity on endpoint tender_torvalds (): Bind for 0.0.0.0:xxxx failed: port is already allocated". You can modify the port to another available one on the host by changing the port before the colon (:) in the command.

You can check if the container has been created and is running properly by running the following command:

```Bash
sudo docker ps
```

If the `STATUS` of the StarRocks container is `Up` as shown below, you have successfully deployed StarRocks in a Docker container.

```Plain
CONTAINER ID   IMAGE                                          COMMAND                  CREATED         STATUS                 PORTS                                                                                                                             NAMES
8962368f9208   starrocks/allin1-ubuntu:branch-3.0-0afb97bbf   "/bin/sh -c ./start_…"   4 minutes ago   Up 4 minutes           0.0.0.0:8037->8030/tcp, :::8037->8030/tcp, 0.0.0.0:8047->8040/tcp, :::8047->8040/tcp, 0.0.0.0:9037->9030/tcp, :::9037->9030/tcp   xxxxx
```

## Step 2: Connect to StarRocks

After successful deployment, you can connect to the StarRocks cluster using the MySQL client.

```Bash
mysql -P9030 -h127.0.0.1 -uroot --prompt="StarRocks > "
```

> **Note**
>
> If you assigned a different port other than `9030` in the `docker run` command, you must replace `9030` in the command above with the port you assigned.

You can view the status of the FE node by executing the following SQL:

```SQL
SHOW PROC '/frontends'\G
```

Example:

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

- If the `Alive` field is `true`, the FE node is started and joined the cluster successfully.
- If the `Role` field is `FOLLOWER`, the FE node is eligible to be elected as the Leader FE node.
- If the `Role` field is `LEADER`, the FE node is the Leader FE node.

You can view the status of the BE node by executing the following SQL:

```SQL
SHOW PROC '/backends'\G
```

Example:

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

If the `Alive` field is `true`, the BE node is started and joined the cluster successfully.

## Stopping and Removing the Docker Container

After completing the entire quick start guide, you can stop and remove the container of the StarRocks using its ID.

> **Note**
>
> You can obtain the `container_id` of the Docker container by running `sudo docker ps`.

Run the following command to stop the container:

```Bash
# Replace <container_id> with the container ID of your StarRocks cluster.
sudo docker stop <container_id>
```

If you no longer need the container, you can remove it by running the following command:

```Bash
# Replace <container_id> with the container ID of your StarRocks cluster.
sudo docker rm <container_id>
```

> **Note**
>
> The operation of deleting the container is irreversible. Before deleting, make sure you have backed up any important data in the container.

## Next Steps

成功部署 StarRocks 后，您可以继续完成有关 [创建表](../quick_start/Create_table.md) 和 [导入和查询数据](../quick_start/Import_and_query.md) 的快速入门教程。