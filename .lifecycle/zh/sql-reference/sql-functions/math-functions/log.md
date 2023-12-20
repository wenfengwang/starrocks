---
displayed_sidebar: English
---

# 日志

## 描述

计算一个数以指定底数（或基数）的对数。如果未指定底数，此函数等同于自然对数[ln](../math-functions/ln.md)。

## 语法

```SQL
log([base,] arg)
```

## 参数

- `base`: 可选。The base. 仅支持**DOUBLE**数据类型。如果未指定此参数，函数等同于[ln](../math-functions/ln.md)。

> **注意**
> StarRocks返回NULL，如果`base`被指定为负数、0或1。

- arg：您想要计算对数的值。仅支持DOUBLE数据类型。

> **注意**
> StarRocks返回**NULL**如果`arg`被指定为负数或0。

## 返回值

返回DOUBLE数据类型的值。

## 示例

示例1：计算以2为底8的对数。

```Plain
mysql> select log(2,8);
+-----------+
| log(2, 8) |
+-----------+
|         3 |
+-----------+
1 row in set (0.01 sec)
```

示例2：计算以*e*为底（未指定底数）的10的对数。

```Plain
mysql> select log(10);
+-------------------+
| log(10)           |
+-------------------+
| 2.302585092994046 |
+-------------------+
1 row in set (0.09 sec)
```
