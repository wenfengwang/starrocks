---
displayed_sidebar: English
---

# 撤销退役

## 描述

此语句用于撤销对节点的退役操作。

:::提示

只有拥有 `cluster_admin` 角色的用户才有权限执行此操作。您可以遵循 [GRANT](../account-management/GRANT.md) 中的指引来授予相应的权限。

:::

语法：

```sql
CANCEL DECOMMISSION BACKEND "<host>:<heartbeat_port>"[,"<host>:<heartbeat_port>"...]
```

## 示例

1. 撤销两个节点的退役操作。

   ```sql
   CANCEL DECOMMISSION BACKEND "host1:port", "host2:port";
   ```
