---
displayed_sidebar: "Japanese"
---

# bitmap_xor

## Description

`lhs` と `rhs` の要素から成るユニークな要素からなるセットを計算します。これは、論理的には `bitmap_andnot(bitmap_or(lhs, rhs), bitmap_and(lhs, rhs))` (補完集合) と同等です。

## Syntax

```Haskell
bitmap_xor(BITMAP lhs, BITMAP rhs)
```

## Examples

```plain text
mysql> select bitmap_to_string(bitmap_xor(bitmap_from_string('1, 3'), bitmap_from_string('2'))) cnt;
+------+
|cnt   |
+------+
|1,2,3 |
+------+
```

## keyword

BITMAP_XOR,  BITMAP