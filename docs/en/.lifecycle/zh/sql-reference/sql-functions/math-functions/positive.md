---
displayed_sidebar: English
---

# positive

## 描述

返回 `x` 的正值。

## 语法

```Haskell
POSITIVE(x);
```

## 参数

`x`：支持 BIGINT、DOUBLE、DECIMALV2、DECIMAL32、DECIMAL64 和 DECIMAL128 数据类型。

## 返回值

返回一个数据类型与 `x` 的数据类型相同的正值。

## 示例

```Plain
mysql> select positive(3);
+-------------+
| positive(3) |
+-------------+
|           3 |
+-------------+
1 row in set (0.01 sec)

mysql> select positive(cast(3.14 as DECIMALV2));
+--------------------------------------+
| positive(CAST(3.14 AS DECIMAL(9,0))) |
+--------------------------------------+
|                                 3.14 |
+--------------------------------------+
1 row in set (0.01 sec)
```