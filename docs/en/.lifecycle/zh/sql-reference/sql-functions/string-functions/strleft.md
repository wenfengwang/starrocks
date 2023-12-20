---
displayed_sidebar: English
---

# strleft

## 描述

此函数用于从字符串中提取指定长度的字符（从左侧开始）。长度单位：utf8字符。
注意：此函数亦称为[left](left.md)。

## 语法

```SQL
VARCHAR strleft(VARCHAR str, INT len)
```

## 示例

```SQL
MySQL > select strleft("Hello starrocks",5);
+-------------------------------+
| strleft('Hello starrocks', 5) |
+-------------------------------+
| Hello                         |
+-------------------------------+
```

## 关键字

STRLEFT