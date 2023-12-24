---
displayed_sidebar: English
---

# 显示经纪人

## 描述

此语句用于查看当前存在的经纪人。

:::提示

只有具有系统级 OPERATE 权限或 `cluster_admin` 角色的用户才能执行此操作。

:::

## 语法

```sql
SHOW BROKER
```

注意：

1. LastStartTime 表示最新的经纪人启动时间。
2. LastHeartbeat 表示最新的心跳信号。
3. Alive 表示节点是否存活。
4. ErrMsg 用于在心跳信号失败时显示错误消息。
