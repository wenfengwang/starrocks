---
displayed_sidebar: "Chinese"
---

# base64_decode_string

## 描述

此函数与[from_base64](from_base64.md)相同。
解码Base64编码的字符串。并且是[to_base64](to_base64.md)的逆操作。

此功能从v3.0开始支持。

## 语法

```Haskell
base64_decode_string(str);
```

## 参数

`str`: 要解码的字符串。它必须是VARCHAR类型。

## 返回值

返回VARCHAR类型的值。如果输入为NULL或无效的Base64字符串，则返回NULL。如果输入为空，则返回错误。

此函数仅接受一个字符串。多于一个输入字符串将导致错误。

## 示例

```Plain Text

mysql> select base64_decode_string(to_base64("Hello StarRocks"));
+----------------------------------------------------+
| base64_decode_string(to_base64('Hello StarRocks')) |
+----------------------------------------------------+
| Hello StarRocks                                    |
+----------------------------------------------------+
```