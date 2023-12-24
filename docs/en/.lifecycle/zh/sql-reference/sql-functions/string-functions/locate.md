---
displayed_sidebar: English
---

# 定位

## 描述

此函数用于查找字符串中子字符串的位置（从 1 开始计数，以字符为单位）。如果指定了第三个参数 pos，它将从 pos 开始查找字符串中 substr 的位置。如果未找到 str，它将返回 0。

## 语法

```Haskell
INT locate(VARCHAR substr, VARCHAR str[, INT pos])
```

## 例子

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

定位
