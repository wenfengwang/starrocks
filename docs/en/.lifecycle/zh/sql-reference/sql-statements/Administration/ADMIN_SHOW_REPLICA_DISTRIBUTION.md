---
displayed_sidebar: English
---

# 显示副本分布的管理员

## 描述

此语句用于显示表或分区副本的分布状态。

:::提示

此操作需要 SYSTEM 级别的 OPERATE 权限。您可以按照 [GRANT](../account-management/GRANT.md) 中的说明来授予此权限。

:::

## 语法

```sql
ADMIN SHOW REPLICA DISTRIBUTION FROM [db_name.]tbl_name [PARTITION (p1, ...)]
```

注意：

结果中的“图形”列以图形方式显示副本的分布比率。

## 例子

1. 查看表的副本分布

    ```sql
    ADMIN SHOW REPLICA DISTRIBUTION FROM tbl1;
    ```

2. 查看表中分区的副本分布

    ```sql
    ADMIN SHOW REPLICA DISTRIBUTION FROM db1.tbl1 PARTITION(p1, p2);
    ```
