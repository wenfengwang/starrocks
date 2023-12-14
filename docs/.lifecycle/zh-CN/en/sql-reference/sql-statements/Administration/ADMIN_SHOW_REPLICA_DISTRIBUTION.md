---
displayed_sidebar: "Chinese"
---

# ADMIN SHOW REPLICA DISTRIBUTION

## 描述

该语句用于显示表或分区副本的分发状态。

语法：

```sql
ADMIN SHOW REPLICA DISTRIBUTION FROM [db_name.]tbl_name [PARTITION (p1, ...)]
```

注意：

结果中的Graph列以图形方式显示副本的分发比例。

## 例子

1. 查看表的副本分发

    ```sql
    ADMIN SHOW REPLICA DISTRIBUTION FROM tbl1;
    ```

2. 查看表中分区的副本分发

    ```sql
    ADMIN SHOW REPLICA DISTRIBUTION FROM db1.tbl1 PARTITION(p1, p2);
    ```