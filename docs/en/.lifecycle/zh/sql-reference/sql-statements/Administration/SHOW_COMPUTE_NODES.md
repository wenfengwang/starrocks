---
displayed_sidebar: English
---

# 显示计算节点

## 描述

查看您的 StarRocks 集群中的所有计算节点。

:::提示

只有具有系统级 OPERATE 权限或 `cluster_admin` 角色的用户才能执行此操作。

:::

## 语法

```SQL
SHOW COMPUTE NODES
```

## 输出

下表描述了该语句返回的参数。

| **参数**        | **描述**                                              |
| -------------------- | ------------------------------------------------------------ |
| LastStartTime        | 计算节点上次启动的时间。                   |
| LastHeartbeat        | 计算节点上次发送心跳的时间。        |
| Alive                | 计算节点是否可用。                     |
| SystemDecommissioned | 如果该参数的值为 `true`，则表示计算节点已从您的 StarRocks 集群中移除。在移除之前，将克隆计算节点中的数据。 |
| ErrMsg               | 计算节点发送心跳失败时的错误消息。  |
| Status               | 计算节点的状态，以 JSON 格式显示。目前，您只能看到计算节点上次发送状态的时间。 |
