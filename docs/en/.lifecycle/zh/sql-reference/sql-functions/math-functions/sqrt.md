---
displayed_sidebar: English
---

# sqrt, dsqrt

## 描述

计算一个值的平方根。dsqrt 与 sqrt 功能相同。

## 语法

```Haskell
DOUBLE SQRT(DOUBLE x);
DOUBLE DSQRT(DOUBLE x);
```

## 参数

`x`：只能指定数值。此函数会在计算前将数值转换成 DOUBLE 类型。

## 返回值

返回 DOUBLE 数据类型的值。

## 使用说明

如果指定的是非数值，则此函数返回 `NULL`。

## 示例

```Plain
mysql> select sqrt(3.14);
+-------------------+
| sqrt(3.14)        |
+-------------------+
| 1.772004514666935 |
+-------------------+
1 row in set (0.01 sec)


mysql> select dsqrt(3.14);
+-------------------+
| dsqrt(3.14)       |
+-------------------+
| 1.772004514666935 |
+-------------------+
```