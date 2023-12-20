---
displayed_sidebar: English
---

# ADMIN SHOW REPLICA STATUS

## 描述

此语句用于显示表或分区的副本状态。

:::tip

此操作需要 SYSTEM 级别的 OPERATE 权限。您可以按照 [GRANT](../account-management/GRANT.md) 中的说明来授予此权限。

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
OK:            副本健康
DEAD:          副本的 Backend 不可用
VERSION_ERROR: 副本数据版本丢失
SCHEMA_ERROR:  副本的 schema hash 不正确
MISSING:       副本不存在
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

3. 查看表的所有非健康副本。

   ```sql
   ADMIN SHOW REPLICA STATUS FROM tbl1
   WHERE STATUS != "OK";
   ```