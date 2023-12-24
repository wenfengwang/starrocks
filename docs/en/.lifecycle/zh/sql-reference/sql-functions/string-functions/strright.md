---
displayed_sidebar: English
---

# strright

## 描述

该函数从指定长度（从右侧开始）的字符串中提取一定数量的字符。长度单位：utf-8字符。
注意：此函数也被命名为 [right](right.md)。

## 语法

```SQL
VARCHAR strright(VARCHAR str,INT len)
```

## 例子

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
