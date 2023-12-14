---
displayed_sidebar: "Chinese"
---

# 取消修改表

## 描述

取消对给定表格使用 ALTER TABLE 语句执行的以下操作：

- 表模式：添加和删除列、重新排序列和修改列的数据类型。
- Rollup 索引：创建和删除 Rollup 索引。

此语句是一个同步操作，需要您在表格上拥有 `ALTER_PRIV` 权限。

## 语法

- 取消模式更改。

    ```SQL
    CANCEL ALTER TABLE COLUMN FROM [db_name.]table_name
    ```

- 取消对 Rollup 索引的更改。

    ```SQL
    CANCEL ALTER TABLE ROLLUP FROM [db_name.]table_name
    ```

## 参数

| **参数**     | **是否必需** | **描述**                                                     |
| ------------ | ------------ | ------------------------------------------------------------ |
| db_name      | 否           | 表所属的数据库名称。如果未指定该参数，默认使用当前数据库。   |
| table_name   | 是           | 表名称。                                                     |

## 示例

示例 1：取消对 `example_db` 数据库中的 `example_table` 的模式更改。

```SQL
CANCEL ALTER TABLE COLUMN FROM example_db.example_table;
```

示例 2：取消对当前数据库中 `example_table` 的 Rollup 索引更改。

```SQL
CANCEL ALTER TABLE ROLLUP FROM example_table;
```