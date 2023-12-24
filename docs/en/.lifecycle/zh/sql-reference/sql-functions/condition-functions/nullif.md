---
displayed_sidebar: English
---

# NULLIF

## 描述

如果 `expr1` 等于 `expr2`，则返回 NULL。否则，返回 `expr1`。

## 语法

```Haskell
nullif(expr1, expr2);
```

## 参数

`expr1` 和 `expr2` 的数据类型必须兼容。

## 返回值

返回值与 `expr1` 的类型相同。

## 例子

```Plain Text
mysql> select nullif(1,2);
+--------------+
| nullif(1, 2) |
+--------------+
|            1 |
+--------------+

mysql> select nullif(1,1);
+--------------+
| nullif(1, 1) |
+--------------+
|         NULL |
+--------------+