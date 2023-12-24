---
displayed_sidebar: English
---

# xx_hash3_64

## 描述

返回输入字符串的 64 位 xxhash3 哈希值。xx_hash3_64 通过使用 AVX2 指令比[murmur_hash3_32](./murmur_hash3_32.md)具有更好的性能，并且具有与许多软件广泛集成的最先进的哈希质量。

此功能从 v3.2.0 版本开始支持。

## 语法

```Haskell
BIGINT XX_HASH3_64(VARCHAR input, ...)
```

## 例子

```Plain Text
MySQL > select xx_hash3_64(null);
+-------------------+
| xx_hash3_64(NULL) |
+-------------------+
|              NULL |
+-------------------+

MySQL > select xx_hash3_64("hello");
+----------------------+
| xx_hash3_64('hello') |
+----------------------+
| -7685981735718036227 |
+----------------------+

MySQL > select xx_hash3_64("hello", "world");
+-------------------------------+
| xx_hash3_64('hello', 'world') |
+-------------------------------+
|           7001965798170371843 |
+-------------------------------+
```

## 关键词

XX_HASH3_64，哈希，xxHash3