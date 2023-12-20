---
displayed_sidebar: English
---

# 正面

## 描述

返回变量 x 的值。

## 语法

```Haskell
POSITIVE(x);
```

## 参数

x：支持以下数据类型：BIGINT、DOUBLE、DECIMALV2、DECIMAL32、DECIMAL64、DECIMAL128。

## 返回值

返回一个与变量 x 数据类型相同的值。

## 示例

```Plain
mysql> select positive(3);
+-------------+
| positive(3) |
+-------------+
|           3 |
+-------------+
1 row in set (0.01 sec)

mysql> select positive(cast(3.14 as decimalv2));
+--------------------------------------+
| positive(CAST(3.14 AS DECIMAL(9,0))) |
+--------------------------------------+
|                                 3.14 |
+--------------------------------------+
1 row in set (0.01 sec)
```
