---
displayed_sidebar: English
---

# pmod

## 描述

返回`dividend`除以`divisor`的正余数。

## 语法

```SQL
pmod(dividend, divisor)
```

## 参数

- `dividend`：要进行除法运算的数字。
- `divisor`：用来除的数字。

`arg1`和`arg2`都支持以下数据类型：

- BIGINT
- DOUBLE

> **注意**
>
> `dividend`和`divisor`在数据类型上必须保持一致。如果它们的数据类型不一致，StarRocks会执行隐式转换。

## 返回值

返回与`dividend`相同数据类型的值。如果指定`divisor`为0，StarRocks会返回NULL。

## 例子

```Plain
mysql> select pmod(3.14,3.14);
+------------------+
| pmod(3.14, 3.14) |
+------------------+
|                0 |
+------------------+

mysql> select pmod(3,6);
+------------+
| pmod(3, 6) |
+------------+
|          3 |
+------------+

mysql> select pmod(11,5);
+-------------+
| pmod(11, 5) |
+-------------+
|           1 |
+-------------+

mysql> select pmod(-11,5);
+--------------+
| pmod(-11, 5) |
+--------------+
|            4 |
+--------------+

mysql> SELECT pmod(11,-5);
+--------------+
| pmod(11, -5) |
+--------------+
|           -4 |
+--------------+

mysql> SELECT pmod(-11,-5);
+---------------+
| pmod(-11, -5) |
+---------------+
|            -1 |
+---------------+
```
