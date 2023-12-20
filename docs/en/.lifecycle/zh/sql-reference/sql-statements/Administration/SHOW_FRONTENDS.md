---
displayed_sidebar: English
---

# 显示前端

## 描述

此语句用于查看 FE 节点。

:::tip

只有具有 SYSTEM 级 OPERATE 权限或 `cluster_admin` 角色的用户才能执行此操作。

:::

## 语法

```sql
SHOW FRONTENDS
```

注意：

1. Name 表示 BDBJE 中 FE 节点的名称。
2. Join 为 true 表示该节点已经加入集群。但这并不意味着节点仍然在集群中，因为它可能已经失联。
3. Alive 指示节点是否存活。
4. ReplayedJournalId 代表该节点当前已重放的最大元数据日志 ID。
5. LastHeartbeat 是最近一次的心跳时间。
6. IsHelper 表示该节点是否是 BDBJE 的辅助节点。
7. ErrMsg 用来在心跳检测失败时显示错误信息。