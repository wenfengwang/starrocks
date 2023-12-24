---
displayed_sidebar: English
---

# 取消管理员维修

## 描述

该语句用于取消对指定表或分区的高优先级修复。

:::提示

此操作需要系统级别的OPERATE权限。您可以按照[GRANT](../account-management/GRANT.md)中的说明授予此权限。

:::

## 语法

```sql
ADMIN CANCEL REPAIR TABLE table_name[ PARTITION (p1,...)]
```

注意
>
> 该语句仅表示系统将不再以高优先级调度修复指定表或分区的分片副本。默认情况下，系统仍会修复这些副本。

## 例子

1. 取消高优先级维修

    ```sql
    ADMIN CANCEL REPAIR TABLE tbl PARTITION(p1);
    ```
