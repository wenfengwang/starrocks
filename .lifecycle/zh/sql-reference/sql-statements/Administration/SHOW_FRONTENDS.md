---
displayed_sidebar: English
---

# 查看前端节点

## 描述

此语句用于查看前端（FE）节点的信息。

:::提示

只有拥有系统级别的OPERATE权限或cluster_admin角色的用户才能进行此操作。

:::

## 语法说明

```sql
SHOW FRONTENDS
```

注意：

1. Name代表BDBJE中前端节点的名称。
2. Join为true意味着该节点已经加入了集群。但这并不表示该节点仍然在集群中，因为可能已经失联。
3. Alive指的是节点是否处于活跃状态。
4. ReplayedJournalId代表该节点已重放的最大元数据日志ID。
5. LastHeartbeat是节点的最后一次心跳时间。
6. IsHelper表示该节点是否是BDBJE中的辅助节点。
7. ErrMsg用来在心跳检测失败时显示错误信息。
