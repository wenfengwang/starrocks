---
displayed_sidebar: English
---

# 替换

## 描述

此函数将字符串中所有出现的特定字符替换为另一个字符串。在搜索模式时，此函数执行区分大小写的匹配。

该函数自v3.0版本起提供支持。

注意：在3.0版本之前，该函数的实现为[regexp_replace](../like-predicate-functions/regexp_replace.md)。

## 语法

```SQL
VARCHAR replace(VARCHAR str, VARCHAR pattern, VARCHAR repl)
```

## 参数

- str：原始字符串。

- pattern：要替换的字符。注意，这不是正则表达式。

- repl：用来替换pattern中字符的字符串。

## 返回值

返回一个替换了特定字符的字符串。

如果任一参数为NULL，结果也为NULL。

如果未找到匹配的字符，将返回原始字符串。

## 示例

```plain
-- Replace '.' in 'a.b.c' with '+'.

MySQL > SELECT replace('a.b.c', '.', '+');
+----------------------------+
| replace('a.b.c', '.', '+') |
+----------------------------+
| a+b+c                      |
+----------------------------+

-- No matching characters are found and the original string is returned.

MySQL > SELECT replace('a b c', '', '*');
+----------------------------+
| replace('a b c', '', '*') |
+----------------------------+
| a b c                      |
+----------------------------+

-- Replace 'like' with an empty string.

MySQL > SELECT replace('We like StarRocks', 'like', '');
+------------------------------------------+
| replace('We like StarRocks', 'like', '') |
+------------------------------------------+
| We  StarRocks                            |
+------------------------------------------+

-- No matching characters are found and the original string is returned.

MySQL > SELECT replace('He is awesome', 'handsome', 'smart');
+-----------------------------------------------+
| replace('He is awesome', 'handsome', 'smart') |
+-----------------------------------------------+
| He is awesome                                 |
+-----------------------------------------------+
```

## 关键字

REPLACE, replace
