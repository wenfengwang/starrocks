---
displayed_sidebar: English
---

# 从二进制转换

## 描述

此函数可以将二进制值根据指定的二进制格式（binary_type）转换成 VARCHAR 字符串。支持以下几种二进制格式：hex、encode64 和 utf8。如果没有指定 binary_type，默认使用 hex 格式。

## 语法

```Haskell
from_binary(binary[, binary_type])
```

## 参数

- binary：需要转换的输入二进制数据，必填项。

- binary_type：转换使用的二进制格式，选填项。

  - hex（默认）：from_binary 采用 hex 方法将输入的二进制数据编码成 VARCHAR 字符串。
  - encode64：from_binary 采用 base64 方法将输入的二进制数据编码成 VARCHAR 字符串。
  - utf8：from_binary 直接将输入的二进制数据转换成 VARCHAR 字符串，不进行任何转换。

## 返回值

返回一个 VARCHAR 类型的字符串。

## 示例

```Plain
mysql> select from_binary(to_binary('ABAB', 'hex'), 'hex');
+----------------------------------------------+
| from_binary(to_binary('ABAB', 'hex'), 'hex') |
+----------------------------------------------+
| ABAB                                         |
+----------------------------------------------+
1 row in set (0.02 sec)

mysql> select from_base64(from_binary(to_binary('U1RBUlJPQ0tT', 'encode64'), 'encode64'));
+-----------------------------------------------------------------------------+
| from_base64(from_binary(to_binary('U1RBUlJPQ0tT', 'encode64'), 'encode64')) |
+-----------------------------------------------------------------------------+
| STARROCKS                                                                   |
+-----------------------------------------------------------------------------+
1 row in set (0.01 sec)

mysql> select from_binary(to_binary('STARROCKS', 'utf8'), 'utf8');
+-----------------------------------------------------+
| from_binary(to_binary('STARROCKS', 'utf8'), 'utf8') |
+-----------------------------------------------------+
| STARROCKS                                           |
+-----------------------------------------------------+
1 row in set (0.01 sec)

```

## 参考资料

- [to_binary](to_binary.md)
- [BINARY/VARBINARY 数据类型](../../sql-statements/data-types/BINARY.md)
