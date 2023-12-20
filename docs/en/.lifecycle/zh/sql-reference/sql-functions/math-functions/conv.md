---
displayed_sidebar: English
---

# conv

## 描述

将数字 `x` 从一个数值基数系统转换到另一个数值基数系统，并以字符串值的形式返回结果。

## 语法

```Haskell
CONV(x,y,z);
```

## 参数

- `x`：要转换的数字。支持 VARCHAR 或 BIGINT。
- `y`：源基数。支持 TINYINT。
- `z`：目标基数。支持 TINYINT。

## 返回值

返回一个 VARCHAR 数据类型的值。

## 示例

将数字 8 从二进制数制转换为十进制数制：

```Plain
mysql> select conv(8, 10, 2);
+----------------+
| conv(8, 10, 2) |
+----------------+
| 1000           |
+----------------+
1 row in set (0.00 sec)
```