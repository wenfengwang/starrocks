---
displayed_sidebar: "中文"
---

# ALTER SYSTEM

## 描述

管理簇中的FE、BE、CN、Broker节点和元数据快照。

> **注意**
>
> 只有 `cluster_admin` 角色有权限执行该SQL语句。

## 语法和参数

### FE

- 添加Follower FE。

  ```SQL
  ALTER SYSTEM ADD FOLLOWER "<fe_host>:<edit_log_port>"[, ...]
  ```

  您可以通过执行 `SHOW PROC '/frontends'\G` 来检查新的Follower FE的状态。

- 删除Follower FE。

  ```SQL
  ALTER SYSTEM DROP FOLLOWER "<fe_host>:<edit_log_port>"[, ...]
  ```

- 添加Observer FE。

  ```SQL
  ALTER SYSTEM ADD OBSERVER "<fe_host>:<edit_log_port>"[, ...]
  ```

  您可以通过执行 `SHOW PROC '/frontends'\G` 来检查新的Observer FE的状态。

- 删除Observer FE。

  ```SQL
  ALTER SYSTEM DROP OBSERVER "<fe_host>:<edit_log_port>"[, ...]
  ```

| **参数**          | **必填** | **描述**                                                     |
| ------------------ | -------- | ----------------------------------------------------------- |
| fe_host            | 是       | FE实例的主机名或IP地址。如果您的实例有多个IP地址，请使用配置项 `priority_networks` 的值。 |
| edit_log_port      | 是       | FE节点的BDB JE通信端口。默认值：`9010`。                  |

### BE

- 添加BE节点。

  ```SQL
  ALTER SYSTEM ADD BACKEND "<be_host>:<heartbeat_service_port>"[, ...]
  ```

  您可以通过执行 [SHOW BACKENDS](../Administration/SHOW_BACKENDS.md) 来检查新BE的状态。

- 删除BE节点。

  > **注意**
  >
  > 您不能删除存储单副本表的BE节点。

  ```SQL
  ALTER SYSTEM DROP BACKEND "<be_host>:<heartbeat_service_port>"[, ...]
  ```

- 下线BE节点。

  ```SQL
  ALTER SYSTEM DECOMMISSION BACKEND "<be_host>:<heartbeat_service_port>"[, ...]
  ```

  与强制删除BE节点不同，下线BE节点是一种安全的操作。这是一个异步操作。当BE下线时，BE上的数据首先会迁移到其他BE，然后BE将从簇中移除。数据加载和查询在数据迁移期间不受影响。您可以通过执行 [SHOW BACKENDS](../Administration/SHOW_BACKENDS.md) 来检查操作是否成功。如果操作成功，已下线的BE不会返回。如果操作失败，BE仍然在线。您可以使用 [CANCEL DECOMMISSION](../Administration/CANCEL_DECOMMISSION.md) 手动取消操作。

| **参数**                | **必填** | **描述**                                                     |
| --------------------- | -------- | ----------------------------------------------------------- |
| be_host               | 是       | BE实例的主机名或IP地址。如果您的实例有多个IP地址，请使用配置项 `priority_networks` 的值。|
| heartbeat_service_port | 是       | BE心跳服务端口。BE使用此端口从FE接收心跳。默认值：`9050`。|

### CN

- 添加CN节点。

  ```SQL
  ALTER SYSTEM ADD COMPUTE NODE "<cn_host>:<heartbeat_service_port>"[, ...]
  ```

  您可以通过执行 [SHOW COMPUTE NODES](../Administration/SHOW_COMPUTE_NODES.md) 来检查新CN的状态。

- 删除CN节点。

  ```SQL
  ALTER SYSTEM DROP COMPUTE NODE "<cn_host>:<heartbeat_service_port>"[, ...]
  ```

> **注意**
>
> 您不能使用 `ALTER SYSTEM DECOMMISSION` 命令下线CN节点。

| **参数**                | **必填** | **描述**                                                     |
| --------------------- | -------- | ----------------------------------------------------------- |
| cn_host               | 是       | CN实例的主机名或IP地址。如果您的实例有多个IP地址，请使用配置项 `priority_networks` 的值。|
| heartbeat_service_port | 是       | CN心跳服务端口。CN使用此端口从FE接收心跳。默认值：`9050`。|

### Broker

