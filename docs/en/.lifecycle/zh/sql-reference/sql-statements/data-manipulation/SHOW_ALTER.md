---
displayed_sidebar: English
---

# 显示 ALTER TABLE

## 描述

显示正在进行的 ALTER TABLE 操作的执行情况，包括：

- 修改列。
- 优化表结构（从 v3.2 开始），包括修改分桶方法和桶的数量。
- 创建和删除汇总索引。

## 语法

- 显示修改列或优化表结构等操作的执行情况。

    ```sql
    SHOW ALTER TABLE { COLUMN | OPTIMIZE } [FROM db_name] [WHERE TableName|CreateTime|FinishTime|State] [ORDER BY] [LIMIT]
    ```

- 显示添加或删除汇总索引的操作的执行情况。

    ```sql
    SHOW ALTER TABLE ROLLUP [FROM db_name]
    ```

## 参数

- `{COLUMN ｜ OPTIMIZE | ROLLUP}`:

  - 如果指定为 `COLUMN`，则该语句显示修改列的操作。
  - 如果指定为 `OPTIMIZE`，则该语句显示优化表结构的操作。
  - 如果指定为 `ROLLUP`，则该语句显示添加或删除汇总索引的操作。

- `db_name`：可选。如果未指定 `db_name`，则默认使用当前数据库。

## 例子

1. 显示在当前数据库中修改列、优化表结构以及创建或删除汇总索引等操作的执行情况。

    ```sql
    SHOW ALTER TABLE COLUMN;
    SHOW ALTER TABLE OPTIMIZE;
    SHOW ALTER TABLE ROLLUP;
    ```

2. 显示在指定数据库中与修改列、优化表结构以及创建或删除汇总索引相关的操作的执行情况。

    ```sql
    SHOW ALTER TABLE COLUMN FROM example_db;
    SHOW ALTER TABLE OPTIMIZE FROM example_db;
    SHOW ALTER TABLE ROLLUP FROM example_db;
    ```

3. 显示在指定表中修改列或优化表结构的最近操作的执行情况。

    ```sql
    SHOW ALTER TABLE COLUMN WHERE TableName = "table1" ORDER BY CreateTime DESC LIMIT 1;
    SHOW ALTER TABLE OPTIMIZE WHERE TableName = "table1" ORDER BY CreateTime DESC LIMIT 1; 
    ```

## 引用

- [创建表](../data-definition/CREATE_TABLE.md)
- [更改表](../data-definition/ALTER_TABLE.md)
- [显示表格](../data-manipulation/SHOW_TABLES.md)
- [显示创建表](../data-manipulation/SHOW_CREATE_TABLE.md)
