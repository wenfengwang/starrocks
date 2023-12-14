---
displayed_sidebar: "Chinese"
---

# 重复

## 描述

此函数根据`count`的值重复`str`。当`count`小于1时，返回空字符串。当`str`或`count`为空时，返回NULL。

## 语法

```Haskell
VARCHAR repeat(VARCHAR str, INT count)
```

## 示例

```Plain Text
MySQL > SELECT repeat("a", 3);
+----------------+
| repeat('a', 3) |
+----------------+
| aaa            |
+----------------+

MySQL > SELECT repeat("a", -1);
+-----------------+
| repeat('a', -1) |
+-----------------+
|                 |
+-----------------+
```

## 关键词

REPEAT