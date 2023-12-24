---
displayed_sidebar: English
---

# IFNULL

## 描述

如果 `expr1` 为 NULL，则返回 `expr2`。如果 `expr1` 不为 NULL，则返回 `expr1`。

## 语法

```Haskell
ifnull(expr1, expr2);
```

## 参数

`expr1` 和 `expr2` 的数据类型必须兼容。

## 返回值

返回值的类型与 `expr1` 相同。

## 例子

```Plain Text
mysql> SELECT IFNULL(2, 4);
+--------------+
| IFNULL(2, 4) |
+--------------+
|            2 |
+--------------+

mysql> SELECT IFNULL(NULL, 2);
+-----------------+
| IFNULL(NULL, 2) |
+-----------------+
|               2 |
+-----------------+
1 行受影响 (0.01 秒)