---
displayed_sidebar: English
---

# 修改系统

## 描述

管理集群中的FE（前端）、BE（后端）、CN（计算节点）、Broker节点以及元数据快照。

> **注意**
> 只有`cluster_admin`角色有权限执行此操作。

## 语法和参数

### FE（前端）

- 添加一个Follower FE（追随者前端）。

  ```SQL
  ALTER SYSTEM ADD FOLLOWER "<fe_host>:<edit_log_port>"[, ...]
  ```

  您可以通过执行SHOW PROC '/frontends'\G来检查新的Follower FE的状态。

- 删除一个Follower FE（追随者前端）。

  ```SQL
  ALTER SYSTEM DROP FOLLOWER "<fe_host>:<edit_log_port>"[, ...]
  ```

- 添加一个Observer FE（观察者前端）。

  ```SQL
  ALTER SYSTEM ADD OBSERVER "<fe_host>:<edit_log_port>"[, ...]
  ```

  您可以通过执行SHOW PROC '/frontends'\G来检查新的Observer FE的状态。

- 删除一个Observer FE（观察者前端）。

  ```SQL
  ALTER SYSTEM DROP OBSERVER "<fe_host>:<edit_log_port>"[, ...]
  ```

|参数|必填|说明|
|---|---|---|
|fe_host|是|FE实例的主机名或IP地址。如果您的实例有多个IP地址，请使用配置项priority_networks的值。|
|edit_log_port|是|FE节点的BDB JE通信端口。默认值：9010。|

### BE（后端）

- 添加一个BE节点。

  ```SQL
  ALTER SYSTEM ADD BACKEND "<be_host>:<heartbeat_service_port>"[, ...]
  ```

  您可以通过执行[SHOW BACKENDS](../Administration/SHOW_BACKENDS.md)来检查新BE节点的状态。

- 删除一个BE节点。

    > **注意**
    > 您不能删除存储单副本表格的BE节点。

  ```SQL
  ALTER SYSTEM DROP BACKEND "<be_host>:<heartbeat_service_port>"[, ...]
  ```

- 下线一个BE节点。

  ```SQL
  ALTER SYSTEM DECOMMISSION BACKEND "<be_host>:<heartbeat_service_port>"[, ...]
  ```

  与强制从集群中删除**BE**节点不同，下线**BE**节点是一种安全移除的方式。这是一个异步操作。下线**BE**节点时，首先会将**BE**上的数据迁移到其他**BE**节点，然后才从集群中移除该**BE**节点。在数据迁移期间，数据加载和查询不会受到影响。您可以使用[SHOW BACKENDS](../Administration/SHOW_BACKENDS.md)来检查操作是否成功。如果操作成功，下线的**BE**节点将不会返回。如果操作失败，**BE**节点仍会在线。您可以使用[CANCEL DECOMMISSION](../Administration/CANCEL_DECOMMISSION.md)手动取消操作。

|参数|必填|说明|
|---|---|---|
|be_host|是|BE 实例的主机名或 IP 地址。如果您的实例有多个IP地址，请使用配置项priority_networks的值。|
|heartbeat_service_port|是|BE 心跳服务端口。 BE使用该端口接收来自FE的心跳。默认值：9050。|

### CN（计算节点）

- 添加一个CN节点。

  ```SQL
  ALTER SYSTEM ADD COMPUTE NODE "<cn_host>:<heartbeat_service_port>"[, ...]
  ```

  您可以通过执行[SHOW COMPUTE NODES](../Administration/SHOW_COMPUTE_NODES.md)来检查新CN节点的状态。

- 删除一个CN节点。

  ```SQL
  ALTER SYSTEM DROP COMPUTE NODE "<cn_host>:<heartbeat_service_port>"[, ...]
  ```

> **注意**
> 您不能使用`ALTER SYSTEM DECOMMISSION`命令下线CN节点。

|参数|必填|说明|
|---|---|---|
|cn_host|是|CN 实例的主机名或IP 地址。如果您的实例有多个IP地址，请使用配置项priority_networks的值。|
|heartbeat_service_port|是|CN心跳服务端口。 CN 使用此端口接收来自 FE 的心跳。默认值：9050。|

### Broker

