---
displayed_sidebar: English
---

# 分割

## 描述

此函数根据分隔符拆分给定的字符串，并在数组中返回拆分的部分。

## 语法

```SQL
ARRAY<VARCHAR> split(VARCHAR content, VARCHAR delimiter)
```

## 例子

```SQL
mysql> select split("a,b,c",",");
+---------------------+
| split('a,b,c', ',') |
+---------------------+
| ["a","b","c"]       |
+---------------------+

mysql> select split("a,b,c",",b,");
+-----------------------+
| split('a,b,c', ',b,') |
+-----------------------+
| ["a","c"]             |
+-----------------------+

mysql> select split("abc","");
+------------------+
| split('abc', '') |
+------------------+
| ["a","b","c"]    |
+------------------+
```

## 关键词

分割
