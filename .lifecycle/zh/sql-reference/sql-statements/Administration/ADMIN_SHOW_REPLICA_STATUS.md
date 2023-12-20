---
displayed_sidebar: English
---

# 管理员查看副本状态

## 描述

此语句用于查看表或分区副本的状态。

:::提示

此操作需要 **SYSTEM** 级别的 **OPERATE** 权限。您可以遵循 [GRANT](../account-management/GRANT.md) 中的指引来授予该权限。

:::

## 语法

```sql
ADMIN SHOW REPLICA STATUS FROM [db_name.]tbl_name [PARTITION (p1, ...)]
[where_clause]
```

```sql
where_clause:
WHERE STATUS [!]= "replica_status"
```

```plain
replica_status:
OK:            The replica is healthy
DEAD:          The Backend of replica is not available
VERSION_ERROR: The replica data version is missing
SCHEMA_ERROR:  The schema hash of replica is incorrect
MISSING:       The replica does not exist
```

## 示例

1. 查看表的所有副本状态。

   ```sql
   ADMIN SHOW REPLICA STATUS FROM db1.tbl1;
   ```

2. 查看状态为 VERSION_ERROR 的分区副本。

   ```sql
   ADMIN SHOW REPLICA STATUS FROM tbl1 PARTITION (p1, p2)
   WHERE STATUS = "VERSION_ERROR";
   ```

3. 查看表的所有不健康的副本。

   ```sql
   ADMIN SHOW REPLICA STATUS FROM tbl1
   WHERE STATUS != "OK";
   ```
