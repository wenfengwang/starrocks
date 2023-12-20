---
displayed_sidebar: English
---

# 创建表结构相同

## 描述

基于另一个表的定义创建一个结构相同的空表。这个定义包括列的定义、分区以及表的属性。

## 语法

```sql
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] [database.]table_name LIKE [database.]table_name
```

> **注意**

1. 您必须对原始表拥有SELECT权限。
2. 您可以复制像MySQL这样的外部表。

## 示例

1. 在test1数据库下，创建一个空表table2，其结构与table1相同。

   ```sql
   CREATE TABLE test1.table2 LIKE test1.table1
   ```

2. 在test2数据库下，创建一个空表table2，其结构与test1数据库中的table1相同。

   ```sql
   CREATE TABLE test2.table2 LIKE test1.table1
   ```

3. 在test1数据库下，创建一个空表table2，其结构与MySQL外部表相同。

   ```sql
   CREATE TABLE test1.table2 LIKE test1.table1
   ```
