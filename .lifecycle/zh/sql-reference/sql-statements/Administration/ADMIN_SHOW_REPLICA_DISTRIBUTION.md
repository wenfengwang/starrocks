---
displayed_sidebar: English
---

# 管理员展示副本分布情况

## 说明

此语句用于展示数据表或分区副本的分布状况。

:::提示

执行此操作需要 **SYSTEM** 级别的 **OPERATE** 权限。您可以依照 [GRANT](../account-management/GRANT.md) 指令中的说明来授予相应权限。

:::

## 语法

```sql
ADMIN SHOW REPLICA DISTRIBUTION FROM [db_name.]tbl_name [PARTITION (p1, ...)]
```

注意：

结果中的 Graph 列通过图形的方式展示了副本分布的比例。

## 示例

1. 查看数据表的副本分布情况

   ```sql
   ADMIN SHOW REPLICA DISTRIBUTION FROM tbl1;
   ```

2. 查看数据表内分区的副本分布情况

   ```sql
   ADMIN SHOW REPLICA DISTRIBUTION FROM db1.tbl1 PARTITION(p1, p2);
   ```
