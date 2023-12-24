---
displayed_sidebar: English
---

# LEFT

## 描述

此函数从给定字符串的左侧返回指定数量的字符。长度单位：utf8 字符。
注意：此函数也被称为 [strleft](strleft.md)。

## 语法

```SQL
VARCHAR left(VARCHAR str,INT len)
```

## 例子

```SQL
MySQL > select left("Hello starrocks",5);
+----------------------------+
| left('Hello starrocks', 5) |
+----------------------------+
| Hello                      |
+----------------------------+
```

## 关键词

LEFT
