---
displayed_sidebar: Chinese
---

# murmur_hash3_32

## 機能

入力された文字列の32ビットmurmur3ハッシュ値を返します。

## 構文

```Haskell
MURMUR_HASH3_32(input, ...)
```

## 引数説明

`input`: 対応するデータ型はVARCHARです。

## 戻り値の説明

戻り値のデータ型はINTです。

## 例

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
