---
displayed_sidebar: English
---

# 管理员设置副本状态

## 描述

此语句用于设置指定副本的状态。

该命令目前仅用于手动将某些副本的状态设置为BAD或OK，以便系统自动修复这些副本。

:::tip

此操作需要 SYSTEM 级别的 OPERATE 权限。您可以按照 [GRANT](../account-management/GRANT.md) 中的说明来授予此权限。

:::

## 语法

```sql
ADMIN SET REPLICA STATUS
PROPERTIES ("key" = "value", ...)
```

目前支持以下属性：

"tablet_id"：必填。指定 Tablet Id。

"backend_id"：必填。指定 Backend Id。

"status"：必填。指定状态。目前仅支持 "bad" 和 "ok"。

如果指定的副本不存在或其状态为坏，该副本将被忽略。

注意：

被设置为 Bad 状态的副本可能会立即被删除，请谨慎操作。

## 示例

1. 将 BE 10001 上的 tablet 10003 的副本状态设置为 bad。

   ```sql
   ADMIN SET REPLICA STATUS PROPERTIES("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");
   ```

2. 将 BE 10001 上的 tablet 10003 的副本状态设置为 ok。

   ```sql
   ADMIN SET REPLICA STATUS PROPERTIES("tablet_id" = "10003", "backend_id" = "10001", "status" = "ok");
   ```