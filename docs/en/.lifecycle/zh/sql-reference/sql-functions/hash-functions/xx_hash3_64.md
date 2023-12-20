---
displayed_sidebar: English
---

# xx_hash3_64

## 描述

返回输入字符串的 64 位 xxhash3 哈希值。xx_hash3_64 通过使用 AVX2 指令，相比 [murmur_hash3_32](./murmur_hash3_32.md) 有更好的性能，并且具有顶尖的哈希质量，已被广泛集成到许多软件中。

该函数从 v3.2.0 版本开始支持。

## 语法

```Haskell
BIGINT XX_HASH3_64(VARCHAR input, ...)
```

## 示例

```Plain
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

## 关键字

XX_HASH3_64, HASH, xxHash3