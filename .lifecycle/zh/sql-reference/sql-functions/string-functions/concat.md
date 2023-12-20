---
displayed_sidebar: English
---

# 拼接

## 描述

此函数用于拼接多个字符串。如果任何一个参数的值为 NULL，它将返回 NULL。

## 语法

```Haskell
VARCHAR concat(VARCHAR,...)
```

## 示例

```Plain
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

## 关键字

CONCAT
