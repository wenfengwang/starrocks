---
displayed_sidebar: "Chinese"
---

# DROP TABLE

## 描述

该语句用于删除表。

## 语法

```sql
DROP TABLE [IF EXISTS] [db_name.]table_name [FORCE]
```

注意：

- 如果在 24 小时内使用 DROP TABLE 语句删除了表，您可以使用 [RECOVER](../data-definition/RECOVER.md) 语句来恢复表。
- 如果执行 DROP TABLE FORCE，表将直接被删除，不能在未检查数据库中是否有未完成的活动情况下恢复。通常不建议执行此操作。

## 示例

1. 删除表。

    ```sql
    DROP TABLE my_table;
    ```

2. 如果存在，则删除指定数据库中的表。

    ```sql
    DROP TABLE IF EXISTS example_db.my_table;
    ```

3. 强制删除表并清除其在磁盘上的数据。

    ```sql
    DROP TABLE my_table FORCE;
    ```

## 参考

- [CREATE TABLE](CREATE_TABLE.md)
- [SHOW TABLES](../data-manipulation/SHOW_TABLES.md)
- [SHOW CREATE TABLE](../data-manipulation/SHOW_CREATE_TABLE.md)
- [ALTER TABLE](ALTER_TABLE.md)
- [SHOW ALTER TABLE](../data-manipulation/SHOW_ALTER.md)