---
displayed_sidebar: English
---

# 正则表达式替换

## 描述

该函数利用 repl 来替换字符串 str 中匹配正则表达式模式的字符序列。

## 语法

```Haskell
VARCHAR regexp_replace(VARCHAR str, VARCHAR pattern, VARCHAR repl)
```

## 示例

```Plain
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

REGEXP_REPLACE、REGEXP、REPLACE
