---
displayed_sidebar: English
---

# ADMIN SHOW REPLICA DISTRIBUTION

## 描述

此语句用于展示表或分区副本的分布状态。

:::tip

此操作需要 SYSTEM 级别的 OPERATE 权限。您可以按照 [GRANT](../account-management/GRANT.md) 文档中的指引来授予此权限。

:::

## 语法

```sql
ADMIN SHOW REPLICA DISTRIBUTION FROM [db_name.]tbl_name [PARTITION (p1, ...)]
```

注意：

结果中的 Graph 列以图形方式展示了副本的分布比例。

## 示例

1. 查看表的副本分布情况

   ```sql
   ADMIN SHOW REPLICA DISTRIBUTION FROM tbl1;
   ```

2. 查看表中分区的副本分布情况

   ```sql
   ADMIN SHOW REPLICA DISTRIBUTION FROM db1.tbl1 PARTITION(p1, p2);
   ```