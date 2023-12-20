---
displayed_sidebar: English
---

# hex_decode_string

## 描述

此函数执行与 [hex()](hex.md) 相反的操作。

它将输入字符串中的每对十六进制数字解读为一个数字，并将其转换为该数字所代表的字节。返回值是一个二进制字符串。

此功能自 v3.0 起提供支持。

## 语法

```Haskell
hex_decode_string(str);
```

## 参数

`str`：要转换的字符串。支持的数据类型为 VARCHAR。如果出现以下任一情况，则返回空字符串：

- 字符串长度为 0 或字符串中的字符数为奇数。
- 字符串包含除 `[0-9]`、`[a-z]` 和 `[A-Z]` 之外的字符。

## 返回值

返回 VARCHAR 类型的值。

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