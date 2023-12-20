---
displayed_sidebar: English
---

# locate

## 描述

此函数用于查找字符串中子字符串的位置（从1开始计数，以字符为单位）。如果指定了第三个参数 pos，则从 pos 指定的位置开始向后查找子字符串 substr 的位置。如果找不到 substr，则返回0。

## 语法

```Haskell
INT locate(VARCHAR substr, VARCHAR str[, INT pos])
```

## 示例

```Plain
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

## 关键字

LOCATE