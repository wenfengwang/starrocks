---
displayed_sidebar: English
---


# 计数

## 描述

返回由表达式指定的总行数。

此函数有三种变体：

- COUNT(*) 对表中的所有行进行计数，不论它们是否包含NULL值。

- COUNT(expr) 计算特定列中非NULL值的行数。

- COUNT(DISTINCT expr) 计算列中不重复的非NULL值的个数。

`COUNT(DISTINCT expr)` 用于精确统计不同的值。如果您需要更高效的不同值统计性能，请参考[使用位图进行精确统计](../../../using_starrocks/Using_bitmap.md)。

从StarRocks 2.4版本开始，您可以在一个语句中使用多个COUNT(DISTINCT)。

## 语法

```Haskell
COUNT(expr)
COUNT(DISTINCT expr [,expr,...])`
```

## 参数

expr：执行count()的依据列或表达式。如果expr是列名，那么该列可以是任意数据类型。

## 返回值

返回一个数值。如果没有找到任何行，则返回0。此函数会忽略NULL值。

## 示例

假设存在一个名为test的表。通过id查询每个订单的国家、分类和供应商。

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

示例1：统计test表中的行数。

```Plain
    select count(*) from test;
    +----------+
    | count(*) |
    +----------+
    |        7 |
    +----------+
```

示例2：统计id列中的值数量。

```Plain
    select count(id) from test;
    +-----------+
    | count(id) |
    +-----------+
    |         7 |
    +-----------+
```

示例3：在忽略NULL值的情况下，统计分类列中的值数量。

```Plain
select count(category) from test;
  +-----------------+
  | count(category) |
  +-----------------+
  |         6       |
  +-----------------+
```

示例4：统计分类列中不同值的数量。

```Plain
select count(distinct category) from test;
+-------------------------+
| count(DISTINCT category) |
+-------------------------+
|                       4 |
+-------------------------+
```

示例5：统计分类和供应商可以组成的不同组合数。

```Plain
select count(distinct category, supplier) from test;
+------------------------------------+
| count(DISTINCT category, supplier) |
+------------------------------------+
|                                  5 |
+------------------------------------+
```

在输出结果中，id为1004的组合与id为1002的组合重复，因此只计算一次。id为1007的组合包含NULL值，不被计算。

示例6：在一个语句中使用多个COUNT(DISTINCT)。

```Plain
select count(distinct country, category), count(distinct country,supplier) from test;
+-----------------------------------+-----------------------------------+
| count(DISTINCT country, category) | count(DISTINCT country, supplier) |
+-----------------------------------+-----------------------------------+
|                                 6 |                                 7 |
+-----------------------------------+-----------------------------------+
```
