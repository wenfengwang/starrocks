---
displayed_sidebar: "Chinese"
---

# 显示计算节点

## 描述

查看你StarRocks集群中的所有计算节点。

## 语法

```SQL
SHOW COMPUTE NODES
```

## 输出

下表描述了该语句返回的参数。

| **参数**              | **描述**                           |
| -------------------- | ----------------------------------- |
| LastStartTime        | 计算节点启动的最后时间。                  |
| LastHeartbeat        | 计算节点发送心跳的最后时间。               |
| Alive                | 计算节点是否可用。                          |
| SystemDecommissioned | 如果参数值为`true`，则计算节点将从StarRocks集群中删除。在删除之前，计算节点中的数据将被克隆。 |
| ErrMsg               | 计算节点未能发送心跳的错误消息。                 |
| Status               | 计算节点的状态，以JSON格式显示。目前，你只能查看计算节点上次发送状态的时间。 |