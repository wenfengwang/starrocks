---
displayed_sidebar: English
---

# 十六进制解码字符串

## 描述

此函数执行的操作与[hex()](hex.md)相反。

它将输入字符串中的每对十六进制数字解读为一个数值，并将其转化为该数值所代表的字节。返回的值是一个二进制字符串。

此函数自v3.0版本起提供支持。

## 语法

```Haskell
hex_decode_string(str);
```

## 参数

str：待转换的字符串。支持的数据类型为VARCHAR。若出现以下任一情况，将返回空字符串：

- 字符串长度为0或字符串中的字符数为奇数。
- 字符串包含[0-9]、[a-z]和[A-Z]以外的字符。

## 返回值

返回的值为VARCHAR类型。

## 示例

```Plain
mysql> select hex_decode_string(hex("Hello StarRocks"));
+-------------------------------------------+
| hex_decode_string(hex('Hello StarRocks')) |
+-------------------------------------------+
| Hello StarRocks                           |
+-------------------------------------------+
```

## 关键字

HEX_DECODE_STRING
