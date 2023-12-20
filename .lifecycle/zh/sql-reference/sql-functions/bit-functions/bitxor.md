---
displayed_sidebar: English
---


# 位元异或

## 描述

返回两个数值表达式的位元异或结果。

## 语法

```Haskell
BITXOR(x,y);
```

## 参数

- x：该表达式的计算结果必须是以下数据类型之一：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

- y：该表达式的计算结果必须是以下数据类型之一：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

> x 和 y 必须是相同的数据类型。

## 返回值

返回值的类型与 x 的类型相同。如果任一值为 NULL，那么结果也为 NULL。

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
