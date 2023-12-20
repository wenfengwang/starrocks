---
displayed_sidebar: English
---

# 重复

## 描述

此函数根据次数 count 重复字符串 str。当 count 小于 1 时，它返回一个空字符串。当 str 或 count 为 NULL 时，它返回 NULL。

## 语法

```Haskell
VARCHAR repeat(VARCHAR str, INT count)
```

## 示例

```Plain
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

## 关键字

REPEAT，
