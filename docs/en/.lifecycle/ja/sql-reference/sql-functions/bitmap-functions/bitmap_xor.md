---
displayed_sidebar: "Japanese"
---

# bitmap_xor

## 説明

`lhs` と `rhs` の要素のうち、ユニークな要素からなるセットを計算します。これは論理的には `bitmap_or(lhs, rhs)` と `bitmap_and(lhs, rhs)` の論理積の補集合と等価です。

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

BITMAP_XOR,  BITMAP
