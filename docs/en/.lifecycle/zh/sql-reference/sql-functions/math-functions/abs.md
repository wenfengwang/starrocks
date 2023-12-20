---
displayed_sidebar: English
---

# abs

## 描述

返回数值 `x` 的绝对值。如果输入值为 NULL，则返回 NULL。

## 语法

```Haskell
ABS(x);
```

## 参数

`x`：数值或表达式。

支持的数据类型：DOUBLE、FLOAT、LARGEINT、BIGINT、INT、SMALLINT、TINYINT、DECIMALV2、DECIMAL32、DECIMAL64、DECIMAL128。

## 返回值

返回值的数据类型与 `x` 的类型相同。

## 示例

```Plain
mysql> select abs(-1);
+---------+
| abs(-1) |
+---------+
|       1 |
+---------+
1 row in set (0.00 sec)
```

## 关键字

abs、absolute