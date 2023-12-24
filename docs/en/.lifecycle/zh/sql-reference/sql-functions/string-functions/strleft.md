---
displayed_sidebar: English
---

# strleft

## 描述

该函数从字符串中提取指定长度的字符（从左边开始）。长度的单位为utf8字符。
注意：该函数也被称为[left](left.md)。

## 语法

```SQL
VARCHAR strleft(VARCHAR str,INT len)
```

## 例子

```SQL
MySQL > select strleft("Hello starrocks",5);
+-------------------------------+
| strleft('Hello starrocks', 5) |
+-------------------------------+
| Hello                         |
+-------------------------------+
```

## 关键词

STRLEFT
