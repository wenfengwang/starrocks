---
displayed_sidebar: English
---

# fmod

## 描述

返回除法（`dividend`/`divisor`）的浮点数余数。它是一个取模函数。

## 语法

```SQL
fmod(dividend, divisor);
```

## 参数

- `dividend`：支持 DOUBLE 或 FLOAT 类型。

- `divisor`：支持 DOUBLE 或 FLOAT 类型。

> **注意**
> `divisor`的数据类型需要与`dividend`的数据类型相同。否则，StarRocks将执行隐式转换以转换数据类型。

## 返回值

输出的数据类型和符号需要与`dividend`的数据类型和符号相同。如果`divisor`为`0`，则返回`NULL`。

## 示例

```Plaintext
mysql> select fmod(3.14,3.14);
+------------------+
| fmod(3.14, 3.14) |
+------------------+
|                0 |
+------------------+

mysql> select fmod(11.5,3);
+---------------+
| fmod(11.5, 3) |
+---------------+
|           2.5 |
+---------------+

mysql> select fmod(3,6);
+------------+
| fmod(3, 6) |
+------------+
|          3 |
+------------+

mysql> select fmod(3,0);
+------------+
| fmod(3, 0) |
+------------+
|       NULL |
+------------+
```