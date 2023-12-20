---
displayed_sidebar: English
---

# 显示 ALTER TABLE 操作

## 描述

显示正在进行的 ALTER TABLE 操作的执行情况，包括：

- 修改列。
- 优化表结构（从v3.2开始），包括修改分桶方法和分桶数量。
- 创建和删除 rollup 索引。

## 语法

- 显示修改列或优化表结构操作的执行情况。

  ```sql
  SHOW ALTER TABLE { COLUMN | OPTIMIZE } [FROM db_name] [WHERE TableName|CreateTime|FinishTime|State] [ORDER BY] [LIMIT]
  ```

- 显示添加或删除 rollup 索引操作的执行情况。

  ```sql
  SHOW ALTER TABLE ROLLUP [FROM db_name]
  ```

## 参数

- `{COLUMN | OPTIMIZE | ROLLUP}`：

  - 如果指定 `COLUMN`，则该语句显示修改列的操作。
  - 如果指定 `OPTIMIZE`，则该语句显示优化表结构的操作。
  - 如果指定 `ROLLUP`，则该语句显示添加或删除 rollup 索引的操作。

- `db_name`：可选。如果未指定 `db_name`，则默认使用当前数据库。

## 示例

1. 显示当前数据库中修改列、优化表结构、创建或删除 rollup 索引操作的执行情况。

   ```sql
   SHOW ALTER TABLE COLUMN;
   SHOW ALTER TABLE OPTIMIZE;
   SHOW ALTER TABLE ROLLUP;
   ```

2. 显示指定数据库中修改列、优化表结构、创建或删除 rollup 索引相关操作的执行情况。

   ```sql
   SHOW ALTER TABLE COLUMN FROM example_db;
   SHOW ALTER TABLE OPTIMIZE FROM example_db;
   SHOW ALTER TABLE ROLLUP FROM example_db;
   ```

3. 显示指定表中最近一次修改列或优化表结构操作的执行情况。

   ```sql
   SHOW ALTER TABLE COLUMN WHERE TableName = "table1" ORDER BY CreateTime DESC LIMIT 1;
   SHOW ALTER TABLE OPTIMIZE WHERE TableName = "table1" ORDER BY CreateTime DESC LIMIT 1; 
   ```

## 参考资料

- [CREATE TABLE](../data-definition/CREATE_TABLE.md)
- [ALTER TABLE](../data-definition/ALTER_TABLE.md)
- [SHOW TABLES](../data-manipulation/SHOW_TABLES.md)
- [SHOW CREATE TABLE](../data-manipulation/SHOW_CREATE_TABLE.md)