---
displayed_sidebar: English
---

# 显示前端

## 描述

此语句用于查看 FE 节点。

:::提示

只有具有系统级 OPERATE 权限或 `cluster_admin` 角色的用户才能执行此操作。

:::

## 语法

```sql
SHOW FRONTENDS
```

注意：

1. Name 表示 BDBJE 中 FE 节点的名称。
2. 如果 Join 为 true，则表示此节点已加入群集。但这并不意味着该节点仍在集群中，因为它可能已丢失。
3. Alive 表示节点是否存活。
4. ReplayedJournalId 表示节点当前已重播的最大元数据日志 ID。
5. LastHeartbeat 是最新的心跳信号。
6. IsHelper 表示节点是否为 BDBJE 中的辅助节点。
7. ErrMsg 用于在心跳信号失败时显示错误消息。
