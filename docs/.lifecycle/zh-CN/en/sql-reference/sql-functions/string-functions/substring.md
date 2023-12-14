---
displayed_sidebar: "Chinese"
---

# substring, substr

## 描述

从字符串中提取字符并返回一个子字符串。

如果未指定`len`，此函数将从指定`pos`位置提取字符。如果指定了`len`，此函数将从指定`pos`位置提取`len`个字符。

`pos`可以是负整数。在这种情况下，此函数将从字符串末尾开始提取字符。

## 语法

```Haskell
VARCHAR substr(VARCHAR str, pos[, len])
```

## 参数值

- `str`: 要提取字符的字符串，必需。它必须是一个VARCHAR值。
- `pos`: 起始位置，必需。字符串中的第一个位置为1。
- `length`: 要提取的字符数，可选。它必须是正整数。

## 返回值

返回一个VARCHAR类型的值。

如果要返回的字符数（`len`）超过了匹配字符的实际长度，则返回所有匹配的字符。

如果由`pos`指定的位置超出了字符串范围，则返回一个空字符串。

## 示例

```Plain Text
MySQL > select substring("starrockscluster", 1, 9);
+-------------------------------------+
| substring('starrockscluster', 1, 9) |
+-------------------------------------+
| starrocks                           |
+-------------------------------------+

MySQL > select substring("starrocks", -5, 5);
+-------------------------------+
| substring('starrocks', -5, 5) |
+-------------------------------+
| rocks                         |
+-------------------------------+

MySQL > select substring("apple", 1, 9);
+--------------------------+
| substring('apple', 1, 9) |
+--------------------------+
| apple                    |
+--------------------------+
```

## 关键词

substring, string, sub