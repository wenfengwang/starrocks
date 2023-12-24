---
displayed_sidebar: English
---

# sqrt、dsqrt

## 描述

计算一个值的平方根。dsqrt 与 sqrt 相同。

## 语法

```Haskell
DOUBLE SQRT(DOUBLE x);
DOUBLE DSQRT(DOUBLE x);
```

## 参数

`x`：只能指定一个数值。此函数在计算前将数值转换为 DOUBLE 类型的值。

## 返回值

返回一个 DOUBLE 数据类型的值。

## 使用说明

如果指定一个非数值，此函数将返回 `NULL`。

## 例子

```Plain
mysql> select sqrt(3.14);
+-------------------+
| sqrt(3.14)        |
+-------------------+
| 1.772004514666935 |
+-------------------+
1 行受影响 (0.01 秒)


mysql> select dsqrt(3.14);
+-------------------+
| dsqrt(3.14)       |
+-------------------+
| 1.772004514666935 |
+-------------------+