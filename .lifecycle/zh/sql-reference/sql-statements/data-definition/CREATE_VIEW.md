---
displayed_sidebar: English
---

# 创建视图

## 描述

创建一个视图。

视图，或称逻辑视图，是一种虚拟表，其数据来源于对其他已存在的物理表进行查询。因此，视图不占用物理存储空间，对视图的所有查询都等同于构建视图时所用查询语句的子查询。

要了解StarRocks支持的物化视图的信息，请参见[Synchronous materialized views](../../../using_starrocks/Materialized_view-single_table.md)和[Asynchronous materialized views](../../../using_starrocks/Materialized_view.md)。

> **注意**
> 只有在特定数据库上拥有 **CREATE VIEW** 权限的用户才能执行此操作。

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

|参数|说明|
|---|---|
|或替换|替换现有视图。|
|database|视图所在的数据库的名称。|
|view_name|视图的名称。|
|column_name|视图中列的名称。请注意，视图中的列和query_statement中查询的列在数量上必须一致。|
|COMMENT|对视图中的列或视图本身的注释。|
|query_statement|用于创建视图的查询语句。可以是StarRocks支持的任何查询语句。|

## 使用须知

- 查询视图需要对视图及其对应基表有 SELECT 权限。
- 如果由于基表的结构变更导致构建视图的查询语句无法执行，当查询视图时 StarRocks 会返回错误。

## 示例

示例 1：在 example_db 数据库中使用针对 example_table 的聚合查询创建名为 example_view 的视图。

```SQL
CREATE VIEW example_db.example_view (k1, k2, k3, v1)
AS
SELECT c1 as k1, k2, k3, SUM(v1) FROM example_table
WHERE k1 = 20160112 GROUP BY k1,k2,k3;
```

示例 2：在 example_db 数据库中使用针对 example_table 的聚合查询创建名为 example_view 的视图，并为视图及其每一列指定注释。

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

## 相关 SQL 语句

- [显示创建视图](../data-manipulation/SHOW_CREATE_VIEW.md)
- [修改视图](./ALTER_VIEW.md)
- [DROP VIEW](./DROP_VIEW.md)
