---
displayed_sidebar: English
---

# 管理员检查表格

## 描述

此语句用于检查一组表格。

:::tip

此操作需要 SYSTEM 级别的 OPERATE 权限。您可以按照 [GRANT](../account-management/GRANT.md) 中的说明来授予此权限。

:::

## 语法

```sql
ADMIN CHECK TABLET (tablet_id1, tablet_id2, ...)
PROPERTIES("type" = "...")
```

注意：

1. 必须指定 tablet_id 和 PROPERTIES 中的 "type" 属性。

2. 目前，"type" 仅支持：

   一致性：检查 tablet 副本的一致性。此命令为异步执行。发送后，StarRocks 将开始检查相应 tablets 之间的一致性。最终结果将在 SHOW PROC "/statistic" 结果的 InconsistentTabletNum 列中显示。

## 示例

1. 检查一组指定 tablets 上副本的一致性

   ```sql
   ADMIN CHECK TABLET (10000, 10001)
   PROPERTIES("type" = "consistency");
   ```