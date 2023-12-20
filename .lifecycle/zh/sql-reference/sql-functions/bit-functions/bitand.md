---
displayed_sidebar: English
---

# 位与

## 描述

返回两个数值表达式的按位与结果。

## 语法

```Haskell
BITAND(x,y);
```

## 参数

- x：该表达式计算后必须是以下数据类型之一：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

- y：该表达式计算后必须是以下数据类型之一：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

> x 和 y 必须是相同的数据类型。

## 返回值

返回值的类型与 x 相同。如果任一值为 NULL，那么结果也为 NULL。

## 示例

```Plain
mysql> select bitand(3,0);
+--------------+
| bitand(3, 0) |
+--------------+
|            0 |
+--------------+
1 row in set (0.01 sec)
```
