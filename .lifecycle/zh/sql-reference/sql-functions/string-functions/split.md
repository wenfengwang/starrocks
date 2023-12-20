---
displayed_sidebar: English
---

# 拆分

## 描述

此函数根据分隔符将指定的字符串拆分，并以数组形式返回拆分后的各部分。

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

拆分
