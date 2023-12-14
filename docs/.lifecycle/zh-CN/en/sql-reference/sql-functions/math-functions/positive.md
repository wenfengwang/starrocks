---
displayed_sidebar: "英文"
---

# positive

## 描述

返回 `x` 作为一个值。

## 语法

```Haskell
POSITIVE(x);
```

## 参数

`x`：支持 BIGINT、DOUBLE、DECIMALV2、DECIMAL32、DECIMAL64 和 DECIMAL128 数据类型。

## 返回值

返回一个值，其数据类型与 `x` 值的数据类型相同。

## 示例

```Plain
mysql> select positive(3);
+-------------+
| positive(3) |
+-------------+
|           3 |
+-------------+
1 行在集 (0.01 秒)

mysql> select positive(cast(3.14 as decimalv2));
+--------------------------------------+
| positive(CAST(3.14 AS DECIMAL(9,0))) |
+--------------------------------------+
|                                 3.14 |
+--------------------------------------+
1 行在集 (0.01 秒)
```