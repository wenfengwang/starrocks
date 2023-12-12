---
displayed_sidebar: "Japanese"
---

# bitmap_andnot

## 説明

`lhs` に存在するが `rhs` に存在しないビットマップの値を返し、新しいビットマップを返します。

## 構文

```Haskell
bitmap_andnot(BITMAP lhs, BITMAP rhs)
```

## 例

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

## キーワード

BITMAP_ANDNOT, BITMAP