---
displayed_sidebar: English
---

# 字符长度

## 说明

此函数用于返回字符串的长度。对于多字节字符，它返回的是字符的数量。目前只支持UTF-8编码。请注意：此函数也被称为character_length。

## 语法

```Haskell
INT char_length(VARCHAR str)
```

## 示例

```Plain
MySQL > select char_length("abc");
+--------------------+
| char_length('abc') |
+--------------------+
|                  3 |
+--------------------+
```

## 关键字

CHAR_LENGTH、CHARACTER_LENGTH
