---
displayed_sidebar: English
---

# 修剪函数

## 描述

该函数用于移除 str 参数末尾（右侧）的尾随空格或指定字符。从 StarRocks 2.5.0 版本开始，支持移除指定字符的功能。

## 语法

```Haskell
VARCHAR rtrim(VARCHAR str[, VARCHAR characters]);
```

## 参数

str：必须提供，需要进行修剪的字符串，其求值结果必须为 VARCHAR 类型。

characters：可选，需要移除的字符，必须是 VARCHAR 类型。如果未提供此参数，默认会移除字符串末尾的空格。如果此参数被设置为空字符串，将会返回错误。

## 返回值

返回修剪后的 VARCHAR 类型的值。

## 示例

示例 1：移除字符串末尾的三个空格。

```Plain
mysql> SELECT rtrim('   ab d   ');
+---------------------+
| rtrim('   ab d   ') |
+---------------------+
|    ab d             |
+---------------------+
1 row in set (0.00 sec)
```

示例 2：移除字符串末尾的特定字符。

```Plain
MySQL > SELECT rtrim("xxabcdxx", "x");
+------------------------+
| rtrim('xxabcdxx', 'x') |
+------------------------+
| xxabcd                 |
+------------------------+
```

## 参考资料

- [trim](trim.md)函数
- [ltrim函数](ltrim.md)
