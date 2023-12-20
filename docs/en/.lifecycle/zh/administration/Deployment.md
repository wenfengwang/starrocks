---
displayed_sidebar: English
---

# 部署具有高可用性的FE集群

本主题介绍 StarRocks FE 节点的高可用性（HA）部署。

## 了解 FE HA 集群

StarRocks 的 FE 高可用集群采用主从复制架构，以避免单点故障。FE 使用类 raft 的 BDB JE 协议实现 leader 选举、日志复制和故障转移。

### 了解 FE 角色

在 FE 集群中，节点分为以下两种角色：

- **FE Follower**

FE Followers 是复制协议的投票成员，参与 Leader FE 的选举并提交日志。FE Followers 的数量是奇数（2n+1）。它需要多数（n+1）来确认，并能容忍少数（n）的故障。

- **FE Observer**

FE Observers 是无投票权的成员，用于异步订阅复制日志。在集群中，FE Observers 的状态落后于 Followers，类似于其他复制协议中的 learner 角色。

FE 集群会自动从 FE Followers 中选举 Leader FE。Leader FE 负责执行所有状态更改。变更可以由非 Leader FE 节点发起，然后转发给 Leader FE 执行。非 Leader FE 节点会在复制日志中记录最新更改的 LSN。读操作可以直接在非 Leader 节点上执行，但它们会等待非 Leader FE 节点的状态与最后一次操作的 LSN 同步。Observer 节点可以提高 FE 集群读操作的负载能力。对于不急迫的读取需求，用户可以读取 Observer 节点。

## 部署 FE HA 集群

要部署具有高可用性的 FE 集群，必须满足以下要求：

- FE 节点之间的时钟差异不应超过 5 秒。使用 NTP 协议校准时间。
- 一台机器上只能部署一个 FE 节点。
- 必须在所有 FE 节点上配置相同的 HTTP 端口。

当满足上述所有要求后，您可以按照以下步骤逐一将 FE 实例添加到 StarRocks 集群中，以实现 FE 节点的 HA 部署。

1. 分发二进制文件和配置文件（与单实例部署相同）。
2. 连接 MySQL 客户端到现有的 FE，并添加新实例的信息，包括角色、IP、端口：

   ```sql
   mysql> ALTER SYSTEM ADD FOLLOWER "host:port";
   ```
   或

   ```sql
   mysql> ALTER SYSTEM ADD OBSERVER "host:port";
   ```
   host 是机器的 IP。如果机器有多个 IP，请选择 `priority_networks` 中的 IP。例如，可以设置 `priority_networks=192.168.1.0/24` 以使用子网 `192.168.1.x` 进行通信。端口是 `edit_log_port`，默认为 `9010`。

      > 注意：出于安全考虑，StarRocks 的 FE 和 BE 只能监听一个 IP 进行通信。如果一台机器有多个网卡，StarRocks 可能无法自动找到正确的 IP。例如，运行 `ifconfig` 命令得知 `eth0 IP` 是 `192.168.1.1`，`docker0` 是 `172.17.0.1`。我们可以设置 `priority_networks=192.168.1.0/24` 来指定 eth0 作为通信 IP。这里我们使用 [CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) 表示法来指定 IP 所在的子网范围，以便在所有 BE 和 FE 上使用。`priority_networks` 需要在 `fe.conf` 和 `be.conf` 中设置。此属性指明 FE 或 BE 启动时应使用的 IP。示例如下：`priority_networks=10.1.3.0/24`。
   如果出现错误，可使用以下命令删除 FE：

   ```sql
   alter system drop follower "fe_host:edit_log_port";
   alter system drop observer "fe_host:edit_log_port";
   ```

3. FE 节点需要成对互联以完成 master 选举、投票、日志提交和复制。当 FE 节点首次启动时，需要指定现有集群中的一个节点作为 helper。helper 节点获取集群中所有 FE 节点的配置信息以建立连接。因此，在启动时，指定 `--helper` 参数：

   ```shell
   ./bin/start_fe.sh --helper host:port --daemon
   ```

   host 是 helper 节点的 IP。如果有多个 IP，选择 `priority_networks` 中的 IP。端口是 `edit_log_port`，默认为 `9010`。

   未来启动时无需指定 `--helper` 参数。FE 会将其他 FE 的配置信息存储在本地目录中。直接启动即可：

   ```shell
   ./bin/start_fe.sh --daemon
   ```

4. 检查集群状态，确认部署成功。

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

共 3 行 (0.05 秒)
```

当 `Alive` 为 `true` 时，表示节点已成功加入集群。在上述示例中，`172.26.x.x_9010_1584965098874` 是 Leader FE 节点。