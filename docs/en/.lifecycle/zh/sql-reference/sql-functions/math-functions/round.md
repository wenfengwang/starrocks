---
displayed_sidebar: English
---

# ROUND, DROUND

## 描述

将数字四舍五入到指定的位数。如果未指定 `n`，`x` 则四舍五入为最接近的整数。如果指定了 `n`，`x` 则四舍五入到 `n` 小数位。如果 `n` 为负数，`x` 则四舍五入到小数点左侧。如果发生溢出，则返回错误。

## 语法

```Haskell
ROUND(x [,n]);
```

## 参数

`x`：支持 DOUBLE 和 DECIMAL128 数据类型。

`n`：支持 INT 数据类型。此参数是可选的。

## 返回值

如果只指定了 `x`，则返回值的数据类型如下：

["DECIMAL128"] -> "DECIMAL128"

["DOUBLE"] -> "BIGINT"

如果同时指定了 `x` 和 `n`，则返回值的数据类型如下：

["DECIMAL128", "INT"] -> "DECIMAL128"

["DOUBLE", "INT"] -> "DOUBLE"

## 例子

```Plain
mysql> select round(3.14);
+-------------+
| round(3.14) |
+-------------+
|           3 |
+-------------+
1 row in set (0.00 sec)

mysql> select round(3.14,1);
+----------------+
| round(3.14, 1) |
+----------------+
|            3.1 |
+----------------+
1 row in set (0.00 sec)

mysql> select round(13.14,-1);
+------------------+
| round(13.14, -1) |
+------------------+
|               10 |
+------------------+
1 row in set (0.00 sec)
```
