---
displayed_sidebar: English
---

# 子串（substring）、子串（substr）

## 描述

从指定位置开始提取字符，并返回一个确定长度的子字符串。

## 语法

```Haskell
VARCHAR substr(VARCHAR str, pos[, len])
```

## 参数

- str：必需，要从中提取字符的字符串。必须是 VARCHAR 类型的值。
- pos：必需，一个整数，用于指定起始位置。注意，字符串中的第一个字符索引是 1，不是 0。
  - 如果 pos 为 0，则返回一个空字符串。
  - pos 可以是一个负整数。在这种情况下，函数将从字符串的末尾开始提取字符。参见示例 2。
  - 如果 pos 指定的位置超过了字符串的范围，则返回一个空字符串。参见示例 3。
- len：可选，一个正整数，用于指定要提取的字符数量。
  - 如果指定了 len，则该函数将从 pos 指定的位置开始提取 len 个字符。
  - 如果没有指定 len，则该函数将提取从 pos 位置开始的所有字符。参见示例 1。
  - 如果 len 超过了实际匹配字符的长度，将返回所有匹配字符。参见示例 4。

## 返回值

返回一个 VARCHAR 类型的值。

## 示例

```Plain
-- Extract all characters starting from the first character "s".
MySQL > select substring("starrockscluster", 1);
+-------------------------------------+
| substring('starrockscluster', 1) |
+-------------------------------------+
| starrocks                           |
+-------------------------------------+

-- The position is negative and the counting is from the end of the string.
MySQL > select substring("starrocks", -5, 5);
+-------------------------------+
| substring('starrocks', -5, 5) |
+-------------------------------+
| rocks                         |
+-------------------------------+

-- The position exceeds the length of the string and an empty string is returned.
MySQL > select substring("apple", 8, 2);
+--------------------------------+
| substring('apple', 8, 2)       |
+--------------------------------+
|                                |
+--------------------------------+

-- There are 5 matching characters. The length 9 exceeds the length of the matching characters and all the matching characters are returned.
MySQL > select substring("apple", 1, 9);
+--------------------------+
| substring('apple', 1, 9) |
+--------------------------+
| apple                    |
+--------------------------+
```

## 关键字

substring、字符串、sub
