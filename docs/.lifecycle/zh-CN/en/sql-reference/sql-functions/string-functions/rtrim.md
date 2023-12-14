---
displayed_sidebar: "Chinese"
---

# rtrim

## 描述

从`str`参数的末尾（右侧）删除尾随的空格或指定的字符。从 StarRocks 2.5.0 开始支持删除指定的字符。

## 语法

```Haskell
VARCHAR rtrim(VARCHAR str[, VARCHAR characters]);
```

## 参数

`str`: 必需，要修剪的字符串，它必须求值为 VARCHAR 值。

`characters`: 可选，要删除的字符，必须是 VARCHAR 值。如果未指定此参数，默认从字符串中删除空格。如果将此参数设置为空字符串，则返回错误。

## 返回值

返回一个 VARCHAR 值。

## 示例

示例 1：从字符串中删除末尾的三个空格。

```Plain Text
mysql> SELECT rtrim('   ab d   ');
+---------------------+
| rtrim('   ab d   ') |
+---------------------+
|    ab d             |
+---------------------+
1 row in set (0.00 sec)
```

示例 2：从字符串末尾删除指定的字符。

```Plain Text
MySQL > SELECT rtrim("xxabcdxx", "x");
+------------------------+
| rtrim('xxabcdxx', 'x') |
+------------------------+
| xxabcd                 |
+------------------------+
```

## 参考

- [trim](trim.md)
- [ltrim](ltrim.md)