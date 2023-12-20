---
displayed_sidebar: English
---

# 删除表

## 描述

此语句用于删除一个表。

## 语法

```sql
DROP TABLE [IF EXISTS] [db_name.]table_name [FORCE]
```

注意：

- 如果表在使用`DROP TABLE`语句后的24小时内被删除，可以使用[RECOVER](../data-definition/RECOVER.md)语句恢复该表。
- 如果执行了DROP TABLE FORCE，表会被直接删除，并且在不检查数据库中是否有未完成的操作的情况下，将无法恢复。通常不建议执行此操作。

## 示例

1. 删除一个表。

   ```sql
   DROP TABLE my_table;
   ```

2. 如果该表存在，则删除指定数据库中的表。

   ```sql
   DROP TABLE IF EXISTS example_db.my_table;
   ```

3. 强制删除表并清除其在磁盘上的数据。

   ```sql
   DROP TABLE my_table FORCE;
   ```

## 参考资料

- [CREATE TABLE](CREATE_TABLE.md)（创建表）
- [显示表格](../data-manipulation/SHOW_TABLES.md)
- [显示创建表的语句](../data-manipulation/SHOW_CREATE_TABLE.md)
- [ALTER TABLE](ALTER_TABLE.md)（修改表）
- [SHOW ALTER TABLE](../data-manipulation/SHOW_ALTER.md)（显示修改表的语句）
