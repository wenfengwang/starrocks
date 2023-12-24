---
displayed_sidebar: English
---

# base64_decode_binary

## 描述

解码 Base64 编码的字符串并返回 BINARY。

此功能从 v3.0 版本开始支持。

## 语法

```Haskell
base64_decode_binary(str);
```

## 参数

`str`：要解码的字符串。它必须是 VARCHAR 类型。

## 返回值

返回 VARBINARY 类型的值。如果输入为 NULL 或无效的 Base64 字符串，则返回 NULL。如果输入为空，则会返回错误。

此函数只接受一个字符串。输入多个字符串将导致错误。

## 例子

```Plain Text
mysql> select hex(base64_decode_binary(to_base64("Hello StarRocks")));
+---------------------------------------------------------+
| hex(base64_decode_binary(to_base64('Hello StarRocks'))) |
+---------------------------------------------------------+
| 48656C6C6F2053746172526F636B73                          |
+---------------------------------------------------------+

mysql> select base64_decode_binary(NULL);
+--------------------------------------------------------+
| base64_decode_binary(NULL)                             |
+--------------------------------------------------------+
| NULL                                                   |
+--------------------------------------------------------+