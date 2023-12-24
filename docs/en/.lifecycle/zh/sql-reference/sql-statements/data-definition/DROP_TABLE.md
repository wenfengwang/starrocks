---
displayed_sidebar: English
---

# 删除表

## 描述

该语句用于删除表。

## 语法

```sql
DROP TABLE [IF EXISTS] [db_name.]table_name [FORCE]
```

注意：

- 如果在 24 小时内使用 DROP TABLE 语句删除了表，则可以使用 [RECOVER](../data-definition/RECOVER.md) 语句恢复该表。
- 如果执行 DROP TABLE FORCE，则该表将被直接删除，而且在未检查数据库中是否有未完成的活动的情况下将无法恢复。通常不建议执行此操作。

## 例子

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

## 引用

- [创建表](CREATE_TABLE.md)
- [显示表格](../data-manipulation/SHOW_TABLES.md)
- [显示创建表](../data-manipulation/SHOW_CREATE_TABLE.md)
- [更改表](ALTER_TABLE.md)
- [显示更改表](../data-manipulation/SHOW_ALTER.md)
