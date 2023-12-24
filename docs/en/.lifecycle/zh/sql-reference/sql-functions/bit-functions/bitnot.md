---
displayed_sidebar: English
---

# 按位取反

## 描述

返回数值表达式的按位取反。

## 语法

```Haskell
BITNOT(x);
```

## 参数

`x`：此表达式必须计算为以下任何数据类型：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

## 返回值

返回值与 `x` 的类型相同。如果任何值为 NULL，则结果为 NULL。

## 例子

```Plain Text
mysql> select bitnot(3);
+-----------+
| bitnot(3) |
+-----------+
|        -4 |
+-----------+