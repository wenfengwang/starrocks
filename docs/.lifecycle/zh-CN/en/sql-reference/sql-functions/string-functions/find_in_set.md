---
displayed_sidebar: "Chinese"
---

# find_in_set

## 描述

此函数返回str在strlist中的第一个位置（从1开始计数）。strlist是用逗号分隔的字符串。如果未找到任何str，则返回0。当参数为NULL时，结果为NULL。

## 语法

```Haskell
INT find_in_set(VARCHAR str, VARCHAR strlist)
```

## 示例

```Plain Text
MySQL > select find_in_set("b", "a,b,c");
+---------------------------+
| find_in_set('b', 'a,b,c') |
+---------------------------+
|                         2 |
+---------------------------+
```

## 关键词

FIND_IN_SET,FIND,IN,SET