---
displayed_sidebar: English
---

# split_part

## 描述

此函数根据分隔符将给定字符串分割，并返回所请求的部分。（从第一部分开始计数）

## 语法

```Haskell
VARCHAR split_part(VARCHAR content, VARCHAR delimiter, INT field)
```

## 示例

```Plain
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
|split_part('hello world', ' ', -1)|
+----------------------------------+
| world                            |
+----------------------------------+

MySQL > select split_part("hello world", " ", -2);
+-----------------------------------+
| split_part('hello world', ' ', -2)|
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

## 关键字

SPLIT_PART, SPLIT, PART