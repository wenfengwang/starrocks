---
displayed_sidebar: English
---

# regexp_extract_all

## 描述

此函数返回目标值中所有与正则表达式模式匹配的子字符串。它提取在 pos 位置上与模式匹配的项。模式必须完全匹配 str 中的某些部分，这样函数才能返回模式中需要匹配的部分。如果未找到匹配项，则会返回空字符串。

## 语法

```Haskell
ARRAY<VARCHAR> regexp_extract_all(VARCHAR str, VARCHAR pattern, int pos)
```

## 示例

```Plain
MySQL > SELECT regexp_extract_all('AbCdE', '([[:lower:]]+)C([[:lower:]]+)', 1);
+-------------------------------------------------------------------+
| regexp_extract_all('AbCdE', '([[:lower:]]+)C([[:lower:]]+)', 1)   |
+-------------------------------------------------------------------+
| ['b']                                                             |
+-------------------------------------------------------------------+

MySQL > SELECT regexp_extract_all('AbCdExCeF', '([[:lower:]]+)C([[:lower:]]+)', 2);
+---------------------------------------------------------------------+
| regexp_extract_all('AbCdExCeF', '([[:lower:]]+)C([[:lower:]]+)', 2) |
+---------------------------------------------------------------------+
| ['d','e']                                                           |
+---------------------------------------------------------------------+
```

## 关键字

REGEXP_EXTRACT_ALL, REGEXP, EXTRACT