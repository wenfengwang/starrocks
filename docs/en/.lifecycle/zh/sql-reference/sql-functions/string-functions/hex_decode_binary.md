---
displayed_sidebar: English
---

# hex_decode_binary

## 描述

将十六进制编码的字符串解码为二进制。

此函数从 v3.0 版本开始支持。

## 语法

```Haskell
hex_decode_binary(str);
```

## 参数

`str`：要转换的字符串。支持的数据类型为 VARCHAR。

如果出现以下任一情况，则返回空的二进制文件：

- 字符串的长度为 0，或者字符串中的字符数为奇数。
- 该字符串包含除 `[0-9]`、`[a-z]` 和 `[A-Z]` 之外的字符。

## 返回值

返回 VARBINARY 类型的值。

## 例子

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

## 关键字

HEX_DECODE_BINARY
