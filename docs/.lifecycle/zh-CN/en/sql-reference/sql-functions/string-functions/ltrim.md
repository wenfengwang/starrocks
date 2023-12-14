---
displayed_sidebar: "Chinese"
---

# ltrim

## 描述

从 `str` 参数的开头（左侧）移除前导空格或指定的字符。从 StarRocks 2.5.0 开始支持删除指定的字符。

## 语法

```Haskell
VARCHAR ltrim(VARCHAR str[, VARCHAR characters])
```

## 参数

`str`：必需，要修剪的字符串，必须计算为 VARCHAR 值。

`characters`：可选，要移除的字符，必须是 VARCHAR 值。如果未指定此参数，默认情况下会从字符串中移除空格。如果将此参数设为空字符串，会返回错误。

## 返回值

返回一个 VARCHAR 值。

## 示例

示例 1：去除字符串开头的空格。

```Plain Text
MySQL > SELECT ltrim('   ab d');
+------------------+
| ltrim('   ab d') |
+------------------+
| ab d             |
+------------------+
```

示例 2：从字符串开头移除指定的字符。

```Plain Text
MySQL > SELECT ltrim("xxabcdxx", "x");
+------------------------+
| ltrim('xxabcdxx', 'x') |
+------------------------+
| abcdxx                 |
+------------------------+
```

## 参考

- [trim（修剪）](trim.md)
- [rtrim（去除右侧字符）](rtrim.md)

## 关键词

LTRIM