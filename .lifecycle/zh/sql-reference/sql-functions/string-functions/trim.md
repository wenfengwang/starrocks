---
displayed_sidebar: English
---

# 修剪

## 描述

该函数用于移除 str 参数开头和结尾的连续空格或指定字符。从 StarRocks 2.5.0 版本开始，支持移除指定字符的功能。

## 语法

```Haskell
VARCHAR trim(VARCHAR str[, VARCHAR characters])
```

## 参数

str：必需，要进行修剪的字符串，返回值必须是 VARCHAR 类型。

characters：可选，要移除的字符，必须是 VARCHAR 类型。如果未指定此参数，默认会移除字符串中的空格。如果此参数被设置为空字符串，则会返回错误。

## 返回值

返回 VARCHAR 类型的值。

## 示例

示例 1：移除字符串开头和结尾的五个空格。

```Plain
MySQL > SELECT trim("   ab c  ");
+-------------------+
| trim('   ab c  ') |
+-------------------+
| ab c              |
+-------------------+
1 row in set (0.00 sec)
```

示例 2：移除字符串开头和结尾的指定字符。

```Plain
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

## 参考资料

- [ltrim](ltrim.md)
- [rtrim](rtrim.md)