- 添加Broker节点。您可以使用Broker节点从HDFS或云存储中加载数据至StarRocks。更多信息，请参见[从HDFS加载数据](../../../loading/hdfs_load.md)或[从云存储加载数据](../../../loading/cloud_storage_load.md)。

  ```SQL
  ALTER SYSTEM ADD BROKER <broker_name> "<broker_host>:<broker_ipc_port>"[, ...]
  ```

  您可以使用一条SQL命令添加多个Broker节点。每对`\u003cbroker_host\u003e:\u003cbroker_ipc_port\u003e`代表一个Broker节点。它们共享同一个`broker_name`。您可以通过执行[SHOW BROKER](../Administration/SHOW_BROKER.md)来检查新Broker节点的状态。

- 删除Broker节点。

> **警告**
> 删除一个**Broker**节点会终止其上当前正在运行的任务。

-   删除具有相同broker_name的一个或多个Broker节点。

    ```SQL
    ALTER SYSTEM DROP BROKER <broker_name> "<broker_host>:<broker_ipc_port>"[, ...]
    ```

-   删除具有相同broker_name的所有Broker节点。

    ```SQL
    ALTER SYSTEM DROP ALL BROKER <broker_name>
    ```

|参数|必填|说明|
|---|---|---|
|broker_name|是|Broker 节点的名称。多个 Broker 节点可以使用相同的名称。|
|broker_host|是|Broker 实例的主机名或 IP 地址。如果您的实例有多个IP地址，请使用配置项priority_networks的值。|
|broker_ipc_port|是|Broker 节点上的 thrift 服务器端口。 Broker节点用它来接收来自FE或BE的请求。默认值：8000。|

### 创建镜像

创建一个镜像文件。镜像文件是FE元数据的快照。

```SQL
ALTER SYSTEM CREATE IMAGE
```

创建镜像是在Leader FE上进行的异步操作。您可以在FE日志文件**fe.log**中查看操作的开始和结束时间。日志中的`手动触发新的检查点...`表示镜像创建已经开始，`完成保存镜像...`表示镜像已经创建。

## 使用说明

- 添加和删除FE、BE、CN或Broker节点是同步操作。您无法取消删除节点的操作。
- 您不能删除单FE集群中的FE节点。
- 您不能直接删除多FE集群中的Leader FE节点。要删除它，您必须先重启它。在StarRocks选举出新的Leader FE之后，您就可以删除之前的Leader FE节点。
- 如果剩余的BE节点数量少于数据副本的数量，则不能删除BE节点。例如，如果您的集群中有三个BE节点，且数据存储为三副本，则不能删除任何BE节点。如果您有四个BE节点和三个副本，则可以删除一个BE节点。
- 删除和下线BE节点的区别在于，删除BE节点时，StarRocks会强制将其从集群中移除，并在移除后补足丢失的数据片段；而下线BE节点时，StarRocks会先将其上的数据片段迁移到其他节点，然后再移除该节点。

## 示例

示例1：添加一个Follower FE节点。

```SQL
ALTER SYSTEM ADD FOLLOWER "x.x.x.x:9010";
```

示例2：同时删除两个Observer FE节点。

```SQL
ALTER SYSTEM DROP OBSERVER "x.x.x.x:9010","x.x.x.x:9010";
```

示例3：添加一个BE节点。

```SQL
ALTER SYSTEM ADD BACKEND "x.x.x.x:9050";
```

示例4：同时删除两个BE节点。

```SQL
ALTER SYSTEM DROP BACKEND "x.x.x.x:9050", "x.x.x.x:9050";
```

示例5：同时下线两个BE节点。

```SQL
ALTER SYSTEM DECOMMISSION BACKEND "x.x.x.x:9050", "x.x.x.x:9050";
```

示例6：添加两个使用相同broker_name - hdfs的Broker节点。

```SQL
ALTER SYSTEM ADD BROKER hdfs "x.x.x.x:8000", "x.x.x.x:8000";
```

示例7：从amazon_s3删除两个Broker节点。

```SQL
ALTER SYSTEM DROP BROKER amazon_s3 "x.x.x.x:8000", "x.x.x.x:8000";
```

示例8：删除amazon_s3中的所有Broker节点。

```SQL
ALTER SYSTEM DROP ALL BROKER amazon_s3;
```
