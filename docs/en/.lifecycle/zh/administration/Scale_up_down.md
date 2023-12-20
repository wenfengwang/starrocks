---
displayed_sidebar: English
---

# 横向扩展和纵向缩容

本主题介绍如何对 StarRocks 的节点进行横向扩展和纵向缩容。

## 扩展和缩容 FE 节点

StarRocks 有两种类型的 FE 节点：Follower 和 Observer。Follower 节点参与选举投票和数据写入。Observer 节点仅用于同步日志和提升读取性能。

* Follower FE 的数量（包括 Leader）必须为奇数，推荐部署 3 个以形成高可用（HA）模式。
* 当 FE 处于高可用部署状态（1 个 Leader，2 个 Follower）时，建议添加 Observer FE 以提高读取性能。* 通常情况下，一个 FE 节点可以支持 10-20 个 BE 节点。建议 FE 节点的总数不超过 10 个。在大多数情况下，3 个就足够了。

### 扩展 FE 节点

部署 FE 节点并启动服务后，运行以下命令来扩展 FE 节点。

```sql
alter system add follower "fe_host:edit_log_port";
alter system add observer "fe_host:edit_log_port";
```

### 缩容 FE 节点

FE 缩容与扩展类似。运行以下命令来缩容 FE 节点。

```sql
alter system drop follower "fe_host:edit_log_port";
alter system drop observer "fe_host:edit_log_port";
```

扩展和缩容后，可以通过运行 `show proc '/frontends';` 来查看节点信息。

## 扩展和缩容 BE 节点

BE 节点扩容或缩容后，StarRocks 会自动进行负载均衡，不会影响整体性能。

### 扩展 BE 节点

运行以下命令来扩展 BE 节点。

```sql
alter system add backend 'be_host:be_heartbeat_service_port';
```

运行以下命令来检查 BE 节点的状态。

```sql
show proc '/backends';
```

### 缩容 BE 节点

缩容 BE 节点有两种方式：`DROP` 和 `DECOMMISSION`。

`DROP` 会立即删除 BE 节点，丢失的副本将由 FE 调度补齐。`DECOMMISSION` 会确保先补齐副本，然后再删除 BE 节点。`DECOMMISSION` 更为温和，推荐用于 BE 节点缩容。

两种方法的命令类似：

* `alter system decommission backend "be_host:be_heartbeat_service_port";`
* `alter system drop backend "be_host:be_heartbeat_service_port";`

删除 backend 是一个高风险操作，因此在执行前需要仔细确认

* `alter system drop backend "be_host:be_heartbeat_service_port";`