---
displayed_sidebar: English
---

# rtrim

## 描述

从 `str` 参数的末尾（右侧）移除尾随空格或指定字符。从 StarRocks 2.5.0 版本开始支持移除指定字符。

## 语法

```Haskell
VARCHAR rtrim(VARCHAR str[, VARCHAR characters]);
```

## 参数

`str`：必需，要修剪的字符串，必须能够计算为 VARCHAR 类型的值。

`characters`：可选，要移除的字符，必须是 VARCHAR 类型的值。如果未指定此参数，默认会移除字符串末尾的空格。如果此参数被设置为空字符串，将返回错误。

## 返回值

返回 VARCHAR 类型的值。

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

示例 2：移除字符串末尾的指定字符。

```Plain
MySQL > SELECT rtrim("xxabcdxx", "x");
+------------------------+
| rtrim('xxabcdxx', 'x') |
+------------------------+
| xxabcd                 |
+------------------------+
```

## 参考资料

- [trim](trim.md)
- [ltrim](ltrim.md)