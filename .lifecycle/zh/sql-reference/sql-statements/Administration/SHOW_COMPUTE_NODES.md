---
displayed_sidebar: English
---

# 显示计算节点

## 描述

查看您的 StarRocks 集群中的所有计算节点。

:::提示

只有拥有 SYSTEM 级别的 OPERATE 权限或 cluster_admin 角色的用户才能执行此操作。

:::

## 语法

```SQL
SHOW COMPUTE NODES
```

## 输出

以下表格描述了该语句返回的参数。

|参数|说明|
|---|---|
|LastStartTime|计算节点上次启动时间。|
|LastHeartbeat|计算节点最后一次发送心跳的时间。|
|Alive|计算节点是否可用。|
|SystemDecommissioned|如果该参数的值为 true，则计算节点将从 StarRocks 集群中删除。删除之前，计算节点中的数据已被克隆。|
|ErrMsg|计算节点发送心跳失败的错误信息。|
|状态|计算节点的状态，以 JSON 格式显示。目前，您只能看到计算节点最后一次发送其状态的时间。|
