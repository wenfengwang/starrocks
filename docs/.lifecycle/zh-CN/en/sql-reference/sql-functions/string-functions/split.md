---
displayed_sidebar: "Chinese"
---

# split

## 描述

此函数根据分隔符拆分给定的字符串，并返回数组中的拆分部分。

## 语法

```SQL
ARRAY<VARCHAR> split(VARCHAR content, VARCHAR delimiter)
```

## 示例

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

SPLIT