---
displayed_sidebar: English
---

# starts_with

## 描述

此函数在字符串以指定前缀开头时返回1。否则返回0。当参数为NULL时，结果为NULL。

## 语法

```Haskell
BOOLEAN starts_with(VARCHAR str, VARCHAR prefix)
```

## 例子

```Plain Text
mysql> select starts_with("hello world","hello");
+-------------------------------------+
|starts_with('hello world', 'hello')  |
+-------------------------------------+
| 1                                   |
+-------------------------------------+

mysql> select starts_with("hello world","world");
+-------------------------------------+
|starts_with('hello world', 'world')  |
+-------------------------------------+
| 0                                   |
+-------------------------------------+
```

## 关键词

START_WITH
