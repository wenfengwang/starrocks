---
displayed_sidebar: English
---

# ALTER SYSTEM

## 描述

管理集群中的 FE、BE、CN、Broker 节点和元数据快照。

> **注意**
> 只有 `cluster_admin` 角色有权限执行此操作。

## 语法和参数

### FE

- 添加一个 Follower FE。

  ```SQL
  ALTER SYSTEM ADD FOLLOWER "<fe_host>:<edit_log_port>"[, ...]
  ```

  你可以通过执行 `SHOW PROC '/frontends'\G` 检查新的 Follower FE 的状态。

- 删除一个 Follower FE。

  ```SQL
  ALTER SYSTEM DROP FOLLOWER "<fe_host>:<edit_log_port>"[, ...]
  ```

- 添加一个 Observer FE。

  ```SQL
  ALTER SYSTEM ADD OBSERVER "<fe_host>:<edit_log_port>"[, ...]
  ```

  你可以通过执行 `SHOW PROC '/frontends'\G` 检查新的 Observer FE 的状态。

- 删除一个 Observer FE。

  ```SQL
  ALTER SYSTEM DROP OBSERVER "<fe_host>:<edit_log_port>"[, ...]
  ```

|**参数**|**必填**|**描述**|
|---|---|---|
|fe_host|是|FE 实例的主机名或 IP 地址。如果你的实例有多个 IP 地址，请使用配置项 `priority_networks` 的值。|
|edit_log_port|是|FE 节点的 BDB JE 通信端口。默认值：`9010`。|

### BE

- 添加一个 BE 节点。

  ```SQL
  ALTER SYSTEM ADD BACKEND "<be_host>:<heartbeat_service_port>"[, ...]
  ```

  你可以通过执行 [SHOW BACKENDS](../Administration/SHOW_BACKENDS.md) 检查新 BE 的状态。

- 删除一个 BE 节点。

    > **注意**
    > 你不能删除存储单副本表的 Tablets 的 BE 节点。

  ```SQL
  ALTER SYSTEM DROP BACKEND "<be_host>:<heartbeat_service_port>"[, ...]
  ```

- 下线一个 BE 节点。

  ```SQL
  ALTER SYSTEM DECOMMISSION BACKEND "<be_host>:<heartbeat_service_port>"[, ...]
  ```

  与强制从集群中移除 BE 节点（删除操作）不同，下线 BE 节点（Decommission）意味着安全地移除。这是一个异步操作。当 BE 节点下线时，首先将该 BE 上的数据迁移到其他 BE，然后再从集群中移除该 BE。数据迁移过程中不会影响数据加载和查询。你可以使用 [SHOW BACKENDS](../Administration/SHOW_BACKENDS.md) 检查操作是否成功。如果操作成功，下线的 BE 将不会返回。如果操作失败，BE 仍然在线。你可以使用 [CANCEL DECOMMISSION](../Administration/CANCEL_DECOMMISSION.md) 手动取消操作。

|**参数**|**必填**|**描述**|
|---|---|---|
|be_host|是|BE 实例的主机名或 IP 地址。如果你的实例有多个 IP 地址，请使用配置项 `priority_networks` 的值。|
|heartbeat_service_port|是|BE 心跳服务端口。BE 使用该端口接收来自 FE 的心跳。默认值：`9050`。|

### CN

- 添加一个 CN 节点。

  ```SQL
  ALTER SYSTEM ADD COMPUTE NODE "<cn_host>:<heartbeat_service_port>"[, ...]
  ```

  你可以通过执行 [SHOW COMPUTE NODES](../Administration/SHOW_COMPUTE_NODES.md) 检查新 CN 的状态。

- 删除一个 CN 节点。

  ```SQL
  ALTER SYSTEM DROP COMPUTE NODE "<cn_host>:<heartbeat_service_port>"[, ...]
  ```

> **注意**
> 你不能使用 `ALTER SYSTEM DECOMMISSION` 命令下线 CN 节点。

|**参数**|**必填**|**描述**|
|---|---|---|
|cn_host|是|CN 实例的主机名或 IP 地址。如果你的实例有多个 IP 地址，请使用配置项 `priority_networks` 的值。|
|heartbeat_service_port|是|CN 心跳服务端口。CN 使用此端口接收来自 FE 的心跳。默认值：`9050`。|

### Broker

