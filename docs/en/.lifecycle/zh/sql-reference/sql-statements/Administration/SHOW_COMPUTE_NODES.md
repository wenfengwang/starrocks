---
displayed_sidebar: English
---

# 显示计算节点

## 描述

查看您的 StarRocks 集群中的所有计算节点。

:::tip

只有拥有 SYSTEM 级别的 OPERATE 权限或 `cluster_admin` 角色的用户才能执行此操作。

:::

## 语法

```SQL
SHOW COMPUTE NODES
```

## 输出

下表描述了此语句返回的参数。

|**参数**|**说明**|
|---|---|
|LastStartTime|计算节点最后一次启动的时间。|
|LastHeartbeat|计算节点最后一次发送心跳的时间。|
|Alive|计算节点是否可用。|
|SystemDecommissioned|如果此参数的值为 `true`，则表示计算节点已从您的 StarRocks 集群中移除。移除前，会克隆计算节点中的数据。|
|ErrMsg|计算节点未能发送心跳时的错误信息。|
|Status|计算节点的状态，以 JSON 格式展示。目前，您只能看到计算节点最后一次发送其状态的时间。|