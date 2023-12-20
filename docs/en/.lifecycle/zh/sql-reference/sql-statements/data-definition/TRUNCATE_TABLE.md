---
displayed_sidebar: English
---

# TRUNCATE TABLE

## 描述

此语句用于清空指定表和分区数据。

语法：

```sql
TRUNCATE TABLE [db.]tbl[ PARTITION(p1, p2, ...)]
```

注意：

1. 此语句用于清空数据，同时保留表或分区。
2. 与 DELETE 不同，此语句只能整体清空指定的表或分区，不能添加过滤条件。
3. 与 DELETE 不同，使用此方法清空数据不会影响查询性能。
4. 此语句会直接删除数据，删除后的数据无法恢复。
5. 执行此操作的表必须处于 NORMAL 状态。例如，不能在进行 SCHEMA CHANGE 的表上执行 TRUNCATE TABLE。

## 示例

1. 清空 `example_db` 下的 `tbl` 表。

   ```sql
   TRUNCATE TABLE example_db.tbl;
   ```

2. 清空 `tbl` 表中的 `p1` 和 `p2` 分区。

   ```sql
   TRUNCATE TABLE tbl PARTITION(p1, p2);
   ```