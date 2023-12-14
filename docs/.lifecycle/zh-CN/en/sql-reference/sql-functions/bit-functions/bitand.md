---
displayed_sidebar: "Chinese"
---

# bitand

## 描述

返回两个数值表达式的按位AND运算结果。

## 语法

```Haskell
BITAND(x,y);
```

## 参数

- `x`: 此表达式必须求值为以下任一数据类型：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

- `y`: 此表达式必须求值为以下任一数据类型：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

> `x` 和 `y` 的数据类型必须相同。

## 返回值

返回值的类型与 `x` 相同。如果任一值为NULL，则结果为NULL。

## 示例

```Plain Text
mysql> select bitand(3,0);
+--------------+
| bitand(3, 0) |
+--------------+
|            0 |
+--------------+
1 行受影响 (0.01 秒)
```