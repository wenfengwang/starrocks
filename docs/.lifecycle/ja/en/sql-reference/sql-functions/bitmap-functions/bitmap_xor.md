---
displayed_sidebar: English
---

# bitmap_xor

## 説明

`lhs`と`rhs`に固有の要素からなる集合を計算します。これは、`bitmap_andnot(bitmap_or(lhs, rhs), bitmap_and(lhs, rhs))`（補集合）と論理的に等価です。

## 構文

```Haskell
bitmap_xor(BITMAP lhs, BITMAP rhs)
```

## 例

```plain text
mysql> select bitmap_to_string(bitmap_xor(bitmap_from_string('1, 3'), bitmap_from_string('2'))) cnt;
+------+
|cnt   |
+------+
|1,2,3 |
+------+
```

## キーワード

BITMAP_XOR, BITMAP
