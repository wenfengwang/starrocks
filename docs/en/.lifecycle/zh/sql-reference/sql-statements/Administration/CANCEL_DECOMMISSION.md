---
displayed_sidebar: English
---

# 取消停用

## 描述

该语句用于撤销节点的停用。

:::提示

只有 `cluster_admin` 角色才有执行此操作的权限。您可以按照 [GRANT](../account-management/GRANT.md) 中的说明授予该权限。

:::

语法：

```sql
CANCEL DECOMMISSION BACKEND "<host>:<heartbeat_port>"[,"<host>:<heartbeat_port>"...]
```

## 例子

1. 取消对两个节点的停用。

    ```sql
    CANCEL DECOMMISSION BACKEND "host1:port", "host2:port";
    ```
