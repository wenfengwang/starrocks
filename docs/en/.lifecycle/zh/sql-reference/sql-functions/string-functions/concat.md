---
displayed_sidebar: English
---

# concat

## 描述

此函数用于将多个字符串合并在一起。如果任何参数值为NULL，则返回值将为NULL。

## 语法

```Haskell
VARCHAR concat(VARCHAR,...)
```

## 例子

```Plain Text
MySQL > select concat("a", "b");
+------------------+
| concat('a', 'b') |
+------------------+
| ab               |
+------------------+

MySQL > select concat("a", "b", "c");
+-----------------------+
| concat('a', 'b', 'c') |
+-----------------------+
| abc                   |
+-----------------------+

MySQL > select concat("a", null, "c");
+------------------------+
| concat('a', NULL, 'c') |
+------------------------+
| NULL                   |
+------------------------+
```

## 关键词

CONCAT
