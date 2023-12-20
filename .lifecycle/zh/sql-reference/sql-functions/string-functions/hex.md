---
displayed_sidebar: English
---

# 十六进制

## 说明

如果 x 是一个数值，该函数将返回该数值的十六进制字符串表示。

如果 x 是一个字符串，该函数将返回字符串的十六进制字符串表示，每个字符都将被转换成两位十六进制数。

## 语法

```Haskell
HEX(x);
```

## 参数

x：需要转换的字符串或数字。支持的数据类型包括 BIGINT、VARCHAR 和 VARBINARY（3.0 版本及以上）。

## 返回值

返回一个 VARCHAR 类型的值。

## 示例

```Plain
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

-- The input is a binary value.

mysql> select hex(x'abab');
+-------------+
| hex('ABAB') |
+-------------+
| ABAB        |
+-------------+
1 row in set (0.01 sec)
```

## 关键字

十六进制
