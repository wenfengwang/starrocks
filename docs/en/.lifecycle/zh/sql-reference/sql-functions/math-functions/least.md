---
displayed_sidebar: English
---

# least

## 描述

返回一个或多个参数列表中的最小值。

通常，返回值与输入具有相同的数据类型。

比较规则与 [greatest](greatest.md) 函数相同。

## 语法

```Haskell
LEAST(expr1,...);
```

## 参数

`expr1`：要比较的表达式。它支持以下数据类型：

- SMALLINT

- TINYINT

- INT

- BIGINT

- LARGEINT

- FLOAT

- DOUBLE

- DECIMALV2

- DECIMAL32

- DECIMAL64

- DECIMAL128

- DATETIME

- VARCHAR

## 示例

示例 1：返回单个输入的最小值。

```Plain
select least(3);
+----------+
| least(3) |
+----------+
|        3 |
+----------+
1 row in set (0.00 sec)
```

示例 2：从一系列值中返回最小值。

```Plain
select least(3,4,5,5,6);
+----------------------+
| least(3, 4, 5, 5, 6) |
+----------------------+
|                    3 |
+----------------------+
1 row in set (0.01 sec)
```

示例 3：其中一个参数是 DOUBLE 类型，返回 DOUBLE 值。

```Plain
select least(4,4.5,5.5);
+--------------------+
| least(4, 4.5, 5.5) |
+--------------------+
|                4.0 |
+--------------------+
```

示例 4：输入参数是数字和字符串的混合，但字符串可以转换为数字。参数按数字进行比较。

```Plain
select least(7,'5');
+---------------+
| least(7, '5') |
+---------------+
| 5             |
+---------------+
1 row in set (0.01 sec)
```

示例 5：输入参数是数字和字符串的混合，但字符串不能转换为数字。参数按字符串进行比较。字符串 '1' 小于 'at'。

```Plain
select least(1,'at');
+----------------+
| least(1, 'at') |
+----------------+
| 1              |
+----------------+
```

示例 6：输入参数是字符。

```Plain
mysql> select least('A','B','Z');
+----------------------+
| least('A', 'B', 'Z') |
+----------------------+
| A                    |
+----------------------+
1 row in set (0.00 sec)
```

## 关键字

LEAST, least