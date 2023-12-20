---
displayed_sidebar: English
---

# HEX

## 描述

如果 `x` 是数值，此函数返回该值的十六进制字符串表示形式。

如果 `x` 是字符串，此函数返回字符串的十六进制字符串表示形式，其中字符串中的每个字符都转换成两个十六进制数字。

## 语法

```Haskell
HEX(x);
```

## 参数

`x`：要转换的字符串或数字。支持的数据类型包括 BIGINT、VARCHAR 和 VARBINARY（v3.0 及以后版本）。

## 返回值

返回 VARCHAR 类型的值。

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

-- 输入是二进制值。

mysql> select hex(x'abab');
+-------------+
| hex('ABAB') |
+-------------+
| ABAB        |
+-------------+
1 row in set (0.01 sec)
```

## 关键字

HEX