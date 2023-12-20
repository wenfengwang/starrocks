---
displayed_sidebar: English
---

# BITNOT

## 描述

返回数字表达式的按位取反结果。

## 语法

```Haskell
BITNOT(x);
```

## 参数

`x`：该表达式必须计算为以下数据类型之一：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

## 返回值

返回值的类型与 `x` 相同。如果任何值为 NULL，结果也为 NULL。

## 示例

```Plain
mysql> select BITNOT(3);
+-----------+
| BITNOT(3) |
+-----------+
|        -4 |
+-----------+
```