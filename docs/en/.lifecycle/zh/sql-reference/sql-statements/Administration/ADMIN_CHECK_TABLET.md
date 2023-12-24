---
displayed_sidebar: English
---

# 管理员检查平板

## 描述

此语句用于检查一组平板电脑。

:::提示

此操作需要 SYSTEM 级别的 OPERATE 权限。您可以按照 [GRANT](../account-management/GRANT.md) 中的说明授予此权限。

:::

## 语法

```sql
ADMIN CHECK TABLET (tablet_id1, tablet_id2, ...)
PROPERTIES("type" = "...")
```

注意：

1. 必须指定 tablet id 和 PROPERTIES 中的“type”属性。

2. 目前，“type”仅支持：

   一致性：检查平板电脑副本的一致性。此命令是异步的。发送命令后，StarRocks 将开始检查对应平板电脑的一致性。最终结果将显示在 SHOW PROC “/statistic” 结果的 InconsistentTabletNum 列中。

## 例子

1. 检查一组指定平板电脑上的副本一致性

    ```sql
    ADMIN CHECK TABLET (10000, 10001)
    PROPERTIES("type" = "consistency");
    ```
