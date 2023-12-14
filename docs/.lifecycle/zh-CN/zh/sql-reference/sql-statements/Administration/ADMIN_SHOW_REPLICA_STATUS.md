---
displayed_sidebar: "中文"
---

# 管理显示副本状态

## 功能

该语句用于显示一个表或分区的副本状态信息。

## 语法

```sql
ADMIN SHOW REPLICA STATUS FROM [db_name.]tbl_name [PARTITION (p1, ...)]
[WHERE STATUS [!]= "replica_status"]
```

说明：

```plain text
replica_status:
OK:             副本处于健康状态
DEAD:           副本所在 Backend 不可用
VERSION_ERROR:  副本数据版本有缺失
SCHEMA_ERROR:   副本的 schema hash 不正确
MISSING:        副本不存在
```

## 示例

1. 查看表的所有副本状态。

    ```sql
    ADMIN SHOW REPLICA STATUS FROM db1.tbl1;
    ```

2. 查看表某个分区状态为 VERSION_ERROR 的副本。

    ```sql
    ADMIN SHOW REPLICA STATUS FROM tbl1 PARTITION (p1, p2)
    WHERE STATUS = "VERSION_ERROR";
    ```

3. 查看所有状态不健康的副本。

    ```sql
    ADMIN SHOW REPLICA STATUS FROM tbl1
    WHERE STATUS != "OK";
    ```