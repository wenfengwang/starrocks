---
displayed_sidebar: English
---

# nullif

## 描述

如果 `expr1` 等于 `expr2`，则返回 NULL。否则，返回 `expr1`。

## 语法

```Haskell
nullif(expr1,expr2);
```

## 参数

`expr1` 和 `expr2` 必须在数据类型上是兼容的。

## 返回值

返回值的类型与 `expr1` 相同。

## 示例

```Plain
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
```