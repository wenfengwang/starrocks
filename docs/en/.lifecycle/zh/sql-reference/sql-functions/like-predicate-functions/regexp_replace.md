---
displayed_sidebar: English
---

# regexp_replace

## 描述

此函数使用 repl 来替换 str 中与正则表达式模式匹配的字符序列。

## 语法

```Haskell
VARCHAR regexp_replace(VARCHAR str, VARCHAR pattern, VARCHAR repl)
```

## 例子

```Plain Text
MySQL > SELECT regexp_replace('a b c', " ", "-");
+-----------------------------------+
| regexp_replace('a b c', ' ', '-') |
+-----------------------------------+
| a-b-c                             |
+-----------------------------------+

MySQL > SELECT regexp_replace('a b c','(b)','<\\1>');
+----------------------------------------+
| regexp_replace('a b c', '(b)', '<\1>') |
+----------------------------------------+
| a <b> c                                |
+----------------------------------------+
```

## 关键词

REGEXP_REPLACE, REGEXP, REPLACE