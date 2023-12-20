---
displayed_sidebar: English
---

# ADMIN REPAIR

## 描述

此语句用于尝试修复指定的表或分区。

:::tip

此操作需要 SYSTEM 级别的 OPERATE 权限。您可以按照 [GRANT](../account-management/GRANT.md) 中的说明来授予此权限。

:::

## 语法

```sql
ADMIN REPAIR TABLE table_name [PARTITION (p1,...)]
```

注意：

1. 此语句仅表示系统会尝试以高优先级修复指定表或分区的分片副本，但不能保证修复一定成功。用户可以通过 ADMIN SHOW REPLICA STATUS 命令查看修复状态。
2. 默认的超时时间是 14400 秒（4 小时）。超时意味着系统将不会以高优先级修复指定表或分区的分片副本。如果发生超时，需要重新使用该命令以应用预期的设置。

## 示例

1. 尝试修复指定表

   ```sql
   ADMIN REPAIR TABLE tbl1;
   ```

2. 尝试修复指定分区

   ```sql
   ADMIN REPAIR TABLE tbl1 PARTITION (p1, p2);
   ```