---
displayed_sidebar: "Chinese"
---

# replace

## 描述

用另一个字符串替换字符串中所有出现的字符。在搜索 `pattern` 时，该函数执行区分大小写的匹配。

此函数在 v3.0 及以上版本中受支持。

注意：在 3.0 之前，此函数实现为 [regexp_replace](../like-predicate-functions/regexp_replace.md)。

## 语法

```SQL
VARCHAR replace(VARCHAR str, VARCHAR pattern, VARCHAR repl)
```

## 参数

- `str`：原始字符串。

- `pattern`：要替换的字符。请注意，这不是正则表达式。

- `repl`：用于替换`pattern`中字符的字符串。

## 返回值

返回一个带有指定字符替换的字符串。

如果任何参数为 NULL，则结果为 NULL。

如果未找到匹配的字符，则返回原始字符串。

## 示例

```plain
-- 用 '+' 替换 'a.b.c' 中的 '.'。

MySQL > SELECT replace('a.b.c', '.', '+');
+----------------------------+
| replace('a.b.c', '.', '+') |
+----------------------------+
| a+b+c                      |
+----------------------------+

-- 未找到匹配字符，返回原始字符串。

MySQL > SELECT replace('a b c', '', '*');
+----------------------------+
| replace('a b c', '', '*') |
+----------------------------+
| a b c                      |
+----------------------------+

-- 用空字符串替换 'like'。

MySQL > SELECT replace('We like StarRocks', 'like', '');
+------------------------------------------+
| replace('We like StarRocks', 'like', '') |
+------------------------------------------+
| We  StarRocks                            |
+------------------------------------------+

-- 未找到匹配字符，返回原始字符串。

MySQL > SELECT replace('He is awesome', 'handsome', 'smart');
+-----------------------------------------------+
| replace('He is awesome', 'handsome', 'smart') |
+-----------------------------------------------+
| He is awesome                                 |
+-----------------------------------------------+
```

## 关键词

REPLACE, replace