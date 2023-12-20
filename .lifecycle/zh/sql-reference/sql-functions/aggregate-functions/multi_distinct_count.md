---
displayed_sidebar: English
---

# 多重唯一计数

## 描述

返回表达式 expr 的不同值的总数，等同于 count(distinct expr)。

## 语法

```Haskell
multi_distinct_count(expr)
```

## 参数

expr：执行 multi_distinct_count() 函数时依据的列或表达式。如果 expr 是列名，那么这一列的数据类型可以是任意类型。

## 返回值

返回一个数值。如果没有找到任何行，则返回 0。此函数会忽略 NULL 值。

## 示例

假设存在一个名为 test 的表格。通过 id 查询每个订单的类别和供应商信息。

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

示例 1：统计类别列中不同值的个数。

```Plain
select multi_distinct_count(category) from test;
+--------------------------------+
| multi_distinct_count(category) |
+--------------------------------+
|                              4 |
+--------------------------------+
```

示例 2：统计供应商列中不同值的个数。

```Plain
select multi_distinct_count(supplier) from test;
+--------------------------------+
| multi_distinct_count(supplier) |
+--------------------------------+
|                              5 |
+--------------------------------+
```