- 添加 Broker 节点。你可以使用 Broker 节点将数据从 HDFS 或云存储加载到 StarRocks 中。更多信息，请参阅 [从 HDFS 加载数据](../../../loading/hdfs_load.md) 或 [从云存储加载数据](../../../loading/cloud_storage_load.md)。

  ```SQL
  ALTER SYSTEM ADD BROKER <broker_name> "<broker_host>:<broker_ipc_port>"[, ...]
  ```

  你可以使用一条 SQL 添加多个 Broker 节点。每对 `<broker_host>:<broker_ipc_port>` 表示一个 Broker 节点。它们共享一个 `broker_name`。你可以通过执行 [SHOW BROKER](../Administration/SHOW_BROKER.md) 检查新 Broker 节点的状态。

- 删除 Broker 节点。

> **警告**
> 删除 Broker 节点将终止当前在其上运行的任务。

-   删除一个或多个具有相同 `broker_name` 的 Broker 节点。

    ```SQL
    ALTER SYSTEM DROP BROKER <broker_name> "<broker_host>:<broker_ipc_port>"[, ...]
    ```

-   删除具有相同 `broker_name` 的所有 Broker 节点。

    ```SQL
    ALTER SYSTEM DROP ALL BROKER <broker_name>
    ```

|**参数**|**必填**|**描述**|
|---|---|---|
|broker_name|是|Broker 节点的名称。多个 Broker 节点可以使用相同的名称。|
|broker_host|是|Broker 实例的主机名或 IP 地址。如果你的实例有多个 IP 地址，请使用配置项 `priority_networks` 的值。|
|broker_ipc_port|是|Broker 节点上的 Thrift 服务器端口。Broker 节点使用它来接收来自 FE 或 BE 的请求。默认值：`8000`。|

### 创建镜像

创建一个镜像文件。镜像文件是 FE 元数据的快照。

```SQL
ALTER SYSTEM CREATE IMAGE
```

创建镜像是 Leader FE 上的异步操作。你可以在 FE 日志文件 **fe.log** 中查看操作的开始时间和结束时间。类似 `triggering a new checkpoint manually...` 的日志表明镜像创建已经开始，类似 `finished save image...` 的日志表明镜像已经创建。

## 使用说明

- 添加和删除 FE、BE、CN 或 Broker 节点是同步操作。你无法取消节点删除操作。
- 你无法删除单 FE 集群中的 FE 节点。
- 你不能直接删除多 FE 集群中的 Leader FE 节点。要删除它，你必须首先重启它。StarRocks 选举出新的 Leader FE 后，你可以删除前一个。
- 如果剩余 BE 节点的数量小于数据副本的数量，则无法删除 BE 节点。例如，如果你的集群中有三个 BE 节点，并且你的数据以三个副本存储，则无法删除任何 BE 节点。如果你有四个 BE 节点和三个副本，你可以删除一个 BE 节点。
- 删除 BE 节点和下线 BE 节点的区别在于，删除操作会强制从集群中移除 BE 节点并在移除后补充掉落的 Tablets，而下线操作则会先将退役 BE 节点上的 Tablets 迁移到其他节点，然后移除该节点。

## 示例

示例 1：添加一个 Follower FE 节点。

```SQL
ALTER SYSTEM ADD FOLLOWER "x.x.x.x:9010";
```

示例 2：同时删除两个 Observer FE 节点。

```SQL
ALTER SYSTEM DROP OBSERVER "x.x.x.x:9010", "x.x.x.x:9010";
```

示例 3：添加一个 BE 节点。

```SQL
ALTER SYSTEM ADD BACKEND "x.x.x.x:9050";
```

示例 4：同时删除两个 BE 节点。

```SQL
ALTER SYSTEM DROP BACKEND "x.x.x.x:9050", "x.x.x.x:9050";
```

示例 5：同时下线两个 BE 节点。

```SQL
ALTER SYSTEM DECOMMISSION BACKEND "x.x.x.x:9050", "x.x.x.x:9050";
```

示例 6：添加两个具有相同 `broker_name` - `hdfs` 的 Broker 节点。

```SQL
ALTER SYSTEM ADD BROKER hdfs "x.x.x.x:8000", "x.x.x.x:8000";
```

示例 7：从 `amazon_s3` 中删除两个 Broker 节点。

```SQL
ALTER SYSTEM DROP BROKER amazon_s3 "x.x.x.x:8000", "x.x.x.x:8000";
```

示例 8：删除 `amazon_s3` 中的所有 Broker 节点。

```SQL
ALTER SYSTEM DROP ALL BROKER amazon_s3;
```