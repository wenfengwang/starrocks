---
displayed_sidebar: English
---

# 四舍五入函数的描述

## 描述

此函数能将数字四舍五入至指定的位数。若未指定n，则x将四舍五入至最接近的整数。若指定了n，则x将四舍五入至小数点后n位。若n为负数，则x将四舍五入至小数点左侧的位数。若发生溢出错误，则会返回错误信息。

## 语法

```Haskell
ROUND(x [,n]);
```

## 参数

x：支持DOUBLE和DECIMAL128数据类型。

n：支持INT数据类型，此参数为可选。

## 返回值

仅指定x时，返回值的数据类型如下：

["DECIMAL128"] -> "DECIMAL128"

["DOUBLE"] -> "BIGINT"

当x和n都被指定时，返回值的数据类型如下：

["DECIMAL128", "INT"] -> "DECIMAL128"

["DOUBLE", "INT"] -> "DOUBLE"

## 示例

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
