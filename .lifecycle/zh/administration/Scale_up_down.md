---
displayed_sidebar: English
---

# 节点扩容与缩容

本节将介绍如何对 StarRocks 的节点进行扩容和缩容操作。

## 扩容与缩容 FE

StarRocks 有两类 FE 节点：Follower 和 Observer。Follower 节点参与选举投票和数据写入。Observer 节点则仅用于同步日志和提高读取性能。

* Follower FE（包括 Leader）的数量必须是奇数，推荐部署 3 个以形成高可用（HA）模式。
* 当 FE 处于高可用部署状态（1 个 Leader，2 个 Follower）时，建议添加 Observer FE 以提升读取性能。* 通常情况下，一个 FE 节点可以支持 10-20 个 BE 节点。建议 FE 节点的总数不超过 10 个，大多数场景下 3 个即可满足需求。

### 扩容 FE

在部署 FE 节点并启动服务后，执行以下命令来进行 FE 扩容。

```sql
alter system add follower "fe_host:edit_log_port";
alter system add observer "fe_host:edit_log_port";
```

### 缩容 FE

FE 缩容操作与扩容类似。执行以下命令来进行 FE 缩容。

```sql
alter system drop follower "fe_host:edit_log_port";
alter system drop observer "fe_host:edit_log_port";
```

扩容或缩容后，可以通过执行 show proc '/frontends'; 命令查看节点信息。

## 扩容与缩容 BE

对 BE 进行扩容或缩容后，StarRocks 将自动执行负载均衡，不会影响整体性能。

### 扩容 BE

执行以下命令来扩容 BE。

```sql
alter system add backend 'be_host:be_heartbeat_service_port';
```

执行以下命令来检查 BE 状态。

```sql
show proc '/backends';
```

### 缩容 BE

缩容 BE 节点有两种方式：DROP 和 DECOMMISSION。

DROP 将立即删除 BE 节点，丢失的副本将由 FE 调度补全。DECOMMISSION 将确保先补全副本，然后再删除 BE 节点。DECOMMISSION 方法更为友好，因此推荐用于 BE 缩容。

两种方法的命令相似：

* alter system decommission backend "be_host:be_heartbeat_service_port";
* alter system drop backend "be_host:be_heartbeat_service_port";

直接删除后端节点是一个高风险操作，在执行前需要进行二次确认：

* alter system drop backend "be_host:be_heartbeat_service_port";
