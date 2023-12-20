---
displayed_sidebar: English
---

# 显示BROKER信息

## 描述

此语句用于查看当前存在的broker。

:::tip

只有具有SYSTEM级别的OPERATE权限或`cluster_admin`角色的用户才能执行此操作。

:::

## 语法

```sql
SHOW BROKER
```

注意：

1. LastStartTime 表示最近一次BE启动的时间。
2. LastHeartbeat 表示最近一次的心跳。
3. Alive指示节点是否存活。
4. ErrMsg 用于显示心跳失败时的错误信息。