---
displayed_sidebar: English
---

# 左侧

## 功能描述

该函数从一个指定的字符串**左侧**返回特定数量的字符。字符长度单位：UTF-8编码的字符。
注意：此函数也被命名为[strleft](strleft.md)。

## 语法结构

```SQL
VARCHAR left(VARCHAR str,INT len)
```

## 示例

```SQL
MySQL > select left("Hello starrocks",5);
+----------------------------+
| left('Hello starrocks', 5) |
+----------------------------+
| Hello                      |
+----------------------------+
```

## 关键字

LEFT
