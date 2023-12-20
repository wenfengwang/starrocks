---
displayed_sidebar: English
---

# 取消 ALTER TABLE 操作

## 描述

取消正在进行的 ALTER TABLE 操作的执行，包括：

- 修改列。
- 优化表架构（从 v3.2 开始），包括修改分桶方法和桶数。
- 创建和删除 rollup 索引。

> **注意**
- 此语句是同步操作。
- 执行此语句需要您对表拥有 `ALTER_PRIV` 权限。
- 此语句仅支持取消使用 ALTER TABLE 的异步操作（如上所述），不支持取消使用 ALTER TABLE 的同步操作，例如 rename。

## 语法

```SQL
CANCEL ALTER TABLE { COLUMN | OPTIMIZE | ROLLUP } FROM [db_name.]table_name
```

## 参数

- `{COLUMN | OPTIMIZE | ROLLUP}`：

  - 如果指定 `COLUMN`，则此语句取消修改列的操作。
  - 如果指定 `OPTIMIZE`，则此语句取消优化表架构的操作。
  - 如果指定 `ROLLUP`，则此语句取消添加或删除 rollup 索引的操作。

- `db_name`：可选。表所属的数据库名称。如果未指定此参数，则默认使用您当前的数据库。
- `table_name`：必填。表名。

## 示例

1. 取消在 `example_db` 数据库中对 `example_table` 表进行修改列的操作。

   ```SQL
   CANCEL ALTER TABLE COLUMN FROM example_db.example_table;
   ```

2. 取消在 `example_db` 数据库中对 `example_table` 表进行表架构优化的操作。

   ```SQL
   CANCEL ALTER TABLE OPTIMIZE FROM example_db.example_table;
   ```

3. 取消在当前数据库中对 `example_table` 表添加或删除 rollup 索引的操作。

   ```SQL
   CANCEL ALTER TABLE ROLLUP FROM example_table;
   ```