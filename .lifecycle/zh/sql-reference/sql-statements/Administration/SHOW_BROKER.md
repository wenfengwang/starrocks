---
displayed_sidebar: English
---

# 查看经纪人信息

## 描述

此语句用于查看当前存在的经纪人信息。

:::提示

只有拥有系统级别的操作权限或 cluster_admin 角色的用户才能执行此操作。

:::

## 语法

```sql
SHOW BROKER
```

注意：

1. LastStartTime 表示最近一次BE（后端）启动的时间。
2. LastHeartbeat 代表最近一次心跳的时间。
3. Alive 表示节点是否存活。
4. ErrMsg 用于当心跳检测失败时显示错误信息。
