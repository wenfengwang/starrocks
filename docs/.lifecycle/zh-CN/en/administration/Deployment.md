---
displayed_sidebar: "Chinese"
---

# 使用高可用性部署FE集群

本主题介绍了StarRocks的FE节点的高可用性(HA)部署。

## 了解FE HA集群

StarRocks的FE高可用性集群使用主从复制架构以避免单点故障。FE使用类似raft的BDB JE协议来实现领导者选择、日志复制和故障切换。

### 了解FE角色

在FE集群中，节点被划分为以下两个角色：

- **FE Follower**

FE Followers是复制协议中有投票权的成员，参与领导者FE的选择并提交日志。FE Followers的数量是奇数(2n+1)。需要多数（n+1）进行确认并容忍少数（n）故障。

- **FE Observer**

FE Observers是无投票权的成员，用于异步订阅复制日志。在集群中，FE Observers的状态落后于Followers的状态，类似于其他复制协议中的leaner角色。

FE集群会自动从FE Followers中选择Leader FE。Leader FE执行所有状态更改。一个更改可以由非Leader FE节点发起，然后转发到Leader FE进行执行。非Leader FE节点记录复制日志中最近更改的LSN。读操作可以直接在非领导者节点上执行，但需要等到非Leader FE节点的状态与最后操作的LSN同步。观察者节点可以增加FE集群上读操作的负载能力。对于没有紧急性的用户来说，可以在观察者节点上进行读取。

## 部署FE HA集群

要部署具有高可用性的FE集群，必须满足以下要求：

- FE节点之间的时钟差异不应超过5秒。使用NTP协议来校准时间。
- 每台机器只能部署一个FE节点。
- 必须在所有FE节点上分配相同的HTTP端口。

当满足上述所有要求时，可以按照以下步骤逐个将FE实例添加到StarRocks集群中，以启用FE节点的HA部署。

1. 分发二进制文件和配置文件（与单个实例相同）。
2. 将MySQL客户端连接到现有的FE，并添加新实例的信息，包括角色、IP、端口：

   ```sql
   mysql> ALTER SYSTEM ADD FOLLOWER "host:port";
   ```

   或者

   ```sql
   mysql> ALTER SYSTEM ADD OBSERVER "host:port";
   ```

   主机是机器的IP。如果机器有多个IP，请选择优先网络中的IP。例如，`priority_networks=192.168.1.0/24` 可以设置为使用子网`192.168.1.x`进行通信。端口是`edit_log_port`，默认为`9010`。

   > 注意：出于安全考虑，StarRocks的FE和BE只能监听一个IP以进行通信。如果一台机器有多张网卡，StarRocks可能无法自动找到正确的IP。例如，运行`ifconfig`命令获取`eth0 IP`为`192.168.1.1`， `docker0: 172.17.0.1`。我们可以设置单词网络`192.168.1.0/24`来指定eth0作为通信IP。在这里，我们使用[CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing)表示法指定IP所在的子网范围，以便它可以在所有BE和FE上使用。`priority_networks` 在`fe.conf`和`be.conf`中都有所写。该属性表示FE或BE启动时使用哪个IP。示例如下：`priority_networks=10.1.3.0/24`。

   如果发生错误，可以使用以下命令删除FE：

   ```sql
   alter system drop follower "fe_host:edit_log_port";
   alter system drop observer "fe_host:edit_log_port";
   ```

3. FE节点需要两两相互连接以完成主节点选择、投票、日志提交和复制。当首次初始化FE节点时，现有集群中的节点需要指定一个帮手。帮手节点获取集群中所有FE节点的配置信息以建立连接。因此，在初始化期间，指定`--helper`参数：

   ```shell
   ./bin/start_fe.sh --helper host:port --daemon
   ```

   主机是帮手节点的IP。如果有多个IP，请选择`priority_networks`中的IP。端口是`edit_log_port`，默认为`9010`。

   未来启动时无需指定`--helper`参数。FE将其他FE的配置信息存储在本地目录中。只需直接启动：

   ```shell
   ./bin/start_fe.sh --daemon
   ```

4. 检查集群状态，确认部署成功。

  ```Plain Text
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

当`Alive`为`true`时，节点成功添加到集群中。在上面的示例中，`172.26.x.x_9010_1584965098874`是Leader FE节点。