---
displayed_sidebar: English
---

# bitmap_has_any

## 説明

2つのBitmapカラム間に共通する要素が存在するかを計算し、戻り値はBoolean値です。

## 構文

```Haskell
BOOLEAN BITMAP_HAS_ANY(BITMAP lhs, BITMAP rhs)
```

## 例

```Plain Text
MySQL > select bitmap_has_any(to_bitmap(1),to_bitmap(2)) cnt;
+------+
| cnt  |
+------+
|    0 |
+------+

MySQL > select bitmap_has_any(to_bitmap(1),to_bitmap(1)) cnt;
+------+
| cnt  |
+------+
|    1 |
+------+
```

## キーワード

BITMAP_HAS_ANY, BITMAP
