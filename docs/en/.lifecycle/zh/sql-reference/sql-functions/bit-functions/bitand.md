---
displayed_sidebar: English
---

# BITAND

## 描述

返回两个数值表达式的按位与结果。

## 语法

```Haskell
BITAND(x,y);
```

## 参数

- `x`：此表达式必须计算为以下数据类型之一：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

- `y`：此表达式必须计算为以下数据类型之一：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

> `x` 和 `y` 的数据类型必须相同。

## 返回值

返回值的类型与 `x` 相同。如果任一值为 NULL，则结果为 NULL。

## 示例

```Plain
mysql> select BITAND(3,0);
+--------------+
| BITAND(3, 0) |
+--------------+
|            0 |
+--------------+
1 row in set (0.01 sec)
```