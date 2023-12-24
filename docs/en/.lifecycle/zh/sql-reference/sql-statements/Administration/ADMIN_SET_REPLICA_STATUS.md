---
displayed_sidebar: English
---

# 管理员设置副本状态

## 描述

此语句用于设置指定副本的状态。

该命令目前仅用于手动将某些副本的状态设置为 BAD 或 OK，以允许系统自动修复这些副本。

:::提示

此操作需要 SYSTEM 级别的 OPERATE 权限。您可以按照 [GRANT](../account-management/GRANT.md) 中的说明授予此权限。

:::

## 语法

```sql
ADMIN SET REPLICA STATUS
PROPERTIES ("key" = "value", ...)
```

目前支持以下属性：

"table_id"：必填。指定平板 ID。

"backend_id"：必填。指定后端 ID。

"status"：必填。指定状态。目前仅支持 "bad" 和 "ok"。

如果指定的副本不存在或其状态为 bad，则将忽略该副本。

注意：

将副本设置为 bad 状态可能会立即删除，请谨慎操作。

## 例子

1. 将平板 10003 上的副本状态设置为 bad，位于 BE 10001 上。

    ```sql
    ADMIN SET REPLICA STATUS PROPERTIES("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");
    ```

2. 将平板 10003 上的副本状态设置为 ok，位于 BE 10001 上。

    ```sql
    ADMIN SET REPLICA STATUS PROPERTIES("tablet_id" = "10003", "backend_id" = "10001", "status" = "ok");
    ```
