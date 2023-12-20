---
displayed_sidebar: English
---

# 修正

## 描述

此函数用于从字符串的右侧开始提取指定长度的字符。长度的单位为：UTF-8字符。
注意：此函数亦可称作[right](right.md)。

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

## 关键字

STRRIGHT
