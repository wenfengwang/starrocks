---
displayed_sidebar: Chinese
---

# bitmap_xor

## 機能

二つのBitmap間の重複しない要素からなる集合を計算し、論理的には `bitmap_andnot(bitmap_or(lhs, rhs), bitmap_and(lhs, rhs))`（補集合）に相当します。

## 文法

```Haskell
bitmap_xor(lhs, rhs)
```

## パラメータ説明

`lhs`: サポートされるデータタイプは BITMAP です。

`rhs`: サポートされるデータタイプは BITMAP です。

## 戻り値の説明

戻り値のデータタイプは BITMAP です。

## 例

```plain text
mysql> select bitmap_to_string(bitmap_xor(bitmap_from_string('1, 3'), bitmap_from_string('2'))) cnt;
+------+
|cnt   |
+------+
|1,2,3 |
+------+
```
