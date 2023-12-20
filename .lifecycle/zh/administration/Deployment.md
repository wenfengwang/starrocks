---
displayed_sidebar: English
---

# 部署具有高可用性的FE集群

本主题将介绍StarRocks的FE节点如何实现高可用性（HA）部署。

## 理解FE HA集群

StarRocks的FE高可用集群采用主从复制架构，以避免出现单点故障。FE使用类raft协议的BDB JE来完成领导者选举、日志复制和故障切换。

### 了解FE的角色

在FE集群中，节点被划分为以下两种角色：

- **FE追随者**

FE追随者是复制协议中有投票权的成员，参与选举Leader FE并提交日志。FE追随者的数量是奇数（2n+1），它需要多数票（n+1）来确认操作，并能容忍少数故障（n）。

- **FE观察者**

FE观察者是没有投票权的成员，用于异步订阅复制日志。在集群中，FE观察者的状态稍落后于追随者，类似于其他复制协议中的学习者角色。

FE集群会自动从FE追随者中选出Leader FE。Leader FE负责执行所有状态变更。变更可以由非Leader FE节点发起，然后转发给Leader FE执行。非Leader FE节点会在复制日志中记录最新变更的LSN（日志序列号）。读操作可以直接在非领导者节点上进行，但它们会等待非Leader FE节点的状态与最后一次操作的LSN同步。观察者节点可以提高FE集群处理读操作的能力。对于不急于获得数据的用户，可以从观察者节点读取数据。

## 部署FE HA集群

要部署具有高可用性的FE集群，必须满足以下条件：

- FE节点之间的时钟差异不能超过5秒。使用NTP协议同步时间。
- 每台机器只能部署一个FE节点。
- 所有FE节点必须配置相同的HTTP端口。

满足上述条件后，您可以按照以下步骤逐一添加FE实例到StarRocks集群中，以实现FE节点的高可用部署。

1. 分发二进制文件和配置文件（与单实例部署相同）。
2. 将MySQL客户端连接到现有FE，并添加新实例的信息，包括角色、IP和端口：

   ```sql
   mysql> ALTER SYSTEM ADD FOLLOWER "host:port";
   ```
   或者

   ```sql
   mysql> ALTER SYSTEM ADD OBSERVER "host:port";
   ```
   主机是机器的IP地址。如果机器有多个IP地址，请选择priority_networks中指定的IP地址。例如，可以设置priority_networks=192.168.1.0/24，以使用192.168.1.x子网进行通信。端口是edit_log_port，其默认值为9010。

      > 注意：出于安全考虑，StarRocks的FE和BE只能监听一个IP地址进行通信。如果一台机器有多个网卡，StarRocks可能无法自动找到正确的IP地址。例如，运行`ifconfig`命令发现`eth0 IP`是`192.168.1.1`，`docker0 : 172.17.0.1`。我们可以设置word network `192.168.1.0/24`，以指定eth0作为通信IP地址。这里我们使用[CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing)表示法来指定IP地址所在的子网范围，以便在所有BE和FE上使用。`priority_networks`应在`fe.conf`和`be.conf`中设置。该属性指示FE或BE启动时使用哪个IP地址。示例：`priority_networks=10.1.3.0/24`。
   如果出现错误，可以使用以下命令删除FE节点：

   ```sql
   alter system drop follower "fe_host:edit_log_port";
   alter system drop observer "fe_host:edit_log_port";
   ```

3. FE节点需要互连以完成领导者选举、投票、日志提交和复制。当FE节点初次启动时，需要指定现有集群中的一个节点作为辅助节点。辅助节点将获取集群中所有FE节点的配置信息以建立连接。因此，在初始化时，需要指定--helper参数：

   ```shell
   ./bin/start_fe.sh --helper host:port --daemon
   ```

   主机是辅助节点的IP地址。如果存在多个IP地址，请选择priority_networks中指定的IP地址。端口是edit_log_port，默认值为9010。

   未来的启动不需要指定--helper参数。FE会将其他FE的配置信息存储在本地目录中。可以直接启动：

   ```shell
   ./bin/start_fe.sh --daemon
   ```

4. 检查集群状态，确认部署是否成功。

```Plain
mysql> SHOW PROC '/frontends'\G

***1\. row***

Name: 172.26.x.x_9010_1584965098874

IP: 172.26.x.x

HostName: sr-test1

......

Role: FOLLOWER

IsMaster: true

......

Alive: true

......

***2\. row***

Name: 172.26.x.x_9010_1584965098874

IP: 172.26.x.x

HostName: sr-test2

......

Role: FOLLOWER

IsMaster: false

......

Alive: true

......

***3\. row***

Name: 172.26.x.x_9010_1584965098874

IP: 172.26.x.x

HostName: sr-test3

......

Role: FOLLOWER

IsMaster: false

......

Alive: true

......

3 rows in set (0.05 sec)
```

当Alive为true时，表示节点已成功加入到集群中。在上述示例中，172.26.x.x_9010_1584965098874是Leader FE节点。
