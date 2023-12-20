---
displayed_sidebar: English
---

# left

## 描述

此函数返回给定字符串左侧指定数量的字符。长度单位：UTF-8字符。
注意：此函数也被称为[strleft](strleft.md)。

## 语法

```SQL
VARCHAR left(VARCHAR str, INT len)
```

## 示例

```SQL
MySQL > select left("Hello starrocks", 5);
+----------------------------+
| left('Hello starrocks', 5) |
+----------------------------+
| Hello                      |
+----------------------------+
```

## 关键字

LEFT