---
displayed_sidebar: English
---

# 清空表

## 描述

此语句用于清空指定表格及其分区中的数据。

语法：

```sql
TRUNCATE TABLE [db.]tbl[ PARTITION(p1, p2, ...)]
```

注意：

1. 此语句用于清空数据，同时保留表格或分区结构。
2. 不同于DELETE语句，此语句只能完全清空指定的表格或分区，且无法添加筛选条件。
3. 不同于DELETE语句，使用此方法清空数据不会影响查询性能。
4. 此语句会直接删除数据，删除后的数据将无法恢复。
5. 进行此操作的表格必须处于正常（NORMAL）状态。例如，你不能在一个正在变更模式（SCHEMA CHANGE）的表格上执行TRUNCATE TABLE操作。

## 示例

1. 清空example_db数据库下的tbl表。

   ```sql
   TRUNCATE TABLE example_db.tbl;
   ```

2. 清空tbl表中的p1和p2分区。

   ```sql
   TRUNCATE TABLE tbl PARTITION(p1, p2);
   ```
