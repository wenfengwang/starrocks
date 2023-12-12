---
displayed_sidebar: "Japanese"
---

# xx_hash3_64

## Description

入力文字列の64ビットxxhash3ハッシュ値を返します。 xx_hash3_64はAVX2命令を使用して[murmur_hash3_32](./murmur_hash3_32.md)よりも優れたパフォーマンスを持ち、多くのソフトウェアで広く統合されている最新のハッシュ品質を持っています。

この関数はv3.2.0からサポートされています。

## Syntax

```Haskell
BIGINT XX_HASH3_64(VARCHAR input, ...)
```

## Examples

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

## keyword

XX_HASH3_64,HASH,xxHash3