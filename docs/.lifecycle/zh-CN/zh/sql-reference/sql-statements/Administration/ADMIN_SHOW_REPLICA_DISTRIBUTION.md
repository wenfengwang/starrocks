---
displayed_sidebar: "中文"
---

# 管理员显示副本分布

## 功能

该语句用于显示表或分区副本的分布状态。

## 语法

```sql
ADMIN SHOW REPLICA DISTRIBUTION FROM [db_name.]tbl_name [PARTITION (p1, ...)]
```

说明：

结果中的 Graph 列以图形方式展示副本分布的比例。

## 示例

1. 查看表的副本分布

    ```sql
    ADMIN SHOW REPLICA DISTRIBUTION FROM tbl1;
    ```

2. 查看具有分区的表的副本分布

    ```sql
    ADMIN SHOW REPLICA DISTRIBUTION FROM db1.tbl1 PARTITION(p1, p2);
    ```