---
displayed_sidebar: "Chinese"
---

# hex_decode_binary

## 描述

将十六进制编码的字符串解码为二进制。

此函数支持v3.0及以上版本。

## 语法

```Haskell
hex_decode_binary(str);
```

## 参数

`str`: 要转换的字符串。支持的数据类型为VARCHAR。

如果出现以下任何情况，则返回一个空的二进制：

- 字符串的长度为0或字符串中的字符数为奇数。
- 字符串包含除`[0-9]`、`[a-z]`和`[A-Z]`之外的字符。

## 返回值

返回VARBINARY类型的值。

## 示例

```Plain Text
mysql> select hex(hex_decode_binary(hex("Hello StarRocks")));
+------------------------------------------------+
| hex(hex_decode_binary(hex('Hello StarRocks'))) |
+------------------------------------------------+
| 48656C6C6F2053746172526F636B73                 |
+------------------------------------------------+

mysql> select hex_decode_binary(NULL);
+--------------------------------------------------+
| hex_decode_binary(NULL)                          |
+--------------------------------------------------+
| NULL                                             |
+--------------------------------------------------+
```

## 关键词

HEX_DECODE_BINARY