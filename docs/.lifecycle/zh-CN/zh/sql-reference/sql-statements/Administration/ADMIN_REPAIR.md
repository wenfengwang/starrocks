---
displayed_sidebar: "中文"
---

# 管理员修复

## 功能

该语句用于尝试以高优先级修复指定的表或分区。

## 语法

```sql
ADMIN REPAIR TABLE 表名 [ PARTITION (分区1,...)]
```

说明：

1. 该语句仅表示让系统尝试以高优先级修复指定表或分区的分片副本，并不保证能够修复成功。用户可以通过 `ADMIN SHOW REPLICA STATUS;` 命令查看修复情况。
2. 默认的超时时长是 14400 秒(4 小时)。超时意味着系统将不会继续以高优先级修复指定表或分区的分片副本。需要时，须重新使用该命令进行设置。

## 示例

1. 尝试修复指定表

    ```sql
    ADMIN REPAIR TABLE tbl1;
    ```

2. 尝试修复指定分区

    ```sql
    ADMIN REPAIR TABLE tbl1 PARTITION (p1, p2);
    ```