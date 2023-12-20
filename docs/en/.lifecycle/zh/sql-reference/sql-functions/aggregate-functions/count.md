---
displayed_sidebar: English
---


# 计数

## 描述

返回由表达式指定的总行数。

此函数有三种变体：

- `COUNT(*)` 对表中的所有行进行计数，不论它们是否包含 NULL 值。

- `COUNT(expr)` 计算特定列中非 NULL 值的行数。

- `COUNT(DISTINCT expr)` 计算列中不同的非 NULL 值的数量。

`COUNT(DISTINCT expr)` 用于精确计算不同的值。如果您需要更高效的精确计数性能，请参考[使用 bitmap 进行精确计数](../../../using_starrocks/Using_bitmap.md)。

从 StarRocks 2.4 版本开始，您可以在一个语句中使用多个 COUNT(DISTINCT)。

## 语法

```Haskell
COUNT(expr)
COUNT(DISTINCT expr [,expr,...])
```

## 参数

`expr`：执行 `count()` 所基于的列或表达式。如果 `expr` 是列名，该列可以是任何数据类型。

## 返回值

返回一个数值。如果没有找到行，则返回 0。此函数忽略 NULL 值。

## 示例

假设有一个名为 `test` 的表。通过 `id` 查询每个订单的国家、类别和供应商。

```Plain
select * from test order by id;
+------+----------+----------+------------+
| id   | country  | category | supplier   |
+------+----------+----------+------------+
| 1001 | US       | A        | supplier_1 |
| 1002 | Thailand | A        | supplier_2 |
| 1003 | Turkey   | B        | supplier_3 |
| 1004 | US       | A        | supplier_2 |
| 1005 | China    | C        | supplier_4 |
| 1006 | Japan    | D        | supplier_3 |
| 1007 | Japan    | NULL     | supplier_5 |
+------+----------+----------+------------+
```

示例 1：统计 `test` 表中的行数。

```Plain
    select count(*) from test;
    +----------+
    | count(*) |
    +----------+
    |        7 |
    +----------+
```

示例 2：计算 `id` 列中的值数量。

```Plain
    select count(id) from test;
    +-----------+
    | count(id) |
    +-----------+
    |         7 |
    +-----------+
```

示例 3：计算 `category` 列中的值数量，同时忽略 NULL 值。

```Plain
select count(category) from test;
  +-----------------+
  | count(category) |
  +-----------------+
  |               6 |
  +-----------------+
```

示例 4：计算 `category` 列中不同值的数量。

```Plain
select count(distinct category) from test;
+-------------------------+
| count(DISTINCT category) |
+-------------------------+
|                        4 |
+-------------------------+
```

示例 5：统计 `category` 和 `supplier` 可以形成的组合数量。

```Plain
select count(distinct category, supplier) from test;
+------------------------------------+
| count(DISTINCT category, supplier) |
+------------------------------------+
|                                  5 |
+------------------------------------+
```

在输出中，`id` 为 1004 的组合与 `id` 为 1002 的组合重复，它们只被计数一次。`id` 为 1007 的组合因为包含 NULL 值而不被计数。

示例 6：在一个语句中使用多个 COUNT(DISTINCT)。

```Plain
select count(distinct country, category), count(distinct country,supplier) from test;
+-----------------------------------+-----------------------------------+
| count(DISTINCT country, category) | count(DISTINCT country, supplier) |
+-----------------------------------+-----------------------------------+
|                                 6 |                                 7 |
+-----------------------------------+-----------------------------------+
```