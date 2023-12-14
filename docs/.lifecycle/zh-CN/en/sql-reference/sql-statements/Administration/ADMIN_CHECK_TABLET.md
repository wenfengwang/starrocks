---
displayed_sidebar: "Chinese"
---

# 管理员检查平板

## 描述

此语句用于检查一组平板的状态。

语法：

```sql
ADMIN CHECK TABLET (tablet_id1, tablet_id2, ...)
PROPERTIES("type" = "...")
```

注意：

1. 平板id和PROPERTIES中必须指定"type"属性。

2. 当前，"type"仅支持：

   一致性：检查平板副本的一致性。此命令是异步的。发送命令后，StarRocks将开始检查相应平板的一致性。最终结果将显示在SHOW PROC "/statistic"结果的InconsistentTabletNum列中。

## 示例

1. 检查指定平板上副本的一致性

    ```sql
    ADMIN CHECK TABLET (10000, 10001)
    PROPERTIES("type" = "consistency");
    ```