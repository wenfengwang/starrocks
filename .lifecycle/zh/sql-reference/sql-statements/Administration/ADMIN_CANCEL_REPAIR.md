---
displayed_sidebar: English
---

# 管理员取消修理

## 说明

此语句用于取消对指定表或分区的高优先级修理操作。

:::提示

该操作需要 **SYSTEM** 级别的 **OPERATE** 权限。您可以遵循 [GRANT](../account-management/GRANT.md) 指令中的说明来授予此权限。

:::

## 语法

```sql
ADMIN CANCEL REPAIR TABLE table_name[ PARTITION (p1,...)]
```

注意
> 此语句只表明系统将不再对指定表或分区的分片副本进行高优先级的修理。系统仍会按照默认的调度对这些副本进行修理。

## 示例

1. 取消高优先级的修理操作

   ```sql
   ADMIN CANCEL REPAIR TABLE tbl PARTITION(p1);
   ```
