---
displayed_sidebar: English
---

# mod

## 描述

返回 `dividend` 除以 `divisor` 的余数的模函数。

## 语法

```SQL
mod(dividend, divisor)
```

## 参数

- `dividend`：要被除的数。
- `divisor`：除数。

`dividend` 和 `divisor` 都支持以下数据类型：

- TINYINT
- SMALLINT
- INT
- BIGINT
- LARGEINT
- FLOAT
- DOUBLE
- DECIMALV2
- DECIMAL32
- DECIMAL64
- DECIMAL128

> **注意**
> `dividend` 和 `divisor` 必须数据类型一致。如果它们的数据类型不一致，StarRocks 会执行隐式类型转换。

## 返回值

返回与 `dividend` 相同数据类型的值。如果 `divisor` 被指定为 0，StarRocks 返回 NULL。

## 示例

```Plain
mysql> select mod(3.14,3.14);
+-----------------+
| mod(3.14, 3.14) |
+-----------------+
|               0 |
+-----------------+

mysql> select mod(3.14, 3);
+--------------+
| mod(3.14, 3) |
+--------------+
|         0.14 |
+--------------+

select mod(11,-5);
+------------+
| mod(11, -5)|
+------------+
|          1 |
+------------+

select mod(-11,5);
+-------------+
| mod(-11, 5) |
+-------------+
|          -1 |
+-------------+
```