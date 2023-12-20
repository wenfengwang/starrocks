---
displayed_sidebar: English
---

# 管理员检查数据表

## 描述

此语句用于检查一组数据表。

:::提示

此操作需要系统级的**OPERATE**权限。您可以按照[GRANT](../account-management/GRANT.md)命令的说明来授予此权限。

:::

## 语法

```sql
ADMIN CHECK TABLE (tablet_id1, tablet_id2, ...)
PROPERTIES("type" = "...")
```

注意：

1. 必须在tablet id和PROPERTIES中指定“type”属性。

2. 目前，“type”属性仅支持以下值：

   Consistency：检查数据表副本的一致性。此命令为异步执行。发送命令后，StarRocks将开始检查对应数据表副本之间的一致性。最终结果将在SHOW PROC "/statistic"命令的结果中的InconsistentTabletNum列显示。

## 示例

1. 检查指定一组数据表上副本的一致性

   ```sql
   ADMIN CHECK TABLET (10000, 10001)
   PROPERTIES("type" = "consistency");
   ```
