---
displayed_sidebar: "Chinese"
---

# sqrt, dsqrt

## 描述

计算一个值的平方根。dsqrt与sqrt相同。

## 语法

```Haskell
DOUBLE SQRT(DOUBLE x);
DOUBLE DSQRT(DOUBLE x);
```

## 参数

`x`：您只能指定一个数值。此函数在计算之前将数值转换为DOUBLE值。

## 返回值

返回DOUBLE数据类型的值。

## 使用注意事项

如果您指定一个非数值，此函数将返回`NULL`。

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