---
displayed_sidebar: "Chinese"
---

# 显示 ALTER TABLE

## 描述

显示正在进行的 ALTER TABLE 任务的执行情况。

## 语法

```sql
SHOW ALTER TABLE {COLUMN | ROLLUP} [FROM <db_name>]
```

## 参数

- COLUMN | ROLLUP

  - 如果指定了 COLUMN，则该语句显示修改列的任务。如果需要嵌套 WHERE 子句，则支持的语法为 `[WHERE TableName|CreateTime|FinishTime|State] [ORDER BY] [LIMIT]`。

  - 如果指定了 ROLLUP，则该语句显示创建或删除 ROLLUP 索引的任务。

- `db_name`：可选。如果未指定 `db_name`，则默认使用当前数据库。

## 示例

示例 1：显示当前数据库中的列修改任务。

```sql
SHOW ALTER TABLE COLUMN;
```

示例 2：显示表的最新列修改任务。

```sql
SHOW ALTER TABLE COLUMN WHERE TableName = "table1"
ORDER BY CreateTime DESC LIMIT 1;
 ```

示例 3：显示指定数据库中创建或删除 ROLLUP 索引的任务。

```sql
SHOW ALTER TABLE ROLLUP FROM example_db;
````

## 引用

- [CREATE TABLE](../data-definition/CREATE_TABLE.md)
- [ALTER TABLE](../data-definition/ALTER_TABLE.md)
- [SHOW TABLES](../data-manipulation/SHOW_TABLES.md)
- [SHOW CREATE TABLE](../data-manipulation/SHOW_CREATE_TABLE.md)