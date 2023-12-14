---
displayed_sidebar: "Chinese"
---

# ADMIN SHOW REPLICA STATUS

## 描述

此语句用于显示表或分区的副本状态。

语法：

```sql
ADMIN SHOW REPLICA STATUS FROM [db_name.]tbl_name [PARTITION (p1, ...)]
[where_clause]
```

```sql
where_clause:
WHERE STATUS [!]= "replica_status"
```

```plain text
replica_status:
OK:            副本状态良好
DEAD:          副本的后端不可用
VERSION_ERROR: 副本数据版本丢失
SCHEMA_ERROR:  副本的模式哈希不正确
MISSING:       副本不存在
```

## 示例

1. 查看表的所有副本状态。

    ```sql
    ADMIN SHOW REPLICA STATUS FROM db1.tbl1;
    ```

2. 查看带有 "VERSION_ERROR" 状态的分区的副本。

    ```sql
    ADMIN SHOW REPLICA STATUS FROM tbl1 PARTITION (p1, p2)
    WHERE STATUS = "VERSION_ERROR";
    ```

3. 查看表的所有不健康的副本。

    ```sql
    ADMIN SHOW REPLICA STATUS FROM tbl1
    WHERE STATUS != "OK";
    ```