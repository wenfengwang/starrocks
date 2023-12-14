---
displayed_sidebar: "Chinese"
---

# base64_decode_binary

## 描述

将Base64编码的字符串解码并返回二进制值。

此功能从v3.0版本开始受支持。

## 语法

```Haskell
base64_decode_binary(str);
```

## 参数

`str`：要解码的字符串。必须为VARCHAR类型。

## 返回值

返回VARBINARY类型的值。如果输入为NULL或无效的Base64字符串，则返回NULL。如果输入为空，则返回错误。

此函数仅接受一个字符串。多个输入字符串将导致错误。

## 示例

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
```