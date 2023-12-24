---
displayed_sidebar: English
---

# 修剪

## 描述

从`str`参数的开头和结尾删除连续的空格或指定的字符。从 StarRocks 2.5.0 版本开始支持删除指定的字符。

## 语法

```Haskell
VARCHAR trim(VARCHAR str[, VARCHAR characters])
```

## 参数

`str`：必需，要修剪的字符串，必须计算为 VARCHAR 值。

`characters`：可选，要删除的字符，必须是 VARCHAR 值。如果未指定此参数，默认情况下会从字符串中删除空格。如果将此参数设置为空字符串，则会返回错误。

## 返回值

返回 VARCHAR 值。

## 例子

示例 1：删除字符串开头和结尾的五个空格。

```Plain Text
MySQL > SELECT trim("   ab c  ");
+-------------------+
| trim('   ab c  ') |
+-------------------+
| ab c              |
+-------------------+
1 row in set (0.00 sec)
```

示例 2：从字符串开头和结尾删除指定的字符。

```Plain Text
MySQL > SELECT trim("abcd", "ad");
+--------------------+
| trim('abcd', 'ad') |
+--------------------+
| bc                 |
+--------------------+

MySQL > SELECT trim("xxabcdxx", "x");
+-----------------------+
| trim('xxabcdxx', 'x') |
+-----------------------+
| abcd                  |
+-----------------------+
```

## 引用

- [ltrim](ltrim.md)
- [rtrim](rtrim.md)
