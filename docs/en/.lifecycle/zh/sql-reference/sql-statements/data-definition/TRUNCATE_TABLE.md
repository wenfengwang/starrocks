---
displayed_sidebar: English
---

# 截断表

## 描述

该语句用于截断指定的表和分区数据。

语法：

```sql
TRUNCATE TABLE [db.]tbl[ PARTITION(p1, p2, ...)]
```

注意：

1. 此语句用于清空数据，但保留表或分区。
2. 与 DELETE 不同，该语句只能整体清空指定的表或分区，无法添加过滤条件。
3. 与 DELETE 不同，使用此方法清除数据不会影响查询性能。
4. 该语句直接删除数据，删除后的数据无法恢复。
5. 执行此操作的表必须处于 NORMAL 状态。例如，不能对正在进行 SCHEMA CHANGE 的表执行 TRUNCATE TABLE。

## 例子

1. 截断 `example_db` 中的 `tbl` 表。

    ```sql
    TRUNCATE TABLE example_db.tbl;
    ```

2. 截断 `tbl` 表中的 `p1` 和 `p2` 分区。

    ```sql
    TRUNCATE TABLE tbl PARTITION(p1, p2);
    ```
