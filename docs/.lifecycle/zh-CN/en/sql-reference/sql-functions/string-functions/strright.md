---
displayed_sidebar: "Chinese"
---

# strright

## 描述

此函数从字符串的右侧提取指定长度的字符（从右侧开始）。长度的单位为 utf-8 字符。
注意：该函数也被称为 [right](right.md)。

## 语法

```SQL
VARCHAR strright(VARCHAR str,INT len)
```

## 示例

```SQL
MySQL > select strright("Hello starrocks",9);
+--------------------------------+
| strright('Hello starrocks', 9) |
+--------------------------------+
| starrocks                      |
+--------------------------------+
```

## 关键词

STRRIGHT