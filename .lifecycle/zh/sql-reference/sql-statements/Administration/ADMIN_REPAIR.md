---
displayed_sidebar: English
---

# 管理员修理

## 描述

此语句用于尝试修复指定的数据表或分区。

:::提示

此操作需要系统级的**OPERATE**权限。您可以遵循[GRANT](../account-management/GRANT.md)命令中的指示来授予相应权限。

:::

## 语法

```sql
ADMIN REPAIR TABLE table_name[ PARTITION (p1,...)]
```

注意：

1. 此语句只意味着系统会尝试优先修复指定数据表或分区的分片副本，但不能保证修复一定会成功。用户可以通过ADMIN SHOW REPLICA STATUS命令查看修复状态。
2. 默认的超时时间是14400秒（4小时）。超时意味着系统将不会优先修复指定数据表或分区的分片副本。在超时的情况下，需要重新使用该命令来进行所需的设置。

## 示例

1. 尝试修复指定的数据表

   ```sql
   ADMIN REPAIR TABLE tbl1;
   ```

2. 尝试修复指定的分区

   ```sql
   ADMIN REPAIR TABLE tbl1 PARTITION (p1, p2);
   ```
