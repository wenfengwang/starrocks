---
displayed_sidebar: "Japanese"
---

# bitmap_andnot

## Description

`lhs` に存在するが `rhs` に存在しないビットマップ値を返し、新しいビットマップを返します。

## Syntax

```Haskell
bitmap_andnot(BITMAP lhs, BITMAP rhs)
```

## Examples

```plain text
mysql> select bitmap_to_string(bitmap_andnot(bitmap_from_string('1, 3'), bitmap_from_string('2'))) cnt;
+------+
|cnt   |
+------+
|1,3   |
+------+

mysql> select bitmap_to_string(bitmap_andnot(bitmap_from_string('1,3,5'), bitmap_from_string('1'))) cnt;
+------+
|cnt   |
+------+
|3,5   |
+------+
```

## keyword

BITMAP_ANDNOT, BITMAP