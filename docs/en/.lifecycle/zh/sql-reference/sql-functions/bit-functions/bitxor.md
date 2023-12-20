---
displayed_sidebar: English
---


# BITXOR

## 描述

返回两个数值表达式的按位异或结果。

## 语法

```Haskell
BITXOR(x,y);
```

## 参数

- `x`：该表达式必须计算为以下数据类型之一：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

- `y`：该表达式必须计算为以下数据类型之一：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

> `x` 和 `y` 必须是相同的数据类型。

## 返回值

返回值的类型与 `x` 相同。如果任一值为 NULL，结果也为 NULL。

## 示例

```Plain
mysql> select bitxor(3,0);
+--------------+
| bitxor(3, 0) |
+--------------+
|            3 |
+--------------+
1 row in set (0.00 sec)
```