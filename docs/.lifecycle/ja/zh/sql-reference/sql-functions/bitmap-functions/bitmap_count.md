---
displayed_sidebar: Chinese
---

# bitmap_count

## 機能

bitmap 中の重複しない値の数をカウントします。

## 文法

```Haskell
INT BITMAP_COUNT(any_bitmap)
```

## 返り値の説明

返り値のデータ型は INT です。

## 例

```Plain Text
MySQL > select bitmap_count(bitmap_from_string("1,2,4"));
+-------------------------------------------+
| bitmap_count(bitmap_from_string('1,2,4')) |
+-------------------------------------------+
|                                         3 |
+-------------------------------------------+

MySQL > select bitmap_count(NULL);
+--------------------+
| bitmap_count(NULL) |
+--------------------+
|                  0 |
+--------------------+
```
