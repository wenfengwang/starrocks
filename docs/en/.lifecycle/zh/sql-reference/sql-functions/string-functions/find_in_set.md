---
displayed_sidebar: English
---

# find_in_set

## 描述

此函数返回 strlist 中第一个 str 的位置（从 1 开始计数）。Strlist 是用逗号分隔的字符串。如果找不到任何 str，则返回 0。当参数为 NULL 时，结果也为 NULL。

## 语法

```Haskell
INT find_in_set(VARCHAR str, VARCHAR strlist)
```

## 例子

```Plain Text
MySQL > select find_in_set("b", "a,b,c");
+---------------------------+
| find_in_set('b', 'a,b,c') |
+---------------------------+
|                         2 |
+---------------------------+
```

## 关键词

FIND_IN_SET, FIND, IN, SET