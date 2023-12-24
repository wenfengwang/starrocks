---
displayed_sidebar: English
---

# lpad

## 描述

此函数返回字符串 `str` 的长度为 `len`（从第一个音节开始计数）。如果 `len` 长于 `str`，则在 `str` 前面添加填充字符，使返回值的长度延长为 `len` 字符。如果 `str` 长于 `len`，则返回值将被缩短为 `len` 字符。这里的 `len` 表示字符的长度，而不是字节的长度。

## 语法

```Haskell
VARCHAR lpad(VARCHAR str, INT len[, VARCHAR pad])
```

## 参数

`str`：必需，要进行填充的字符串，必须为 VARCHAR 值。

`len`：必需，返回值的长度，表示字符的长度，而不是字节的长度，必须为 INT 值。

`pad`：可选，要添加到 `str` 前面的字符，必须为 VARCHAR 值。如果未指定此参数，则默认添加空格。

## 返回值

返回一个 VARCHAR 值。

## 例子

```Plain Text
MySQL > SELECT lpad("hi", 5, "xy");
+---------------------+
| lpad('hi', 5, 'xy') |
+---------------------+
| xyxhi               |
+---------------------+

MySQL > SELECT lpad("hi", 1, "xy");
+---------------------+
| lpad('hi', 1, 'xy') |
+---------------------+
| h                   |
+---------------------+

MySQL > SELECT lpad("hi", 5);
+---------------------+
| lpad('hi', 5, ' ')  |
+---------------------+
|    hi               |
+---------------------+
```

## 关键词

LPAD
