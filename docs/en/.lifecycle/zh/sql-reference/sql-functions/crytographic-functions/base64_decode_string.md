---
displayed_sidebar: English
---

# base64_decode_string

## 描述

此函数与 [from_base64](from_base64.md) 相同。
解码 Base64 编码的字符串，是 [to_base64](to_base64.md) 的逆操作。

该函数自 v3.0 起支持。

## 语法

```Haskell
base64_decode_string(str);
```

## 参数

`str`：要解码的字符串。必须是 VARCHAR 类型。

## 返回值

返回 VARCHAR 类型的值。如果输入为 NULL 或非法的 Base64 字符串，则返回 NULL。如果输入为空字符串，则返回错误。

此函数只接受一个字符串参数。输入多个字符串将导致错误。

## 示例

```Plain

mysql> select base64_decode_string(to_base64("Hello StarRocks"));
+----------------------------------------------------+
| base64_decode_string(to_base64('Hello StarRocks')) |
+----------------------------------------------------+
| Hello StarRocks                                    |
+----------------------------------------------------+
```