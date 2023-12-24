---
displayed_sidebar: English
---

# char_length

## 描述

此函数返回字符串的长度。对于多字节字符，它返回字符的数量。目前仅支持 utf8 编码。注意：此函数也被称为character_length。

## 语法

```Haskell
INT char_length(VARCHAR str)
```

## 例子

```Plain Text
MySQL > select char_length("abc");
+--------------------+
| char_length('abc') |
+--------------------+
|                  3 |
+--------------------+
```

## 关键词

CHAR_LENGTH，CHARACTER_LENGTH