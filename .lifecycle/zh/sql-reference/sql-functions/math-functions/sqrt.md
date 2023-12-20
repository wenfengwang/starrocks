---
displayed_sidebar: English
---

# 平方根，双精度平方根

## 描述

计算数值的平方根。dsqrt与sqrt功能相同。

## 语法

```Haskell
DOUBLE SQRT(DOUBLE x);
DOUBLE DSQRT(DOUBLE x);
```

## 参数

x：只能指定数值类型。该函数在计算前会将数值转换成DOUBLE类型。

## 返回值

返回DOUBLE数据类型的值。

## 使用须知

如果输入的是非数值类型，该函数将返回NULL。

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
