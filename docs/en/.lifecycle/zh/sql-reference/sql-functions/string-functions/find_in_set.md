---
displayed_sidebar: English
---

# find_in_set

## 描述

此函数返回 str 在 strlist 中的位置（位置计数从 1 开始）。strlist 是由逗号分隔的字符串。如果没有找到 str，则返回 0。当参数为 NULL 时，结果也是 NULL。

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

FIND_IN_SET,FIND,IN,SET