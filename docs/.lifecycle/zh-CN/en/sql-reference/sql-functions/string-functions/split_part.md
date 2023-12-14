---
displayed_sidebar: "Chinese"
---

# split_part

## Description

此函数根据分隔符拆分给定的字符串，并返回请求的部分。(从开头开始计数)

## Syntax

```Haskell
VARCHAR split_part(VARCHAR content, VARCHAR delimiter, INT field)
```

## Examples

```Plain Text
MySQL > select split_part("hello world", " ", 1);
+----------------------------------+
|split_part('hello world', ' ', 1) |
+----------------------------------+
| hello                            |
+----------------------------------+

MySQL > select split_part("hello world", " ", 2);
+-----------------------------------+
| split_part('hello world', ' ', 2) |
+-----------------------------------+
| world                             |
+-----------------------------------+

MySQL > select split_part("hello world", " ", -1);
+----------------------------------+
|split_part('hello world', ' ', -1) |
+----------------------------------+
| world                            |
+----------------------------------+

MySQL > select split_part("hello world", " ", -2);
+-----------------------------------+
| split_part('hello world', ' ', -2) |
+-----------------------------------+
| hello                             |
+-----------------------------------+

MySQL > select split_part("abca", "a", 1);
+----------------------------+
| split_part('abca', 'a', 1) |
+----------------------------+
|                            |
+----------------------------+

select split_part("abca", "a", -1);
+-----------------------------+
| split_part('abca', 'a', -1) |
+-----------------------------+
|                             |
+-----------------------------+

select split_part("abca", "a", -2);
+-----------------------------+
| split_part('abca', 'a', -2) |
+-----------------------------+
| bc                          |
+-----------------------------+
```

## keyword

SPLIT_PART,SPLIT,PART