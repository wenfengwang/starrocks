---
displayed_sidebar: "中文"
---

# 负数

## 描述

返回输入参数 `arg` 的负数。

## 语法

```Plain
negative(arg)
```

## 参数

`arg` 支持以下数据类型：

- BIGINT
- DOUBLE
- DECIMALV2
- DECIMAL32
- DECIMAL64
- DECIMAL128

## 返回值

返回与输入参数相同数据类型的值。

## 示例

```Plain
mysql> select negative(3);
+-------------+
| negative(3) |
+-------------+
|          -3 |
+-------------+
1 row in set (0.00 sec)

mysql> select negative(cast(3.14 as decimalv2));
+--------------------------------------+
| negative(CAST(3.14 AS DECIMAL(9,0))) |
+--------------------------------------+
|                                -3.14 |
+--------------------------------------+
1 row in set (0.01 sec)
```