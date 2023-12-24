---
displayed_sidebar: English
---

# 创建视图

## 描述

创建视图。

视图，或逻辑视图，是一个虚拟表，其数据源自对其他现有物理表的查询。因此，视图不占用物理存储空间，对视图的所有查询都等同于构建视图所使用的查询语句的子查询。

有关StarRocks支持的物化视图的信息，请参见[同步物化视图](../../../using_starrocks/Materialized_view-single_table.md)和[异步物化视图](../../../using_starrocks/Materialized_view.md)。

> **注意**
>
> 只有对特定数据库具有CREATE VIEW权限的用户才能执行此操作。

## 语法

```SQL
CREATE [OR REPLACE] VIEW [IF NOT EXISTS]
[<database>.]<view_name>
(
    <column_name>[ COMMENT 'column comment']
    [, <column_name>[ COMMENT 'column comment'], ...]
)
[COMMENT 'view comment']
AS <query_statement>
```

## 参数

| **参数**   | **描述**                                              |
| --------------- | ------------------------------------------------------------ |
| OR REPLACE      | 替换现有视图。                                    |
| database        | 视图所在的数据库名称。             |
| view_name       | 视图的名称。                                        |
| column_name     | 视图中的列名。请注意，视图中的列和`query_statement`中查询的列在数量上必须一致。 |
| COMMENT         | 视图中列或视图本身的注释。    |
| query_statement | 用于创建视图的查询语句。可以是StarRocks支持的任何查询语句。 |

## 使用说明

- 查询视图需要对视图及其对应的基表具有SELECT权限。
- 如果由于基表的模式更改导致无法执行构建视图的查询语句，当查询视图时StarRocks会返回错误。

## 例子

示例1：在`example_db`中创建名为`example_view`的视图，其中包含针对`example_table`的聚合查询。

```SQL
CREATE VIEW example_db.example_view (k1, k2, k3, v1)
AS
SELECT c1 as k1, k2, k3, SUM(v1) FROM example_table
WHERE k1 = 20160112 GROUP BY k1,k2,k3;
```

示例2：在数据库`example_db`中创建名为`example_view`的视图，其中包含针对表`example_table`的聚合查询，并为视图及其中的每列指定注释。

```SQL
CREATE VIEW example_db.example_view
(
    k1 COMMENT 'first key',
    k2 COMMENT 'second key',
    k3 COMMENT 'third key',
    v1 COMMENT 'first value'
)
COMMENT 'my first view'
AS
SELECT c1 as k1, k2, k3, SUM(v1) FROM example_table
WHERE k1 = 20160112 GROUP BY k1,k2,k3;
```

## 相关 SQL

- [显示创建视图](../data-manipulation/SHOW_CREATE_VIEW.md)
- [更改视图](./ALTER_VIEW.md)
- [删除视图](./DROP_VIEW.md)
