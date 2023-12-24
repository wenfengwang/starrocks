---
displayed_sidebar: English
---


# bitxor

## 描述

返回两个数值表达式的按位异或。

## 语法

```Haskell
BITXOR(x,y);
```

## 参数

- `x`：此表达式必须计算为以下任何数据类型：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

- `y`：此表达式必须计算为以下任何数据类型：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

> `x` 和 `y` 的数据类型必须一致。

## 返回值

返回值与 `x` 的类型相同。如果任何值为 NULL，则结果为 NULL。

## 例子

```Plain Text
mysql> select bitxor(3,0);
+--------------+
| bitxor(3, 0) |
+--------------+
|            3 |
+--------------+
1 行受影响 (0.00 秒)