- 添加Broker节点。您可以使用Broker节点从HDFS或云存储加载数据到StarRocks。有关更多信息，请参阅 [从HDFS加载数据](../../../loading/hdfs_load.md) 或 [从云存储加载数据](../../../loading/cloud_storage_load.md)。

  ```SQL
  ALTER SYSTEM ADD BROKER <broker_name> "<broker_host>:<broker_ipc_port>"[, ...]
  ```

  您可以使用一个SQL添加多个Broker节点。每个 `<broker_host>:<broker_ipc_port>` 对应一个Broker节点。它们共享一个公共的 `broker_name`。您可以通过执行 [SHOW BROKER](../Administration/SHOW_BROKER.md) 来检查新的Broker节点的状态。

- 删除Broker节点。

> **注意**
>
> 删除Broker节点会终止当前正在其上运行的任务。

  - 删除使用相同的 `broker_name` 的一个或多个Broker节点。

    ```SQL
    ALTER SYSTEM DROP BROKER <broker_name> "<broker_host>:<broker_ipc_port>"[, ...]
    ```

  - 删除使用相同的 `broker_name` 的所有Broker节点。

    ```SQL
    ALTER SYSTEM DROP ALL BROKER <broker_name>
    ```

| **参数**          | **必填** | **描述**                                                              |
| ------------------ | -------- | -------------------------------------------------------------------- |
| broker_name        | 是       | Broker节点的名称。多个Broker节点可以共享相同的名称。                 |
| broker_host        | 是       | Broker实例的主机名或IP地址。如果您的实例有多个IP地址，请使用配置项 `priority_networks` 的值。|
| broker_ipc_port    | 是       | Broker节点上的thrift服务器端口。Broker节点用它来接收来自FE或BE的请求。默认值：`8000`。|

### 创建镜像

创建一个镜像文件。镜像文件是FE元数据的快照。

```SQL
ALTER SYSTEM CREATE IMAGE
```

创建镜像是在Leader FE上的异步操作。您可以在FE日志文件 **fe.log** 中检查操作的开始时间和结束时间。类似 `triggering a new checkpoint manually...` 的日志表示镜像创建已经开始，类似 `finished save image...` 的日志表示镜像已被创建。

## 使用注意事项

- 添加和删除FE、BE、CN或Broker节点都是同步操作。您无法取消删除节点的操作。
- 在单FE簇中无法删除FE节点。
- 在多FE簇中无法直接删除Leader FE节点。要删除它，您首先必须重新启动它。当StarRocks选举出新的Leader FE后，您可以删除之前的FE节点。
- 如果剩余BE节点的数量小于数据副本的数量，则无法删除BE节点。例如，如果您的簇中有三个BE节点，并且您将数据存储为三个副本，您就无法删除任何BE节点。如果您有四个BE节点和三个副本，那么您可以删除一个BE节点。
- 删除和下线BE节点的区别在于，当您删除BE节点时，StarRocks会强制将其从簇中移除，并在移除后补充已经移除的tablet，而当您下线BE节点时，StarRocks首先将下线节点上的tablet迁移到其他节点，然后再删除该节点。

## 示例

示例 1：添加一个Follower FE节点。

```SQL
ALTER SYSTEM ADD FOLLOWER "x.x.x.x:9010";
```

示例 2：同时删除两个Observer FE节点。

```SQL
ALTER SYSTEM DROP OBSERVER "x.x.x.x:9010","x.x.x.x:9010";
```

示例 3：添加一个BE节点。

```SQL
ALTER SYSTEM ADD BACKEND "x.x.x.x:9050";
```

示例 4：同时删除两个BE节点。

```SQL
ALTER SYSTEM DROP BACKEND "x.x.x.x:9050", "x.x.x.x:9050";
```

示例 5：同时下线两个BE节点。

```SQL
ALTER SYSTEM DECOMMISSION BACKEND "x.x.x.x:9050", "x.x.x.x:9050";
```

示例 6：添加两个使用相同的 `broker_name` - `hdfs` 的Broker节点。

```SQL
ALTER SYSTEM ADD BROKER hdfs "x.x.x.x:8000", "x.x.x.x:8000";
```

示例 7：从 `amazon_s3` 删除两个Broker节点。

```SQL
ALTER SYSTEM DROP BROKER amazon_s3 "x.x.x.x:8000", "x.x.x.x:8000";
```

示例 8：删除 `amazon_s3` 中的所有Broker节点。

```SQL
ALTER SYSTEM DROP ALL BROKER amazon_s3;
```