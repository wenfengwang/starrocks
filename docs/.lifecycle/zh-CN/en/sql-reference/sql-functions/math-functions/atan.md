---
displayed_sidebar: "Chinese"
---

# atan

## Description

计算参数的反正切。

## Syntax

```Haskell
DOUBLE atan(DOUBLE arg)
```

### Parameters

`arg`: 您只能指定一个数值。此函数在计算值的反正切之前将数值转换为DOUBLE值。

## Return value

返回DOUBLE数据类型的值。

## Usage notes

如果指定了非数值，此函数将返回`NULL`。

## Examples

```Plain
mysql> select atan(1);
+--------------------+
| atan(1)            |
+--------------------+
| 0.7853981633974483 |
+--------------------+

mysql> select atan(0);
+---------+
| atan(0) |
+---------+
|       0 |
+---------+

mysql> select atan(-1);
+---------------------+
| atan(-1)            |
+---------------------+
| -0.7853981633974483 |
+---------------------+

mysql> select atan("");
+----------+
| atan('') |
+----------+
|     NULL |
+----------+
```

## keyword

ATAN