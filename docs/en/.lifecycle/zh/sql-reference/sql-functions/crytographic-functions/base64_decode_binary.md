---
displayed_sidebar: English
---

# base64_decode_binary

## 描述

解码 Base64 编码的字符串并返回一个 BINARY 类型的值。

该函数从 v3.0 版本开始支持。

## 语法

```Haskell
base64_decode_binary(str);
```

## 参数

`str`：要解码的字符串。必须是 VARCHAR 类型。

## 返回值

返回一个 VARBINARY 类型的值。如果输入为 NULL 或不是有效的 Base64 字符串，则返回 NULL。如果输入为空，则返回错误。

此函数只接受一个字符串参数。输入多个字符串将导致错误。

## 示例

```Plain
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
```