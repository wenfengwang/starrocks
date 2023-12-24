---
displayed_sidebar: English
---

# ASCII

## 描述

此函数返回给定字符串最左侧字符的 ASCII 值。

## 语法

```Haskell
INT ascii(VARCHAR str)
```

## 例子

```Plain Text
MySQL > select ascii('1');
+------------+
| ascii('1') |
+------------+
|         49 |
+------------+

MySQL > select ascii('234');
+--------------+
| ascii('234') |
+--------------+
|           50 |
+--------------+
```

## 关键词

ASCII