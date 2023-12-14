---
displayed_sidebar: "中文"
---

# CREATE TABLE LIKE

## 功能

该语句用于创建一个表结构和另一张表完全相同的空表。

## 语法

```sql
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] [database.]table_name LIKE [database.]table_name
```

说明:

1. 复制的表结构包括列定义、分区、表属性等。
2. 用户需要对被复制的原表具有 `SELECT` 权限，关于权限控制请参考 [GRANT](../account-management/GRANT.md) 章节。
3. 支持对 MySQL 等外部表的复制。

## 示例

1. 在 test1 数据库下创建一个和 table1 表结构相同的空表，新表名为 table2。

    ```sql
    CREATE TABLE test1.table2 LIKE test1.table1;
    ```

2. 在 test2 数据库下创建一个和 test1.table1 表结构相同的空表，新表名为 table2。

    ```sql
    CREATE TABLE test2.table2 LIKE test1.table1;
    ```

3. 在 test1 数据库下创建一个和 MySQL 外表 table1 表结构相同的空表，新表名为 table2。

    ```sql
    CREATE TABLE test1.table2 LIKE test1.table1;
    ```