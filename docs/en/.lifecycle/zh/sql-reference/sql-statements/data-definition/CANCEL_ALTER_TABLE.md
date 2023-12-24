---
displayed_sidebar: English
---

# 取消 ALTER TABLE

## 描述

取消正在进行的 ALTER TABLE 操作的执行，包括：

- 修改列。
- 优化表结构（从 v3.2 开始），包括修改分桶方法和分桶数量。
- 创建和删除汇总索引。

> **注意**
>
> - 该语句是同步操作。
> - 该语句要求您对表具有 `ALTER_PRIV` 权限。
> - 该语句仅支持取消使用 ALTER TABLE 进行的异步操作（如上所述），不支持取消使用 ALTER TABLE 进行的同步操作，例如重命名。

## 语法

   ```SQL
   CANCEL ALTER TABLE { COLUMN | OPTIMIZE | ROLLUP } FROM [db_name.]table_name
   ```

## 参数

- `{COLUMN ｜ OPTIMIZE | ROLLUP}`：

  - 如果指定 `COLUMN`，则该语句取消修改列的操作。
  - 如果指定 `OPTIMIZE`，则该语句取消优化表结构的操作。
  - 如果指定 `ROLLUP`，则该语句取消添加或删除汇总索引的操作。

- `db_name`：可选。表所属的数据库名称。如果未指定此参数，默认使用当前数据库。
- `table_name`：必填。表名。

## 例子

1. 取消在数据库 `example_db` 中对 `example_table` 的列修改操作。

   ```SQL
   CANCEL ALTER TABLE COLUMN FROM example_db.example_table;
   ```

2. 取消在数据库 `example_db` 中对 `example_table` 的表结构优化操作。

   ```SQL
   CANCEL ALTER TABLE OPTIMIZE FROM example_db.example_table;
   ```

3. 取消在当前数据库中对 `example_table` 的汇总索引添加或删除操作。

   ```SQL
   CANCEL ALTER TABLE ROLLUP FROM example_table;
   ```
