---
displayed_sidebar: English
---

# 取消退役

## 描述

此语句用于撤销节点的退役操作。

:::tip

只有 `cluster_admin` 角色拥有执行此操作的权限。您可以按照 [GRANT](../account-management/GRANT.md) 中的指引来授予该权限。

:::

语法：

```sql
CANCEL DECOMMISSION BACKEND "<host>:<heartbeat_port>"[,"<host>:<heartbeat_port>"...]
```

## 示例

1. 取消两个节点的退役。

   ```sql
   CANCEL DECOMMISSION BACKEND "host1:port", "host2:port";
   ```