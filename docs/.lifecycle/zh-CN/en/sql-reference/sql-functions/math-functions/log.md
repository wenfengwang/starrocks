---
displayed_sidebar: "Chinese"
---

# 对数

## 描述

计算一个数的对数到指定的底数（或基数）。如果未指定底数，则此函数等同于[ln](../math-functions/ln.md)。

## 语法

```SQL
log([base,] arg)
```

## 参数

- `base`：可选。底数。仅支持DOUBLE数据类型。如果未指定此参数，则此函数等同于[ln](../math-functions/ln.md)。

> **注意**
>
> 如果将 `base` 指定为负数、0 或 1，则StarRocks返回NULL。

- `arg`：要计算其对数的值。仅支持DOUBLE数据类型。

> **注意**
>
> 如果将 `arg` 指定为负数或0，则StarRocks返回NULL。

## 返回值

返回DOUBLE数据类型的值。

## 示例

示例 1：计算 8 的以2为底的对数。

```Plain
mysql> select log(2,8);
+-----------+
| log(2, 8) |
+-----------+
|         3 |
+-----------+
1 row in set (0.01 sec)
```

示例 2：计算 10 的对数（未指定底数，默认为*e*）。

```Plain
mysql> select log(10);
+-------------------+
| log(10)           |
+-------------------+
| 2.302585092994046 |
+-------------------+
1 row in set (0.09 sec)
```