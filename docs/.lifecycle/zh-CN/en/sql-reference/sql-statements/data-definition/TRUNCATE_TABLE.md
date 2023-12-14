---
displayed_sidebar: "Chinese"
---

# 截断表

## 描述

该语句用于截断指定表和分区数据。

语法：

```sql
TRUNCATE TABLE [db.]tbl[ PARTITION(p1, p2, ...)]
```

注意：

1. 该语句用于截断数据，同时保留表或分区。
2. 与DELETE不同，该语句只能整体清空指定的表或分区，不能添加过滤条件。
3. 与DELETE不同，使用该方法清除数据不会影响查询性能。
4. 该语句直接删除数据。删除的数据无法恢复。
5. 执行此操作的表必须处于NORMAL状态。例如，不能对正在进行SCHEMA CHANGE的表执行TRUNCATE TABLE操作。

## 示例

1. 截断`example_db`中`tbl`表。

    ```sql
    TRUNCATE TABLE example_db.tbl;
    ```

2. 截断`tbl`表中的`p1`和`p2`分区。

    ```sql
    TRUNCATE TABLE tbl PARTITION(p1, p2);
    ```