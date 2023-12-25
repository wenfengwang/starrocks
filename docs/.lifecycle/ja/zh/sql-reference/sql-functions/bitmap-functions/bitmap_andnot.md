---
displayed_sidebar: Chinese
---

# bitmap_andnot

## 機能

二つの入力されたbitmapの差集合を計算します。差集合とは、第一の集合に存在し、第二の集合には存在しない全ての要素を含む集合のことです。

## 文法

```Haskell
bitmap_andnot(lhs, rhs)
```

## パラメータ説明

`lhs`: サポートされるデータ型はBITMAPです。

`rhs`: サポートされるデータ型はBITMAPです。

## 戻り値の説明

戻り値のデータ型はBITMAPです。

## 例

```plain text
select bitmap_to_string(bitmap_andnot(bitmap_from_string('1, 3'), bitmap_from_string('2'))) cnt;
+------+
|cnt   |
+------+
|1,3   |
+------+

select bitmap_to_string(bitmap_andnot(bitmap_from_string('1,3,5'), bitmap_from_string('1'))) cnt;
+------+
|cnt   |
+------+
|3,5   |
+------+
```
