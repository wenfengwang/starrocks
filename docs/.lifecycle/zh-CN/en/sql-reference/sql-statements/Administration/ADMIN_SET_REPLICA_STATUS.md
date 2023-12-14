---
displayed_sidebar: "Chinese"
---

# ADMIN SET REPLICA STATUS

## 描述

此语句用于设置指定副本的状态。

当前此命令仅用于手动设置一些副本的状态为BAD或OK，使系统能够自动修复这些副本。

语法：

```sql
ADMIN SET REPLICA STATUS
PROPERTIES ("key" = "value", ...)
```

当前支持以下属性：

"table_id": 必填。指定Tablet Id。

"backend_id": 必填。指定Backend Id。

"status": 必填。指定状态。当前仅支持"bad"和"ok"。

如果指定的副本不存在或其状态为bad，则将忽略该副本。

注意：

设置为Bad状态的副本可能会立即被删除，请谨慎操作。

## 示例

1. 将BE 10001上Tablet 10003的副本状态设置为bad。

    ```sql
    ADMIN SET REPLICA STATUS PROPERTIES("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");
    ```

2. 将BE 10001上Tablet 10003的副本状态设置为ok。

    ```sql
    ADMIN SET REPLICA STATUS PROPERTIES("tablet_id" = "10003", "backend_id" = "10001", "status" = "ok");
    ```