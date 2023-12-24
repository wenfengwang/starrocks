---
displayed_sidebar: English
---

# 创建类似表格

## 描述

根据另一个表的定义创建一个相同的空表。该定义包括列定义、分区和表属性。

## 语法

```sql
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] [database.]table_name LIKE [database.]table_name
```

> **注意**

1. 您必须对原始表具有 `SELECT` 权限。
2. 您可以复制外部表，例如 MySQL。

## 例

1. 在 test1 数据库中，创建一个与 table1 具有相同表结构的空表，命名为 table2。

    ```sql
    CREATE TABLE test1.table2 LIKE test1.table1
    ```

2. 在 test2 数据库中，创建一个与 test1.table1 具有相同表结构的空表，命名为 table2。

    ```sql
    CREATE TABLE test2.table2 LIKE test1.table1
    ```

3. 在 test1 数据库中，创建一个与 MySQL 外部表具有相同表结构的空表，命名为 table2。

    ```sql
    CREATE TABLE test1.table2 LIKE test1.table1
    ```
