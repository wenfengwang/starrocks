---
displayed_sidebar: "Chinese"
---

# 显示前端

## 描述

此语句用于查看FE节点。

语法：

```sql
SHOW FRONTENDS
```

注意：

1. Name表示BDBJE中FE节点的名称。
2. Join为true意味着该节点已经加入集群。但这并不意味着该节点仍然在集群中，因为它可能已丢失。
3. Alive表示节点是否存活。
4. ReplayedJournalId表示节点当前已经重放的最大元数据日志ID。
5. LastHeartbeat是最新的心跳。
6. IsHelper表示节点是否是BDBJE的辅助节点。
7. ErrMsg用于在心跳失败时显示错误消息。