---
displayed_sidebar: "Chinese"
---

# cosh

## Description

计算参数的双曲余弦。

此函数支持从v3.0开始。

## Syntax

```Haskell
DOUBLE cosh(DOUBLE arg)
```

### Parameters

`arg`: 您只能指定一个数值。此函数在计算数值的双曲余弦值之前将数值转换为DOUBLE值。

## Return value

返回DOUBLE数据类型的值。

## Usage notes

如果您指定一个非数值的值，则此函数返回 `NULL`。

## Examples

```Plain
mysql> select cosh(-1);
+--------------------+
| cosh(-1)           |
+--------------------+
| 1.5430806348152437 |
+--------------------+

mysql> select cosh(0);
+---------+
| cosh(0) |
+---------+
|       1 |
+---------+

mysql> select cosh(1);
+--------------------+
| cosh(1)            |
+--------------------+
| 1.5430806348152437 |
+--------------------+

mysql> select cosh("");
+----------+
| cosh('') |
+----------+
|     NULL |
+----------+
```

## keyword

COSH