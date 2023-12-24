---
displayed_sidebar: English
---

# 按位与

## 描述

返回两个数值表达式的按位与运算结果。

## 语法

```Haskell
BITAND(x,y);
```

## 参数

- `x`：此表达式必须求值为以下任何数据类型：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

- `y`：此表达式必须求值为以下任何数据类型：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

> `x` 和 `y` 的数据类型必须一致。

## 返回值

返回值与 `x` 的类型相同。如果任何值为 NULL，则结果也为 NULL。

## 例子

```Plain Text
mysql> select bitand(3,0);
+--------------+
| bitand(3, 0) |
+--------------+
|            0 |
+--------------+
1 row in set (0.01 sec)