---
displayed_sidebar: Chinese
---

# xx_hash3_64

## 機能

64ビットのxxhash3ハッシュ値を入力文字列に対して返します。xx_hash3_64はAVX2命令セットを使用し、[murmur_hash3_32](./murmur_hash3_32.md)よりも高速で優れたパフォーマンスを提供します。

この関数はバージョン3.2.0からサポートされています。

## 文法

```Haskell
BIGINT XX_HASH3_64(VARCHAR input, ...)
```

## パラメータ説明

`input`: 対応するデータタイプはVARCHARです。

## 戻り値の説明

戻り値のデータタイプはBIGINTです。入力値がNULLの場合、戻り値もNULLになります。

## 例

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

## キーワード

XX_HASH3_64、HASH、xxHash3
