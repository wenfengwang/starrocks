---
displayed_sidebar: English
---

# 转换函数

## 描述

将数字 x 从一个数制基底转换到另一个数制基底，并以字符串形式返回结果。

## 语法

```Haskell
CONV(x,y,z);
```

## 参数

- x：待转换的数字。支持 VARCHAR 或 BIGINT 类型。
- y：原数制基底。支持 TINYINT 类型。
- z：目标数制基底。支持 TINYINT 类型。

## 返回值

返回一个 VARCHAR 数据类型的值。

## 示例

将数字 8 从二进制数制转换为十进制数制：

```Plain
mysql> select conv(8,10,2);
+----------------+
| conv(8, 10, 2) |
+----------------+
| 1000           |
+----------------+
1 row in set (0.00 sec)
```
