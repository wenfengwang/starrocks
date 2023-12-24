---
displayed_sidebar: English
---

# 部署具有高可用性的 FE 集群

本主题介绍 StarRocks 的 FE 节点的高可用性（HA）部署。

## 了解 FE HA 集群

StarRocks 的 FE 高可用性集群采用主从复制架构，以避免单点故障。FE 使用类似于 raft 的 BDB JE 协议来实现领导者选择、日志复制和故障切换。

### 了解 FE 角色

在一个 FE 集群中，节点被划分为以下两种角色：

- **FE Follower**

FE Follower 是复制协议的投票成员，参与领导者 FE 的选举并提交日志。FE Follower 的数量是奇数（2n+1）。需要大多数（n+1）来确认，并容忍少数（n）的故障。

- **FE Observer**

FE Observer 是非投票成员，用于异步订阅复制日志。在集群中，FE Observer 的状态落后于 Follower，类似于其他复制协议中的追随者角色。

FE 集群会自动从 FE Follower 中选择领导者 FE。领导者 FE 执行所有状态更改。更改可以由非领导者 FE 节点发起，然后转发到领导者 FE 执行。非领导者 FE 节点记录在复制日志中最近更改的 LSN。读操作可以直接在非领导者节点上执行，但它们会等待直到非领导者 FE 节点的状态与上次操作的 LSN同步。观察者节点可以增加 FE 集群上读操作的负载能力。对于没有紧急需求的用户，可以读取观察者节点。

## 部署 FE HA 集群

要部署具有高可用性的 FE 集群，必须满足以下要求：

- FE 节点之间的时钟差不应超过 5 秒。使用 NTP 协议来校准时间。
- 每台机器上只能部署一个 FE 节点。
- 必须在所有 FE 节点上分配相同的 HTTP 端口。

当满足上述所有要求后，您可以按照以下步骤逐个将 FE 实例添加到 StarRocks 集群中，以实现 FE 节点的高可用性部署。

1. 分发二进制文件和配置文件（与单个实例相同）。
2. 将 MySQL 客户端连接到现有的 FE，并添加新实例的信息，包括角色、IP、端口：

   ```sql
   mysql> ALTER SYSTEM ADD FOLLOWER "host:port";
   ```

   或者

   ```sql
   mysql> ALTER SYSTEM ADD OBSERVER "host:port";
   ```

   主机是机器的 IP。如果机器有多个 IP，请在 priority_networks 中选择 IP。例如，`priority_networks=192.168.1.0/24` 可以设置为使用子网 `192.168.1.x` 进行通信。端口是 `edit_log_port`，默认为 `9010`。

   > 注意：出于安全考虑，StarRocks 的 FE 和 BE 只能监听一个 IP 用于通信。如果一台机器有多个网卡，StarRocks 可能无法自动找到正确的 IP。例如，运行 `ifconfig` 命令以获取 `eth0 IP`，`192.168.1.1`，`docker0 : 172.17.0.1`。我们可以设置 `priority_networks=192.168.1.0/24` 来指定 eth0 作为通信 IP。这里我们使用 [CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) 表示法来指定 IP 所在的子网范围，以便它可以在所有 BE 和 FE 上使用。`priority_networks` 在 `fe.conf` 和 `be.conf` 中都要进行配置。该属性指示启动 FE 或 BE 时要使用的 IP。示例如下：`priority_networks=10.1.3.0/24`。

   如果出现错误，请使用以下命令删除 FE：

   ```sql
   alter system drop follower "fe_host:edit_log_port";
   alter system drop observer "fe_host:edit_log_port";
   ```

3. FE 节点需要成对互联，以完成主节点的选择、投票、日志提交和复制。首次启动 FE 节点时，需要指定现有集群中的某个节点作为辅助节点。辅助节点获取集群中所有 FE 节点的配置信息，建立连接。因此，在启动期间，请指定 `--helper` 参数：

   ```shell
   ./bin/start_fe.sh --helper host:port --daemon
   ```

   主机是辅助节点的 IP。如果有多个 IP，请在 `priority_networks` 中选择 IP。端口是 `edit_log_port`，默认为 `9010`。

   未来启动时，无需指定 `--helper` 参数。FE 将其他 FE 的配置信息存储在本地目录中。要直接启动：

   ```shell
   ./bin/start_fe.sh --daemon
   ```

4. 检查集群状态并确认部署成功。

  ```Plain Text
  mysql> SHOW PROC '/frontends'\G

  ***1\. 行***

  Name: 172.26.x.x_9010_1584965098874

  IP: 172.26.x.x

  HostName: sr-test1

  ......

  Role: FOLLOWER

  IsMaster: true

  ......

  Alive: true

  ......

  ***2\. 行***

  Name: 172.26.x.x_9010_1584965098874

  IP: 172.26.x.x

  HostName: sr-test2

  ......

  Role: FOLLOWER

  IsMaster: false

  ......

  Alive: true

  ......

  ***3\. 行***

  Name: 172.26.x.x_9010_1584965098874

  IP: 172.26.x.x

  HostName: sr-test3

  ......

  Role: FOLLOWER

  IsMaster: false

  ......

  Alive: true

  ......

  3 行结果 (0.05 秒)
  ```

当 `Alive` 为 `true` 时，节点已成功添加到集群中。在上面的示例中，`172.26.x.x_9010_1584965098874` 是领导者 FE 节点。
