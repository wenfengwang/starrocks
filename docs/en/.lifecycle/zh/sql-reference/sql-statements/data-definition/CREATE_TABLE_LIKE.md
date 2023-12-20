---
displayed_sidebar: English
---

# CREATE TABLE LIKE

## 描述

根据另一个表的定义创建一个相同的空表。定义包括列定义、分区以及表属性。

## 语法

```sql
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] [database.]table_name LIKE [database.]table_name
```

> **注意**

1. 您必须拥有原表的 `SELECT` 权限。
2. 您可以复制一个外部表，例如 MySQL。

## 示例

1. 在 test1 数据库下，创建一个与 table1 结构相同的空表，命名为 table2。

   ```sql
   CREATE TABLE test1.table2 LIKE test1.table1
   ```

2. 在 test2 数据库下，创建一个与 test1.table1 结构相同的空表，命名为 table2。

   ```sql
   CREATE TABLE test2.table2 LIKE test1.table1
   ```

3. 在 test1 数据库下，创建一个与 MySQL 外部表结构相同的空表，命名为 table2。

   ```sql
   CREATE TABLE test1.table2 LIKE test1.table1
   ```