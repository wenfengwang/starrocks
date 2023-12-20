---
displayed_sidebar: English
---

# Base64解码字符串

## 描述

此函数功能与[from_base64](from_base64.md)相同，用于解码Base64编码的字符串，是[to_base64](to_base64.md)的反向操作。

该函数从v3.0版本开始支持。

## 语法

```Haskell
base64_decode_string(str);
```

## 参数

str：需要解码的字符串。它必须是VARCHAR类型。

## 返回值

返回VARCHAR类型的值。如果输入为NULL或不是有效的Base64字符串，将返回NULL。如果输入为空字符串，将报错。

此函数只接受单个字符串作为输入。输入多个字符串将会引发错误。

## 示例

```Plain

mysql> select base64_decode_string(to_base64("Hello StarRocks"));
+----------------------------------------------------+
| base64_decode_string(to_base64('Hello StarRocks')) |
+----------------------------------------------------+
| Hello StarRocks                                    |
+----------------------------------------------------+
```
