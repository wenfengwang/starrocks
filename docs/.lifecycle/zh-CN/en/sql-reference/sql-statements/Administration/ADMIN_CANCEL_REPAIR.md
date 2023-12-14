---
displayed_sidebar: "Chinese"
---

# 管理员取消修复

## 描述

此语句用于取消优先级高的指定表或分区的修复。

## 语法

```sql
ADMIN CANCEL REPAIR TABLE table_name[PARTITION(p1,...)]
```

注意
>
> 此语句仅表示系统不再修复指定表或分区的分片副本，优先级较高。它仍然按默认调度修复这些副本。

## 示例

1. 取消高优先级修复

    ```sql
    ADMIN CANCEL REPAIR TABLE tbl PARTITION(p1);
    ```