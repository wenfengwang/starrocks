---
displayed_sidebar: "Chinese"
---

# hex

## 描述

如果 `x` 是一个数值，此函数将返回该值的十六进制字符串表示。

如果 `x` 是一个字符串，此函数将返回该字符串的十六进制表示，其中字符串中的每个字符都被转换为两个十六进制数字。

## 语法

```Haskell
HEX(x);
```

## 参数

`x`: 要转换的字符串或数字。支持的数据类型为BIGINT、VARCHAR和VARBINARY（v3.0及更高版本）。

## 返回值

返回一个VARCHAR类型的值。

## 示例

```Plain Text
mysql> select hex(3);
+--------+
| hex(3) |
+--------+
| 3      |
+--------+
1 row in set (0.00 sec)

mysql> select hex('3');
+----------+
| hex('3') |
+----------+
| 33       |
+----------+
1 row in set (0.00 sec)

mysql> select hex('apple');
+--------------+
| hex('apple') |
+--------------+
| 6170706C65   |
+--------------+

-- 输入是一个二进制值。

mysql> select hex(x'abab');
+-------------+
| hex('ABAB') |
+-------------+
| ABAB        |
+-------------+
1 row in set (0.01 sec)
```

## 关键词

HEX