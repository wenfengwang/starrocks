---
displayed_sidebar: English
---

# RPAD

## 描述

此函数返回`str`中从第一个音节开始计数的长度为`len`的字符串。如果`len`大于`str`，则通过在`str`后面添加填充字符将返回值延长为长度为`len`的字符串。如果`str`大于`len`，则返回值将缩短为长度为`len`的字符串。`len`表示字符的长度，而不是字节的长度。

## 语法

```Haskell
VARCHAR rpad(VARCHAR str, INT len[, VARCHAR pad])
```

## 参数

`str`：必需，要填充的字符串，必须计算为VARCHAR值。

`len`：必需，返回值的长度，表示字符的长度，而不是字节的长度，必须计算为INT值。

`pad`：可选，要在`str`后面添加的字符，必须是VARCHAR值。如果未指定此参数，默认添加空格。

## 返回值

返回VARCHAR值。

## 例子

```Plain Text
MySQL > SELECT rpad("hi", 5, "xy");
+---------------------+
| rpad('hi', 5, 'xy') |
+---------------------+
| hixyx               |
+---------------------+

MySQL > SELECT rpad("hi", 1, "xy");
+---------------------+
| rpad('hi', 1, 'xy') |
+---------------------+
| h                   |
+---------------------+

MySQL > SELECT rpad("hi", 5);
+---------------------+
| rpad('hi', 5, ' ')  |
+---------------------+
| hi                  |
+---------------------+
```

## 关键词

RPAD
