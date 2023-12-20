---
displayed_sidebar: English
---

# 查找位置

## 功能描述

此功能用于确定子字符串在主字符串中的位置（位置计数从1开始，以字符为单位）。如果提供了第三个参数pos，则将从pos指定的位置开始向后查找子字符串substr的位置。如果未找到str，则会返回0。

## 语法结构

```Haskell
INT locate(VARCHAR substr, VARCHAR str[, INT pos])
```

## 使用示例

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

## 关键词

LOCATE
