---
displayed_sidebar: "Chinese"
---

# murmur_hash3_32

## 描述

返回输入字符串的32位murmur3哈希值。

## 语法

```Haskell
INT MURMUR_HASH3_32(VARCHAR input, ...)
```

## 示例

```Plain Text
MySQL > select murmur_hash3_32(null);
+-----------------------+
| murmur_hash3_32(NULL) |
+-----------------------+
|                  NULL |
+-----------------------+

MySQL > select murmur_hash3_32("hello");
+--------------------------+
| murmur_hash3_32('hello') |
+--------------------------+
|               1321743225 |
+--------------------------+

MySQL > select murmur_hash3_32("hello", "world");
+-----------------------------------+
| murmur_hash3_32('hello', 'world') |
+-----------------------------------+
|                         984713481 |
+-----------------------------------+
```

## 关键词

MURMUR_HASH3_32,哈希