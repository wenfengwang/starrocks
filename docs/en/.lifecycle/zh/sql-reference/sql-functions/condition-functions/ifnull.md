---
displayed_sidebar: English
---

# ifnull

## 描述

如果 `expr1` 为 NULL，则返回 `expr2`。如果 `expr1` 不为 NULL，则返回 `expr1`。

## 语法

```Haskell
ifnull(expr1, expr2);
```

## 参数

`expr1` 和 `expr2` 必须在数据类型上是兼容的。

## 返回值

返回值的类型与 `expr1` 相同。

## 示例

```Plain
mysql> select ifnull(2, 4);
+--------------+
| ifnull(2, 4) |
+--------------+
|            2 |
+--------------+

mysql> select ifnull(NULL, 2);
+-----------------+
| ifnull(NULL, 2) |
+-----------------+
|               2 |
+-----------------+
1 row in set (0.01 sec)
```