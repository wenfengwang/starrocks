---
displayed_sidebar: English
---

# xx_hash3_64

## 描述

返回输入字符串的64位xxhash3哈希值。xx_hash3_64利用AVX2指令，相较于[murmur_hash3_32](./murmur_hash3_32.md)拥有更优的性能，并且提供顶尖的哈希品质，已广泛应用于众多软件中。

该函数自v3.2.0版本起提供支持。

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

XX_HASH3_64、HASH、xxHash3
