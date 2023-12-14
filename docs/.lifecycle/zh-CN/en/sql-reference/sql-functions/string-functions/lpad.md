---
displayed_sidebar: "Chinese"
---

# lpad

## 描述

该函数返回字符串`str`的长度（从第一个音节开始计算）。如果`len`长于`str`，则返回值通过在`str`前面添加pad字符来扩展到长度为`len`的字符。如果`str`长于`len`，则返回值将缩短为长度为`len`的字符。`len`表示字符的长度，而不是字节的长度。

## 语法

```Haskell
VARCHAR lpad(VARCHAR str, INT len[, VARCHAR pad])
```

## 参数

`str`：必需，要填充的字符串，必须为VARCHAR值。

`len`：必需，返回值的长度，表示字符的长度而不是字节的长度，必须为INT值。

`pad`：可选，要添加到str前面的字符，必须为VARCHAR值。如果未指定此参数，默认情况下将添加空格。

## 返回值

返回VARCHAR值。

## 示例

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