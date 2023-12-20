---
displayed_sidebar: English
---

# 显示ALTER TABLE操作

## 描述

展示正在执行中的ALTER TABLE操作情况，包括：

- 修改列。
- 优化表结构（自v3.2版本起），包括改变分桶策略和桶的数量。
- 创建和删除rollup索引。

## 语法

- 展示正在执行的修改列或优化表结构的操作。

  ```sql
  SHOW ALTER TABLE { COLUMN | OPTIMIZE } [FROM db_name] [WHERE TableName|CreateTime|FinishTime|State] [ORDER BY] [LIMIT]
  ```

- 展示正在执行的添加或删除rollup索引的操作。

  ```sql
  SHOW ALTER TABLE ROLLUP [FROM db_name]
  ```

## 参数

- {COLUMN | OPTIMIZE | ROLLUP}：

  - 如果指定COLUMN，该语句将展示修改列的操作。
  - 如果指定OPTIMIZE，该语句将展示优化表结构的操作。
  - 如果指定ROLLUP，该语句将展示添加或删除rollup索引的操作。

- db_name：可选项。如果未指定db_name，默认使用当前数据库。

## 示例

1. 在当前数据库中展示修改列、优化表结构以及创建或删除rollup索引的操作执行情况。

   ```sql
   SHOW ALTER TABLE COLUMN;
   SHOW ALTER TABLE OPTIMIZE;
   SHOW ALTER TABLE ROLLUP;
   ```

2. 在指定数据库中展示与修改列、优化表结构以及创建或删除rollup索引相关的操作执行情况。

   ```sql
   SHOW ALTER TABLE COLUMN FROM example_db;
   SHOW ALTER TABLE OPTIMIZE FROM example_db;
   SHOW ALTER TABLE ROLLUP FROM example_db;
   ```

3. 在指定表中展示最近一次修改列或优化表结构的操作执行情况。

   ```sql
   SHOW ALTER TABLE COLUMN WHERE TableName = "table1" ORDER BY CreateTime DESC LIMIT 1;
   SHOW ALTER TABLE OPTIMIZE WHERE TableName = "table1" ORDER BY CreateTime DESC LIMIT 1; 
   ```

## 参考资料

- [创建表](../data-definition/CREATE_TABLE.md)
- [ALTER TABLE](../data-definition/ALTER_TABLE.md)
- [SHOW TABLES](../data-manipulation/SHOW_TABLES.md)
- [显示创建表](../data-manipulation/SHOW_CREATE_TABLE.md)
