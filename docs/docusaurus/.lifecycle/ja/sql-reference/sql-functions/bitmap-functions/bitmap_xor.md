---
displayed_sidebar: "Japanese"
---

# bitmap_xor

## 説明

`lhs` と `rhs` に固有の要素で構成されるセットを計算します。これは `bitmap_or(lhs, rhs)` と `bitmap_and(lhs, rhs)` の論理的な相当であり、 `bitmap_andnot(bitmap_or(lhs, rhs), bitmap_and(lhs, rhs))`（補集合）と同等です。

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