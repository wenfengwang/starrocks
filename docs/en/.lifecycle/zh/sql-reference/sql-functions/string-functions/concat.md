---
displayed_sidebar: English
---

# concat

## 描述

此函数用于合并多个字符串。如果任何参数值为 NULL，则返回 NULL。

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