---
displayed_sidebar: English
---

# 在集合中查找

## 描述

此函数返回字符串列表strlist中第一个str的位置（计数从1开始）。strlist是由逗号分隔的字符串。如果没有找到相应的str，它将返回0。当参数为NULL时，返回结果也为NULL。

## 语法

```Haskell
INT find_in_set(VARCHAR str, VARCHAR strlist)
```

## 示例

```Plain
MySQL > select find_in_set("b", "a,b,c");
+---------------------------+
| find_in_set('b', 'a,b,c') |
+---------------------------+
|                         2 |
+---------------------------+
```

## 关键字

FIND_IN_SET，查找，IN，集合
