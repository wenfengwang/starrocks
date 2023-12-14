---
displayed_sidebar: "Chinese"
---


# bitxor

## 描述

返回两个数字表达式的按位异或运算结果。

## 语法

```Haskell
BITXOR(x,y);
```

## 参数

- `x`: 该表达式必须求值为以下任一数据类型：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

- `y`: 该表达式必须求值为以下任一数据类型：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

> `x` 和 `y` 的数据类型必须相同。

## 返回值

返回值的类型与 `x` 相同。如果任一值为 NULL，则结果为 NULL。

## 示例

```Plain Text
mysql> select bitxor(3,0);
+--------------+
| bitxor(3, 0) |
+--------------+
|            3 |
+--------------+
1 行受影响 (0.00 秒)
```