---
displayed_sidebar: English
---

# 管理员取消修复

## 描述

此语句用于取消修复指定表或分区的高优先级修复任务。

:::tip

此操作需要 **SYSTEM** 级别的 **OPERATE** 权限。您可以遵循 [GRANT](../account-management/GRANT.md) 中的指引来授予此权限。

:::

## 语法

```sql
ADMIN CANCEL REPAIR TABLE table_name[ PARTITION (p1,...)]
```

注意
> 此语句仅表示系统不再对指定表或分区的分片副本进行高优先级修复。系统仍会按照默认的调度策略对这些副本进行修复。

## 示例

1. 取消高优先级修复

   ```sql
   ADMIN CANCEL REPAIR TABLE tbl PARTITION(p1);
   ```