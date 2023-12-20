---
displayed_sidebar: English
---

# 管理员设置副本状态

## 说明

此语句用于设定指定副本的状态。

目前，这个命令只用于手动将一些副本的状态设置为“BAD”或“OK”，以便系统能够自动对这些副本进行修复。

:::提示

执行此操作需要**SYSTEM**级别的**OPERATE**权限。您可以依照[GRANT](../account-management/GRANT.md)中的指南来授予该权限。

:::

## 语法

```sql
ADMIN SET REPLICA STATUS
PROPERTIES ("key" = "value", ...)
```

当前支持以下属性：

"table_id"：必需。指定Tablet的ID。

"backend_id"：必需。指定Backend的ID。

"status"：必需。指定状态。目前只支持“bad”和“ok”。

如果指定的副本不存在或其状态不良，则会忽略该副本。

注意：

被设置为Bad状态的副本可能会立即被删除，请慎重操作。

## 示例

1. 将BE 10001上的tablet 10003的副本状态设置为bad。

   ```sql
   ADMIN SET REPLICA STATUS PROPERTIES("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");
   ```

2. 将BE 10001上的tablet 10003的副本状态设置为ok。

   ```sql
   ADMIN SET REPLICA STATUS PROPERTIES("tablet_id" = "10003", "backend_id" = "10001", "status" = "ok");
   ```
