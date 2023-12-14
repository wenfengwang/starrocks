---
displayed_sidebar: "Chinese"
---

# xx_hash3_64

## 描述

返回输入字符串的64位xxhash3哈希值。xx_hash3_64通过使用AVX2指令具有比[murmur_hash3_32](./murmur_hash3_32.md)更好的性能，并且具有先进的哈希质量，广泛集成到许多软件中。

该函数从v3.2.0版本开始支持。

## 语法

```Haskell
BIGINT XX_HASH3_64(VARCHAR input, ...)
```

## 示例

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

XX_HASH3_64,HASH,xxHash3