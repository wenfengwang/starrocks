---
displayed_sidebar: English
---

# 替换

## 描述

将字符串中的所有指定字符替换为另一个字符串。此函数在搜索`pattern`时执行区分大小写的匹配。

该函数从v3.0版本开始支持。

注意：在3.0版本之前，此函数的实现为[regexp_replace](../like-predicate-functions/regexp_replace.md)。

## 语法

```SQL
VARCHAR replace(VARCHAR str, VARCHAR pattern, VARCHAR repl)
```

## 参数

- `str`：原始字符串。

- `pattern`：要替换的字符。注意，这不是正则表达式。

- `repl`：用来替换`pattern`中字符的字符串。

## 返回值

返回替换指定字符后的字符串。

如果任何参数为NULL，结果为NULL。

如果未找到匹配字符，则返回原始字符串。

## 示例

```plain
-- 将'a.b.c'中的'.'替换为'+'。

MySQL > SELECT replace('a.b.c', '.', '+');
+----------------------------+
| replace('a.b.c', '.', '+') |
+----------------------------+
| a+b+c                      |
+----------------------------+

-- 未找到匹配字符，返回原始字符串。

MySQL > SELECT replace('a b c', '', '*');
+----------------------------+
| replace('a b c', '', '*')  |
+----------------------------+
| a b c                      |
+----------------------------+

-- 将'like'替换为空字符串。

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

## 关键字

REPLACE, replace