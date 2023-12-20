---
displayed_sidebar: English
---

# 数据恢复

StarRocks 支持误删除数据库/表/分区的数据恢复功能。在执行删除表格或删除数据库的操作后，StarRocks 并不会立刻从物理层面删除数据，而是会将其暂存于垃圾箱中一定周期（默认为1天）。管理员可以利用 RECOVER 命令来恢复误删除的数据。

## 相关命令

语法：

```sql
-- 1) Recover database
RECOVER DATABASE db_name;
-- 2) Restore table
RECOVER TABLE [db_name.]table_name;
-- 3) Recover partition
RECOVER PARTITION partition_name FROM [db_name.]table_name;
```

## 注意事项

1. 此操作仅能恢复已删除的元数据信息。默认时间为1天，该时间周期可以通过在 fe.conf 文件中设置 catalog_trash_expire_second 参数来进行调整。
2. 如果在元数据被删除之后，创建了具有相同名称和类型的新元数据，那么之前被删除的元数据将无法被恢复。

## 示例

1. 恢复名为 example_db 的数据库

   ```sql
   RECOVER DATABASE example_db;
   ~~~ 2.
   
   ```

2. 恢复名为 example_tbl 的表格

   ```sql
   RECOVER TABLE example_db.example_tbl;
   ~~~ 3.
   
   ```

3. 恢复名为 p1 的分区，在 example_tbl 表格中

   ```sql
   RECOVER PARTITION p1 FROM example_tbl;
   ```
