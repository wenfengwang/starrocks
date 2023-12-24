---
displayed_sidebar: English
---

# 重复

## 描述

此函数根据 `count` 参数重复多次 `str`。当 `count` 小于 1 时，它返回一个空字符串。当 `str` 或 `count` 为 NULL 时，它将返回 NULL。

## 语法

```Haskell
VARCHAR repeat(VARCHAR str, INT count)
```

## 例子

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