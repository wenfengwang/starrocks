---
displayed_sidebar: English
---

# 扩展和缩减

本主题描述了如何扩展和缩减 StarRocks 的节点。

## 扩展 FE

StarRocks 有两种类型的 FE 节点：Follower 和 Observer。追随者参与选举投票和写入。观察者仅用于同步日志和提高读取性能。

> * follower FE（包括 leader）的数量必须为奇数，建议部署 3 个以形成高可用（HA）模式。
> * 当 FE 处于高可用部署（1 个 leader、2 个 follower）时，建议添加 Observer FE 以获得更好的读取性能。* 通常一个 FE 节点可以与 10-20 个 BE 节点一起工作。建议 FE 节点的总数少于 10 个。在大多数情况下，三个就足够了。

### 扩展 FE

部署 FE 节点并启动服务后，运行以下命令进行 FE 扩展。

~~~sql
alter system add follower "fe_host:edit_log_port";
alter system add observer "fe_host:edit_log_port";
~~~

### 缩减 FE

部署 FE 节点并启动服务后，运行以下命令进行 FE 缩减。

~~~sql
alter system drop follower "fe_host:edit_log_port";
alter system drop observer "fe_host:edit_log_port";
~~~

扩展和缩减后，您可以通过运行 `show proc '/frontends';` 查看节点信息。

## 扩展和缩减 BE

在 BE 扩展或缩减后，StarRocks 将自动进行负载平衡，而不会影响整体性能。

### 扩展 BE

运行以下命令进行 BE 扩展。

~~~sql
alter system add backend 'be_host:be_heartbeat_service_port';
~~~

运行以下命令检查 BE 状态。

~~~sql
show proc '/backends';
~~~

### 缩减 BE

有两种方法可以缩减 BE 节点 – `DROP` 和 `DECOMMISSION`。

`DROP` 将立即删除 BE 节点，并丢失的副本将由 FE 进行补偿。 `DECOMMISSION` 将确保首先进行副本补偿，然后再删除 BE 节点。 `DECOMMISSION` 更加友好，推荐用于 BE 缩减。

这两种方法的命令类似：

* `alter system decommission backend "be_host:be_heartbeat_service_port";`
* `alter system drop backend "be_host:be_heartbeat_service_port";`

删除后端是一项危险的操作，因此在执行之前需要确认两次

* `alter system drop backend "be_host:be_heartbeat_service_port";`
