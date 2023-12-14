---
displayed_sidebar: "中文"
---

# 伸缩In和Out

本主题描述了如何调整StarRocks的节点的伸缩大小。

## 伸缩FE In和Out

StarRocks有两种类型的FE节点：Follower和Observer。Follower参与选举投票和写入。Observer仅用于同步日志和扩展读性能。

> * Follower FE的数量（包括领导者）必须为奇数，并建议部署3个以形成高可用（HA）模式。
> * 在FE处于高可用部署（1个领导者，2个跟随者）时，建议添加Observer FE以获得更好的读性能。* 通常一个FE节点可以与10-20个BE节点配合使用。建议FE节点的总数小于10。在大多数情况下，3个足够了。

### 伸缩FE Out

部署FE节点并启动服务后，运行以下命令伸缩FE Out。

~~~sql
alter system add follower "fe_host:edit_log_port";
alter system add observer "fe_host:edit_log_port";
~~~

### 伸缩FE In

FE的缩小操作类似于扩大操作。运行以下命令进行伸缩FE In。

~~~sql
alter system drop follower "fe_host:edit_log_port";
alter system drop observer "fe_host:edit_log_port";
~~~

扩张和收缩之后，您可以运行 `show proc '/frontends';` 查看节点信息。

## 伸缩BE In和Out

在BE缩小或扩大后，StarRocks将自动进行负载平衡，不会影响整体性能。

### 伸缩BE Out

运行以下命令进行BE扩展。

~~~sql
alter system add backend 'be_host:be_heartbeat_service_port';
~~~

运行以下命令检查BE状态。

~~~sql
show proc '/backends';
~~~

### 伸缩BE In

缩小BE节点有两种方式 - `DROP` 和 `DECOMMISSION`。

`DROP` 将立即删除BE节点，并由FE调度补充丢失的副本。`DECOMMISSION` 将确保首先补充副本，然后再删除BE节点。`DECOMMISSION` 更友好，建议用于BE伸缩In。

两种方法的命令类似：

* `alter system decommission backend "be_host:be_heartbeat_service_port";`
* `alter system drop backend "be_host:be_heartbeat_service_port";`

删除后端是危险的操作，因此在执行之前需要确认两次。

* `alter system drop backend "be_host:be_heartbeat_service_port";`