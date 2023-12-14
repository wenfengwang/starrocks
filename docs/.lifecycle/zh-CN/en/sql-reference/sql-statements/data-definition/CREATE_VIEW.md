---
displayed_sidebar: "Chinese"
---

# 创建视图

## 描述

创建一个视图。

视图，又称为逻辑视图，是一个虚拟表，其数据源自对其他已存在的物理表的查询。因此，视图不使用物理存储，对视图的所有查询都相当于对用于构建视图的查询语句的子查询。

关于StarRocks支持的物化视图的信息，请参阅[Synchronous materialized views]（../../../using_starrocks/Materialized_view-single_table.md）和[Asynchronous materialized views]（../../../using_starrocks/Materialized_view.md）。

> **注意**
>
> 只有在特定数据库上具有CREATE VIEW权限的用户才能执行此操作。

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

| **参数**        | **描述**                         |
| --------------- | ---------------------------------|
| OR REPLACE      | 替换现有视图。                  |
| database        | 视图所在数据库的名称。           |
| view_name       | 视图的名称。                     |
| column_name     | 视图中的列名。请注意，视图中的列和在`query_statement`中查询的列数量必须一致。 |
| COMMENT         | 视图中列或视图本身的注释。       |
| query_statement | 用于创建视图的查询语句。可以是StarRocks支持的任何查询语句。 |

## 使用注意事项

- 查询视图需要对视图和其对应的基本表具有SELECT权限。
- 如果用于构建视图的查询语句由于基本表的模式更改而无法执行，当您查询该视图时，StarRocks会返回错误。

## 示例

示例1：在`example_db`中创建名为`example_view`的视图，使用对`example_table`的聚合查询。

```SQL
CREATE VIEW example_db.example_view (k1, k2, k3, v1)
AS
SELECT c1 as k1, k2, k3, SUM(v1) FROM example_table
WHERE k1 = 20160112 GROUP BY k1, k2, k3;
```

示例2：在数据库`example_db`中创建名为`example_view`的视图，使用对表`example_table`的聚合查询，并为视图及其中的每列指定注释。

```SQL
CREATE VIEW example_db.example_view
(
    k1 COMMENT '第一个键',
    k2 COMMENT '第二个键',
    k3 COMMENT '第三个键',
    v1 COMMENT '第一个值'
)
COMMENT '我的第一个视图'
AS
SELECT c1 as k1, k2, k3, SUM(v1) FROM example_table
WHERE k1 = 20160112 GROUP BY k1, k2, k3;
```

## 相关SQL

- [SHOW CREATE VIEW](../data-manipulation/SHOW_CREATE_VIEW.md)
- [ALTER VIEW](./ALTER_VIEW.md)
- [DROP VIEW](./DROP_VIEW.md)