---
displayed_sidebar: English
---

# LTRIM

## 描述

该函数用于移除字符串参数 str 开头（左侧）的空白字符或指定字符。从 StarRocks 2.5.0 版本开始，支持移除指定字符。

## 语法

```Haskell
VARCHAR ltrim(VARCHAR str[, VARCHAR characters])
```

## 参数

str：必选，需要进行修剪的字符串，其计算结果必须是 VARCHAR 类型。

characters：可选，要移除的字符，必须是 VARCHAR 类型。如果未指定此参数，默认将移除字符串开头的空格。如果此参数被设置为空字符串，将返回错误。

## 返回值

返回 VARCHAR 类型的值。

## 示例

示例 1：移除字符串开头的空格。

```Plain
MySQL > SELECT ltrim('   ab d');
+------------------+
| ltrim('   ab d') |
+------------------+
| ab d             |
+------------------+
```

示例 2：移除字符串开头的指定字符。

```Plain
MySQL > SELECT ltrim("xxabcdxx", "x");
+------------------------+
| ltrim('xxabcdxx', 'x') |
+------------------------+
| abcdxx                 |
+------------------------+
```

## 参考资料

- [trim](trim.md)
- [rtrim](rtrim.md)

## 关键字

LTRIM
