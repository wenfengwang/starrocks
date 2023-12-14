---
displayed_sidebar: "Chinese"
---

# multi_distinct_count

## 描述

返回 `expr` 的总行数，相当于 count(distinct expr)。

## 语法

```Haskell
multi_distinct_count(expr)
```

## 参数

`expr`：进行 `multi_distinct_count()` 操作的列或表达式。如果 `expr` 是列名，则列可以是任何数据类型。

## 返回值

返回一个数值。如果找不到行，则返回 0。此函数会忽略 NULL 值。

## 示例

假设有一个名为 `test` 的表。查询每个订单的类别和供应商按 `id` 排序。

~~~Plain
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
~~~

示例 1：计算 `category` 列中不同值的数量。

~~~Plain
select multi_distinct_count(category) from test;
+--------------------------------+
| multi_distinct_count(category) |
+--------------------------------+
|                              4 |
+--------------------------------+
~~~

示例 2：计算 `supplier` 列中不同值的数量。

~~~Plain
select multi_distinct_count(supplier) from test;
+--------------------------------+
| multi_distinct_count(supplier) |
+--------------------------------+
|                              5 |
+--------------------------------+
~~~