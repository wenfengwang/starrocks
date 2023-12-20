---
displayed_sidebar: English
---

# 恢复

## 说明

此语句用于恢复被删除的数据库、表或分区。

语法：

1. 恢复数据库

   ```sql
   RECOVER DATABASE <db_name>
   ```

2. 恢复表

   ```sql
   RECOVER TABLE [<db_name>.]<table_name>
   ```

3. 恢复分区

   ```sql
   RECOVER PARTITION partition_name FROM [<db_name>.]<table_name>
   ```

注意：

1. 它仅能够恢复一段时间前删除的元信息。默认时间为：一天。（您可以通过在fe.conf文件中配置参数catalog_trash_expire_second来修改这个时间。）
2. 如果元信息被删除的同时创建了一个相同的元信息，那么之前的元信息将无法被恢复。

## 示例

1. 恢复名为example_db的数据库

   ```sql
   RECOVER DATABASE example_db;
   ```

2. 恢复名为example_tbl的表

   ```sql
   RECOVER TABLE example_db.example_tbl;
   ```

3. 恢复名为p1的example_tbl表中的分区

   ```sql
   RECOVER PARTITION p1 FROM example_tbl;
   ```
