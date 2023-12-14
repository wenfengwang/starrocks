---
displayed_sidebar: "Chinese"
---

# locate

## 描述

此函数用于查找字符串中子字符串的位置（从1开始计数，以字符为单位）。如果指定第三个参数 pos，则它将开始查找字符串中 substr 的位置，从 pos 之后的位置开始。如果找不到 str，则返回 0。

## 语法

```Haskell
INT locate(VARCHAR substr, VARCHAR str[, INT pos])
```

## 示例

```Plain Text
MySQL > SELECT LOCATE('bar', 'foobarbar');
+----------------------------+
| locate('bar', 'foobarbar') |
+----------------------------+
|                          4 |
+----------------------------+

MySQL > SELECT LOCATE('xbar', 'foobar');
+--------------------------+
| locate('xbar', 'foobar') |
+--------------------------+
|                        0 |
+--------------------------+

MySQL > SELECT LOCATE('bar', 'foobarbar', 5);
+-------------------------------+
| locate('bar', 'foobarbar', 5) |
+-------------------------------+
|                             7 |
+-------------------------------+
```

## 关键词

LOCATE