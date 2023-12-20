---
displayed_sidebar: English
---

# trim

## 描述

从 `str` 参数的开头和结尾删除连续空格或指定字符。从 StarRocks 2.5.0 版本开始支持删除指定字符。

## 语法

```Haskell
VARCHAR trim(VARCHAR str[, VARCHAR characters])
```

## 参数

`str`：必需，要修剪的字符串，必须能够计算为 VARCHAR 类型的值。

`characters`：可选，要移除的字符，必须是 VARCHAR 类型的值。如果未指定此参数，则默认从字符串中移除空格。如果此参数被设置为空字符串，将返回错误。

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