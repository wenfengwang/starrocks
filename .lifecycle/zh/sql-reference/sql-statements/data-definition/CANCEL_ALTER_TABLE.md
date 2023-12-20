---
displayed_sidebar: English
---

# 取消ALTER TABLE操作

## 描述

取消当前正在执行的ALTER TABLE操作，包括：

- 修改列。
- 优化表结构（从v3.2版本开始），包括修改桶分配方法和桶的数量。
- 创建和删除rollup索引。

> **注意事项**
- 此语句是一项同步操作。
- 执行此语句需要您对该表拥有ALTER_PRIV权限。
- 此语句只支持取消ALTER TABLE的异步操作（如上所述），不支持取消ALTER TABLE的同步操作，例如重命名。

## 语法

```SQL
CANCEL ALTER TABLE { COLUMN | OPTIMIZE | ROLLUP } FROM [db_name.]table_name
```

## 参数

- {COLUMN | OPTIMIZE | ROLLUP}：

  - 如果指定COLUMN，该语句将取消修改列的操作。
  - 如果指定OPTIMIZE，该语句将取消优化表结构的操作。
  - 如果指定ROLLUP，该语句将取消添加或删除rollup索引的操作。

- db_name：可选。指定表所属的数据库名称。如果未指定此参数，默认使用您当前的数据库。
- table_name：必需。指定的表名。

## 示例

1. 取消在example_db数据库中对example_table表修改列的操作。

   ```SQL
   CANCEL ALTER TABLE COLUMN FROM example_db.example_table;
   ```

2. 取消在example_db数据库中对example_table表优化表结构的操作。

   ```SQL
   CANCEL ALTER TABLE OPTIMIZE FROM example_db.example_table;
   ```

3. 取消在当前数据库中对example_table表添加或删除rollup索引的操作。

   ```SQL
   CANCEL ALTER TABLE ROLLUP FROM example_table;
   ```
