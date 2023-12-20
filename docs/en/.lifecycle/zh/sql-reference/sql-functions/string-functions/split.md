---
displayed_sidebar: English
---

# split

## 描述

此函数根据分隔符将给定字符串分割，并以 ARRAY 形式返回分割后的各部分。

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

## 关键字

SPLIT