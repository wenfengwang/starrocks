---
displayed_sidebar: English
---

# 比特或

## 描述

返回两个数值表达式的按位或运算结果。

## 语法

```Haskell
BITOR(x,y);
```

## 参数

- x：此表达式必须计算为以下数据类型之一：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

- y：此表达式也必须计算为以下数据类型之一：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

> x 和 y 的数据类型必须相同。

## 返回值

返回值的类型与 x 相同。如果任何值为 NULL，则结果为 NULL。

## 示例

```Plain
mysql> select bitor(3,0);
+-------------+
| bitor(3, 0) |
+-------------+
|           3 |
+-------------+
1 row in set (0.00 sec)
```
