---
displayed_sidebar: English
---

# 左函数

## 描述

该函数从字符串的左侧开始提取指定长度的字符。长度的单位是UTF-8字符。
注意：这个函数也被称作[left](left.md)。

## 语法

```SQL
VARCHAR strleft(VARCHAR str,INT len)
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

## 关键词

STRLEFT
