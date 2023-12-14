---
displayed_sidebar: "Chinese"
---

# 管理员修复

## 描述

此语句用于首先尝试修复指定的表或分区。

语法:

```sql
ADMIN REPAIR TABLE table_name[ PARTITION (p1,...)]
```

注意:

1. 此语句仅表示系统会优先尝试修复指定表或分区的分片副本，但不保证修复一定成功。用户可以通过ADMIN SHOW REPLICA STATUS命令查看修复状态。
2. 默认超时时间为14400秒（4小时）。超时意味着系统将不会优先修复指定表或分区的分片副本。在超时的情况下，需要重新使用该命令进行意图设置。

## 例子

1. 尝试修复指定的表

    ```sql
    ADMIN REPAIR TABLE tbl1;
    ```

2. 尝试修复指定的分区

    ```sql
    ADMIN REPAIR TABLE tbl1 PARTITION (p1, p2);
    ```